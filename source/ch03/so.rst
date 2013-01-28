.. include:: ../LINKS.rst


是也乎
==============


这里的一个建议是，尽量不要对 ngx.var.request_uri 进行正则匹配，因为 Nginx 的标准变量 $request_uri
是未经过 URI 解码的原始形式，比如 "/a b" 和 "/a%20b" 是彼此等价的
URI，但作为字符串它们又是不相等的。建议改成分别对 ngx.var.uri 和 ngx.req.get_args()["test"]
进行匹配。

可以看一看下面这个例子：

    location /test {
        default_type "text/plain";
        content_by_lua '
            ngx.say("request uri: ", ngx.var.request_uri)
            ngx.say("uri: ", ngx.var.uri)
            ngx.say("arg test: ", ngx.var.arg_test)
            ngx.say("arg test 2: ", ngx.req.get_uri_args()["test"])
        ';
    }

假设当前 nginx 监听的是本机的 1984 端口，则当在 Firefox 地址栏里输入下面这个 URL 时，

    http://localhost:1984/test it?test=a b

会得到这样的页面输出：

    request uri: /test%20it?test=a%20b
    uri: /test it
    arg test: a%20b
    arg test 2: a b

我们看到，Firefox 作为编写良好的 HTTP 客户端，根据 RFC 的要求，自动把原始 URL 中的空格字符编码为了 %20 序列。

