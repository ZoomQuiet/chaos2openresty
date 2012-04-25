.. include:: ../LINKS.rst


使用 gzip 数据
============================
   agentzh agentzh@gmail.com
发件人当地时间:   发送时间 14:29 (GMT+08:00)。发送地当前时间：下午3:50。 ✆
发送至:   openresty@googlegroups.com
日期:  2012年3月20日 下午2:29
主题:  Re: [openresty:302] 【OT】请教个nginx proxy时候后端返回gzip问题

问题
----------------------------


分析
----------------------------
On Tue, Mar 20, 2012 at 2:10 PM, 刘太华 <defage@gmail.com> wrote:
> 这样说来，
> 1、nginx在处理压缩的时候，会看后端response返回的header中是否有Content-Encoding:gzip？然后才来决定是否对此次response做压缩

是的。实现细节可以参见 nginx 1.0.12 源码树中 src/http/modules/ngx_http_gzip_filter_module.c 文件的第 250 行。相关的那段代码是

   if (!conf->enable
       || (r->headers_out.status != NGX_HTTP_OK
           && r->headers_out.status != NGX_HTTP_FORBIDDEN
           && r->headers_out.status != NGX_HTTP_NOT_FOUND)
       || (r->headers_out.content_encoding
           && r->headers_out.content_encoding->value.len)
       || (r->headers_out.content_length_n != -1
           && r->headers_out.content_length_n < conf->min_length)
       || ngx_http_test_content_type(r, &conf->types) == NULL
       || r->header_only)
   {
       return ngx_http_next_header_filter(r);
   }

即当 Content-Encoding 响应头已经存在时，ngx_gzip 的响应头过滤器会直接跳过处理，转给下一个过滤器。

> 2、 或者ngx接到后端response，解压后再次做压缩，然后发送给客户端？
>

这样做开销太大，解压和再压缩没有实际意义，只是徒耗 CPU 计算资源 :)


> 如果是1的情况，那使用ngx_lua做一些capture的时候，能碰到获取到的response压缩过的，那么此时用lua对返回的body做逻辑的时候，应该会遇到解压缩的问题？
>

一般，让使用 ngx_proxy 访问后端 http 服务，同时在 ngx_lua 中使用 capture API 时，应该显式地控制是否启用 gzip（因为默认情况下，nginx 子请求会自动继承父请求的请求头）. 例如在配置了 proxy_pass 的内部 location 中通过

  proxy_set_header  Accept-Encoding  "";

显式地禁用后端的 gzip 压缩，或者通过

  proxy_set_header  Accept-Encoding  "gzip";

显式地启用后端压缩。看你具体的应用场景了。如果让上游 http 服务自己进行 gzip 压缩的话，在 lua 里得到的数据自然就是 gzip 压缩以后的，所以需要使用 lua 世界里的 gzip 库来解压，例如 lua-zlib 库：https://github.com/brimworks/lua-zlib


解决
----------------------------
Issue #12: How to handle gziped capture? · chaoslawful/lua-nginx-module
https://github.com/chaoslawful/lua-nginx-module/issues/12


If you're using the ngx_proxy module for your subrequests, try using the following directive to prevent forwarding your original request's headers:

    proxy_pass_request_headers off;

See http://wiki.nginx.org/NginxHttpProxyModule#proxy_pass_request_headers for more details.

Alternatively, you can explicitly remove the Accep-Encoding header in your subrequest location like this:

    more_clear_input_headers Accept-Encoding;

You need the ngx_headers_more module though. See http://wiki.nginx.org/NginxHttpHeadersMoreModule#more_clear_input_headers for more details :)


>>>>>>>

All three options work.

OPTION ONE

Placed:
    more_clear_input_headers Accept-Encoding;
    in "location /cap2"

OPTION TWO

    Placed:
    proxy_pass_request_headers off;
    in "location /cap2"

OPTION THREE

    Downloaded https://github.com/brimworks/lua-zlib :
    make linux
    cp zlib.so /usr/lib/lua/5.1/zlib.so

    added following to conf:
    local zlib = require("zlib")
    local stream = zlib.inflate()
    local inflated_body = stream(res1.body)
    local res2 = ngx.location.capture("/cap2?"..inflated_body)

I like having all three as options (may need gz some time and good to be able to modify the headers). I also like the work that both of you are doing; very cool.




