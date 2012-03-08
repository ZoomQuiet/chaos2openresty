
撞!
==============

*1 lua handler aborted: runtime error: /usr/local/openresty/nginx/conf/lua/chk.lua:6: module 'ksc' not found:
        no field package.preload['ksc']
        no file '/usr/local/openresty/lualib/ksc.lua'
        no file './ksc.lua'
        no file '/usr/local/openresty/luajit/share/luajit-2.0.0-beta9/ksc.lua'
        no file '/usr/local/share/lua/5.1/ksc.lua'
        no file '/usr/local/share/lua/5.1/ksc/init.lua'
        no file '/usr/local/openresty/luajit/share/lua/5.1/ksc.lua'
        no file '/usr/local/openresty/luajit/share/lua/5.1/ksc/init.lua'
        no file '/usr/local/openresty/lualib/ksc.so'
        no file './ksc.so'
        no file '/usr/local/lib/lua/5.1/ksc.so'
        no file '/usr/local/openresty/luajit/lib/lua/5.1/ksc.so'
        no file '/usr/local/lib/lua/5.1/loadall.so', client: 127.0.0.1, server: localhost, request: "POST /=/chk HTTP/1.1", host: "127.0.0.1:9090"
        

Checking configuration for correct syntax and
then trying to open files referenced in configuration...
nginx: [emerg] "lua_package_path" directive is not allowed here in /usr/local/openresty/nginx/conf/my_openresty.conf:10
nginx: configuration file /usr/local/openresty/nginx/conf/nginx.conf test failed




