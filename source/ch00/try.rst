.. include:: ../LINKS.rst


+22分钟:初尝
==============

整起来先!
- 嗯嗯嗯,现在可以计时了: `00:00`

安装
--------

`参考:` `官方安装说明 <http://openresty.org/cn/#Installation>`_


.. sidebar:: 提示

    - 不专门说明的话,指的都是笔者的个人环境:

      - MAC OS X 10.7.3
      - brew 0.8.1
      - ngx_openresty-1.0.8.26
      - lua 5.1.4
      - luarocks 2.0.8

走起 ::

    $ brew install pcre
    $ axel http://agentzh.org/misc/nginx/ngx_openresty-1.0.8.26.tar.gz
    $ tar xzvf ngx_openresty-1.0.8.26.tar.gz
    $ cd ngx_openresty-1.0.8.26
    $ ./configure  --with-cc-opt="-I/usr/local/Cellar/pcre/8.21/include" \
      --with-ld-opt="-L/usr/local/Cellar/pcre/8.21/lib"
    $ make
    $ make install
    $ $ /usr/local/openresty/nginx/sbin/nginx -v
    nginx: nginx version: ngx_openresty/1.0.10.48


.. note:: (~_~)

  - 这儿的配置,和官方不同
  - 是因为: `Issue #3: <https://github.com/agentzh/ngx_openresty/issues/3>`_



跑
--------

玩 `OpenResty`_ 就是要反复调整内置nginx 的配置,为了方便起见

- 推荐 :download:`openresty.server <../_static/openresty.server>`.
- 是简化版的: `Nginx-init-ubuntu <http://wiki.nginx.org/Nginx-init-ubuntu>`_
- 兼容 Linux/Unix/MAC 系统

::

    $ /opt/sbin/openresty.server 
    usage: /opt/sbin/openresty.server {check|start|term|stop|reload|restart|upgrade}


建议部署在 `/opt/sbin` 目录 然后就可以随时:

  - `/opt/sbin/openresty.server check` 检验配置脚本是否正确
  - `/opt/sbin/openresty.server start` 启动 Nginx
  - `/opt/sbin/openresty.server reload` 热加载新的配置文件
  - 测试是否在跑? 使用 `ps aux | grep nginx`





玩
----------

那么开始 `OpenResty`_ 的编程吧!

理解重要的目录结构::

    /usr/local/openresty/
      +- luajit         Lua实时编译组件
      +- lualib         Lua预装原生库
      +- nginx          openresty 狠狠定制过的稳定版本 Nginx
        +- log          默认的日志,pid 存放目录
        +- conf         默认的配置文件目录
          +- nginx.conf


那么,要作是只是:

- 修订 主配置文件,加入我的专用配置文件包含: ::

    http {
        include my_openresty.conf;
        ...



- 创建 `my_openresty.conf` 并写入: ::

    server {
      listen       9090;
      server_name  localhost;
      error_log   logs/error.my_openresty.log info;
      location / {
          content_by_lua "ngx.say('Hello,world!')";
      }
    }


- 然后享受成果  ::

    $ /opt/sbin/openresty.server reload
    $ curl localhost:9090
    Hello,world!





改
--------

每次都要修改和各种 web 服务配置杂在一起的 nginx 配置,而且再重启 nginx 才能见到成果?!

当然有好招
- 引入独立 Lua 脚本,到指定 url 路由
- 增补 `my_openresty.conf` : ::

    server {
      listen       9090;
      server_name  localhost;
      error_log   logs/error.my_openresty.log info;
      location / {
          content_by_lua "ngx.say('Hello,world!')";
      }
      location /readme {
          lua_code_cache off; # <<< 关键技巧!
          content_by_lua_file conf/lua/readme.lua;
      }
    }


- 创建业务脚本: `/usr/local/openresty/nginx/conf/lua/readme.lua`


.. code-block:: lua

    -- readme for my openresty app.
    ngx.say("Hallo World!")


- 再重启 nginx, `curl localhost:9090/readme` 获得同样的 "Hallo World!"

  - 要是看到:

::

    nginx: [warn] lua_code_cache is off; this will hurt performance in /usr/local/openresty/nginx/conf/my_openresty.conf:23


- 别怕,正常的提醒,說没有 Lua 脚本的缓存,整个性能会下载,没关系,测试开发阶段,俺们不论运行性能,先关注开发效率!

- 然后,增补 `readme.lua`

.. code-block:: lua

    -- readme for my openresty app.
    VERTION="URIsAok4openresty v12.03.6"

    ngx.say(VERTION
        ,"\n\tusage:"
        ,"$crul -d 'uri=http://sina.com' locahost:9090/=/chk"
        )


- 不用重启, 再次 curl 就发现返回已经变化了!

::

    $ curl localhost:9090/readme
    URIsAok4openresty v12.03.6
      usage:$crul -d 'uri=http://sina.com' 127.0.0.1:9090/=/chk



小结
------

不出意外的话, `22:00` 用在这个阶段,太足够了!

应该已经体验到  `OpenResty`_ 的核心爽直了?!


具体的:

- 不用跟 c 死磕,想給 Nginx 扩展功能,不用以往麻烦的调试过程循环 ::

      改c代码
      ^  `->编译
      |   `->替换老.so
      |     `->重启nginx (加载模块,有时,还必须整个重新编译 nginx)
      |       `-> curl 请求测试
      |               |
      +---------------/


- 而是,非常直接的: ::


      改lua代码
       ^ `-> curl 请求测试
       |        |
       +--------/



而且, **性能几乎没有下降!**



