.. include:: ../LINKS.rst



替换 luacurl 回归非阻塞
============================


问题
----------------------------


分析
----------------------------


解决
----------------------------


agentzh agentzh@gmail.com
发送至:   openresty@googlegroups.com
日期:  2012年3月16日 下午6:33
主题:  Re: ngx_lua 中访问远方 HTTP 服务的两种推荐的做法 (Was Re: [openresty:287] 42分钟,乱入 OpenResty 手册!-))
明显是因为你漏了我的示例代码中的下面这一行：
>>
>>     resolver 114.114.114.114;
>>
>> 咱能直接复制粘贴么？上面的 Nginx 出错信息也指示你未配置 resolver 指令：
>>
>> http://wiki.nginx.org/HttpCoreModule#resolver
>>
>
> - FT! 俺想 DNS 当前可用就没有配置吼,,
> - 不明白的代码就没抄,,,
> 看来都有深意,先用起来,再理解,,,
>    - 是为了防止各种内部网络的乱解析?
>

因为 Nginx 不会自动读取当前系统中的 DNS resolver 配置。



非阻塞访问远方 http 服务有两种做法。下面以从 ngx_lua 中访问百度搜索为例，演示一下这两种做法：

1. 使用 nginx 子请求 + ngx_proxy 模块：

    resolver 114.114.114.114;

    location = /baidu {
        internal;
        proxy_pass http://www.baidu.com/s?wd=$arg_query;
    }
    
    location = /test { 
        content_by_lua '
            local res =
                ngx.location.capture("/baidu", { args = { query = "openresty" }})
            if res.status ~= 200 then
                ngx.say("failed to query baidu: ", res.status, ": ", res.body)
                return
            end
            ngx.say(res.body)
        ';
    }

然后请求这里的 /test 接口便可以得到 baidu 里搜索 openresty 查询词时的 HTML 结果页（不过要仔细字符编码哦）。

2. 使用 cosocket API 访问之：

    resolver 114.114.114.114;

    location = /test {
        content_by_lua '
            local sock = ngx.socket.tcp()
            sock:settimeout(1000)
            local ok, err = sock:connect("www.baidu.com", 80)
            if not ok then
                ngx.say("failed to connect to baidu: ", err)
                return
            end
            local req = "GET /s?wd=openresty HTTP/1.0\\r\\nHost: www.baidu.com\\r\\n\\r\\n"
            sock:send(req)
            local read_headers = sock:receiveuntil("\\r\\n\\r\\n")
            headers, err = read_headers()
            if not headers then
                ngx.say("failed to read response headers: ", err)
                return
            end
            local body, err = sock:receive("*a")
            if not body then
                ngx.say("failed to read response body: ", err)
                return
            end
            ngx.say(body)
        ';
    }

这里直接用 TCP cosocket API 现写了一个简单的 HTTP 1.0 客户端访问 baidu.com，得到了 openresty 查询词的结果页。当然，未来等有人仿照上面的做法，专门基于 cosocket 实现了完整的  HTTP 客户端库 lua-resty-http 之后，便会更有方便，就像那些已经基于 cosocket 实现的 lua-resty-memcached, lua-resty-redis 和 lua-resty-mysql 库一样（见 https://github.com/agentzh/lua-resty-memcached ，还有 https://github.com/agentzh/lua-resty-redis 以及 https://github.com/agentzh/lua-resty-mysql ）

以且仅以上面两种方式访问远方服务在 ngx_lua 中才是非阻塞的。

Regards,
-agentzh


怎么可以观察到  proxy_pass 真实进行请求的 url ?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 
> - 以便明确lua 的处理是否正确?

有两种方法：

一是使用 nc 工具。比如在本地用下面的命令启动 nc 以监听 1978 端口：

   nc -l 1978

（有的系统会需要使用命令 nc -l -p 1978）,然后临时修改你的 nginx 配置以便让 proxy_pass 指向它：

   proxy_pass http://127.0.0.1:1978/phish/?$args

重新加载 nginx 配置后如从前那般请求你的接口，然后在启动 nc 的终端检查 ngx_proxy 发送的原始的 HTTP 请求。

2. 重新编译 nginx 以启用 Nginx 调试日志，然后在 nginx 配置文件中使用 debug 日志级别。然后正常请求你的接口，在 Nginx 的 error.log 文件中应当会看到类似下面这样的 dump:

[debug] 20844#0: *1 http proxy header:
"GET /foo?blah HTTP/1.0^M
Host: 127.0.0.1:1987^M
Connection: close^M
^M
"  

当然，使用 tcpdump 之类的抓包工具来搞也是可以的。



