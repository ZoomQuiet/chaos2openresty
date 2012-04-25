.. include:: ../LINKS.rst


固化nginx 运行根目录
============================


问题
----------------------------


分析
----------------------------


解决
----------------------------


嗯嗯嗯,福灵心致,无意中解决,,,
> 参考:lua-users wiki: Modules Tutorial
> http://lua-users.org/wiki/ModulesTutorial
> 俺的布局:
> /usr/local/openresty/nginx/conf
>    +- lua
>       +- ksc.lua
>       +- chk.lua
> -- ksc.lua
> module(..., package.seeall)
> ,,,
> -- chk.lua
> local KSC = require "lua.ksc"
>
> 就一切安定了,agentzh 没说:
> - module(..., package.seeall) 这儿直接就用 ...
> - 所有 openresty 环境中,俺猜都是从 nginx/conf 为起点的
>  - 所以,即使 包和应用都在同一级目录中
>  - 依然要使用子包的模式来引用 ;-(
>

目前，在 ngx_lua 中，如果在 lua_package_path 或者 lua_package_cpath 配置指令中指定相对路径作为 Lua 包的搜索路径，其实是相对于启动 nginx 服务器时的当前工作目录的，而不是 nginx 的“配置前缀”（configure prefix）。

举例来说，假设你是在 /home/agentzh/ 这个当前工作目录下用下面的命令启动 nginx:

   /usr/local/openresty/nginx/sbin/nginx

则 nginx 内部包括 Lua 环境所使用的当前工作目录就是执行启动命令时的当前工作目录，也就是 /home/agentzh/，而不是许多人期望的 Nginx “配置前缀”（在该例中便是 /usr/local/openresty/nginx/sbin/nginx/）。

通过下面的接口可以动态获取实际使用的当前工作目录：

   location = /pwd {
       content_by_lua '
           local fname = "/tmp/pwd.out"
           assert(os.execute("pwd > " .. fname) == 0)
           local f = io.open(fname, "r")
           assert(f)
           local content = f:read("*a")
           ngx.say("pwd: ", content)
           f:close()
       ';
   }

对于上面那个例子，访问 /pwd 可以得到输出

   pwd: /home/agentzh

这个确实有些违反直觉。毕竟在 nginx 中的许多地方，相对路径都是相对于“配置前缀”路径的，而非真实的当前工作目录。

或许我们应该在 nginx 启动时自动把整个进程所使用的当前工作目录切换到 nginx 的“配置前缀”路径？大家伙有什么看法？如果真要做这件事情的话，因为在 ngx_lua 中实现，还是以单独的 nginx 模块的形式实现，还是以 nginx 核心的补丁的形式来实现？

Regards,
-agentzh


> 其实最简单的方法是在你的启动脚本中先自己 cd 到 nginx 的配置前缀目录，比如
> /usr/local/openresty/nginx/，然后再行启动 nginx.
>