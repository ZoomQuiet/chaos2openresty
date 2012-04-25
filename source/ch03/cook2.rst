.. include:: ../LINKS.rst



自制 lua-resty-* 模块
============================


问题
----------------------------


分析
----------------------------
On Sat, Mar 10, 2012 at 4:56 PM, Zoom.Quiet <zoom.quiet@gmail.com> wrote:

    - 是也乎,是也乎,这种情景在面向 web 开发时,最常见:
     - 现在 openresty 对 mysql/mongo/memcached/redis/pg 进行了合理的非阻塞封装


值得注意的是，ngx_openresty 本身并没有对这些上层协议进行任何封装。ngx_lua 仅是提供了 TCP socket API 而已。你列出的那些东西都是在 TCP 协议之上搞了一套上层应用协议罢了。
 

     - 其它的,得自行解决了吧?


其他的多可以用几行 Lua 自行解决。
 

       - 比较自然的,通过内部消息系统进行子任务分发
       - 前端直接302 响应,給出任务等待url ?


对于大多数 web 应用来说，所谓的“内部消息系统”本身并不是一种优美的封装。ngx_lua 的并发模型对于用户程序员来说，长得和 nodejs 这样的“显式”事件模型是很不相同的。
 

    即:
       - 确认阻塞任务后,立即返回請求,給出新的請求url
       - 将阻塞任务,丢给内部消息系统,不进行具体的阻塞执行
       - 通过其它专用进程,进行 apache prefork 式的阻塞处理,和响应


整个系统就不应该有阻塞的网络 I/O 通信。否则 C10K 根本无法实现 :)
 

       只是,需要明白 http RESTful 行为的客户端来配合?


这与 Nginx 下游的 HTTP 客户端的实现无关。
 

    过程类似 附件泳道图?


不类似。
 

    问题在:
       - 怎么能将一个请求在内部,进行安全的分离?


ngx_lua 是通过 Lua coroutine （或者说 Lua 协程）来实现请求之间的透明隔离的。

 

       即:图中 1.0->2.0 怎么作到?


大部分 ngx_lua 的应用并不需要引入用户级别的任务队列。对于那些特别需要队列的场合，一般是使用 beanstalkd 或者 activemq 这样专门的外部队列服务。
 

       - 而且,怎么将阻塞工作进程和非阻塞工作进程进行显示的标定/指代使用?


我觉得这里你混淆了阻塞与非阻塞 I/O，同步与异步操作这两对术语。

ngx_lua 追求的模型是同步操作 + 非阻塞 I/O. 非阻塞 I/O 并不一定需要异步操作，同理，同步操作未必总是阻塞的。Nodejs 中的做法并不是唯一选择。

另：当你打算讨论新的问题时，建议开辟新的 mail thread，我觉得我们已经离题太远了。当前的 thread 主题可是“[BSD] agx.now() 以及 xss 问题”哈。

Regards,
-agentzh



解决
----------------------------


agentzh agentzh@gmail.com
日期:  2012年3月10日 下午9:34
主题:  Re: ngx_openresty 应用的开发模式(Was Re: ngx_lua 中访问远方 HTTP 服务的两种推荐的做法 (Was Re: [openresty:187] 42分钟,乱入 OpenResty 手册!-)))

貌似我的搜索水平更高一些：

http://www.nginx-discovery.com/2011/03/day-32-moving-to-testnginx.html
http://www.nginx-discovery.com/2011/03/day-40-testnginx-new-features.html

我准备的中文材料和文档相对较少，是因为这个项目从一开始就是国际化的。我只能先保证英文文档，毕竟我们的资源是非常有限的 :) 试想 Nginx
项目早期也只有俄文文档。

我欢迎汉语还不错的热心用户在现有的英文资料的基础上，贡献更多的汉语材料。

> - Test::Nginx 设计目标是对 nginx 的 C 扩展模块进行测试
>  - 俺还是要问: 怎么应用于 lua 模块的开发?

Test::Nginx 不仅可以运行测试 Nginx C 模块，也可以用来测试 lua-resty-* 库这样基于 ngx_openresty 的 Lua 库。
另外，现有的 lua-resty-* 库和我们维护的所有 nginx 模块的测试集都是 Test::Nginx 的“活教材”。

例如 lua-resty-memcached 库的测试集：

    https://github.com/agentzh/lua-resty-memcached/blob/master/t/sanity.t

再比如 ngx_lua 模块的规模庞大的测试集：

    https://github.com/chaoslawful/lua-nginx-module/tree/master/t/

我的不少 Test::Nginx 的用户都是从“依葫芦画瓢”入手的。因为这些测试例大多非常直观，这也正是基于 Test::Base 的测试框架的主要优势之一 :)

Best regards,
-agentzh

>
> https://github.com/agentzh/lua-resty-redis/blob/master/t/sanity.t
> 应该是对应的测试案例;
> 但是,
> lua-resty-redis/valgrind.suppress at master · agentzh/lua-resty-redis
> https://github.com/agentzh/lua-resty-redis/blob/master/valgrind.suppress
> 看起来,还要配合 valgrind 一起跑的?
>

Test::Nginx 默认情况下并不使用 valgrind 运行测试集。仅当你指定了 TEST_NGINX_VALGRIND=1
时才会使用 valgrind 运行测试集。细节可以参考 Test::Nginx::Socket 中相关的官方文档：


这里有一处笔误，当是名为 TEST_NGINX_USE_VALGRIND 环境变量。

http://search.cpan.org/dist/Test-Nginx/lib/Test/Nginx/Socket.pm#TEST_NGINX_USE_VALGRIND

>
> 那么,其实,还是需要本地跑个nginx 实例进程:
> - 然后通过 Test::Nginx 进行测试的自动加载/案例执行/結果收集通告
> - 形成写lua->跑案例->改 lua 的开发流程?
>
> 暂时想象不出具体,是怎么样的一个 TDD 操作,,,
>

以 lua-resty-memcached 库为例，一个典型的 TDD 周期是这样的：

1. 编辑 t/sanity.t，准备测试用例;
2. 在 lib/resty/memcached.lua 中实现相关的代码;
3. 运行命令 PATH=/usr/local/openresty/nginx/sbin/nginx prove t/sanity.t
会运行测试文件 t/sanity.t 并生成测试报告到终端;
4. 根据测试结果，重返步骤 1 或 2.

其中步骤 3 中的命令一般会封装进一个 Makefile 文件，所以一般直接执行命令 make test 就可以了。

>
> - 嗯嗯嗯,看来 DSL 的创造习惯,从 perl 时代就已经开始了,,,
> - 呜乎矣哉,俺是习惯寻找现行的 dsl 解决问题,而不是相反
> 实在是面对的领域不同,,,

Test::Nginx 提供了现成的 DSL 记法，可以直接解决问题。我不明白你说的"相反"是什么意思。


On Sat, Mar 10, 2012 at 9:28 PM, agentzh <agentzh@gmail.com> wrote:
>
> 以 lua-resty-memcached 库为例，一个典型的 TDD 周期是这样的：
>
> 1. 编辑 t/sanity.t，准备测试用例;
> 2. 在 lib/resty/memcached.lua 中实现相关的代码;
> 3. 运行命令 PATH=/usr/local/openresty/nginx/sbin/nginx prove t/sanity.t
> 会运行测试文件 t/sanity.t 并生成测试报告到终端;
> 4. 根据测试结果，重返步骤 1 或 2.
>

这里的步骤 3 有误，应当作：

    PATH=/usr/local/openresty/nginx/sbin:$PATH prove t/sanity.t

或者这样写以便运行 t/ 目录下的所有 .t 文件（包括子孙目录中的 .t 文件）：

PATH=/usr/local/openresty/sbin/nginx/sbin:$PATH prove -r t

这里设置 PATH 环境的目的是为了指定 Test::Nginx 测试台所使用的 nginx 可执行文件的位置。

如果只想运行某个文件中的特定用例，直接在该用例上标记 --- ONLY 即可。或者用 --- SKIP
跳过相应的用例。另一个常用的特殊标记是 --- LAST，表示运行到该用例就不再继续。




