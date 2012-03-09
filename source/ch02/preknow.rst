.. include:: ../LINKS.rst


调研
==============

核心是先理解 `Lua`_ 的包管理机制:

- `lua-users wiki: Modules Tutorial <http://lua-users.org/wiki/ModulesTutorial>`_
- `lua-users wiki: Module Definition <http://lua-users.org/wiki/ModuleDefinition>`_
- 以及更加直接的动力:

`OpenResty`_ 邮件列表中的讨论过程,核心作者的`明确提示 <https://groups.google.com/forum/?fromgroups#!topic/openresty/AK5s69NMOFA>`_ :

::

    agentzh agentzh@gmail.com
    发件人当地时间:     发送时间 11:00 (GMT+08:00)。发送地当前时间：上午12:07。 ✆
    回复:  openresty@googlegroups.com
    主题:  Re: [openresty:99] 纯lua 问题: attempt to ... (a nil value)

    ...
    一般地，由于全局变量是每请求的生命期，因此以此种方式定义的函数的生命期也是每请求的。
    为了避免每请求创建和销毁 Lua closure 的开销，建议将函数的定义都放置在自己的 Lua module 中，
    例如：

        -- my_module.lua
        module("my_module", package.seeall)
        function foo() ... end

    然后，再在 content_by_lua_file 指向的 .lua 文件中调用它：

        local my_module = require "my_module"
        my_module:foo()

    因为 Lua module 只会在第一次请求时加载一次
    （除非显式禁用了 lua_code_cache 配置指令），后续请求便可直接复用。

    Regards,
    -agentzh



.. warning:: 

    - 来吧!

