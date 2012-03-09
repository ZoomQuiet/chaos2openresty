.. include:: ../LINKS.rst


整个儿的
==============

最终,完成所有功能的配置和代码:

- 版本仓库: https://github.com/ZoomQuiet/urisaok/tree/openresty
- 对外发布: http://py.kingsoft.net:8008/=/chk    
- 相关视频: `{3月8日语音讲座vol.45}ZQ: 网址云服务嵌入Nginx - Youku <http://v.youku.com/v_show/id_XMzYyNjIzMDQ4.html>`_



nginx.conf
---------------


::

    http {
        ...
        # 设置纯Lua扩展库PATH(';;' is the default path):
        lua_package_path '/usr/local/openresty/nginx/conf/lua/?.lua;;';
        include my_openresty.conf;
        ...


my_openresty.conf
----------------------

::

    server {
        listen       9090;
        server_name  localhost;
        error_log   logs/error.my_openresty.log info;
        default_type 'text/plain';

        location / {
            content_by_lua_file conf/lua/readme.lua;
        }
        location /readme {
            content_by_lua_file conf/lua/readme.lua;
        }

        location ~ ^/=/(\w+) {
            content_by_lua_file conf/lua/$1.lua;

            lua_code_cache off;
        }
    }



readme.lua
----------------------

.. code-block:: lua

    -- readme for /=/ export base help!
    ngx.req.read_body()
    VERTION="URIsAok4openresty v12.03.6"
    ngx.say(VERTION
        ,"\n\tusage:"
        ,"$crul -d 'uri=http://sina.com' 127.0.0.1:9090/=/chk"
        )


ksc.lua
----------------------

.. code-block:: lua

    -- KCS API support 
    module("ksc", package.seeall)
    curl = require "luacurl"
    function _fetch_uri(url, c)
        local result = { }
        if c == nil then 
            c = curl.new() 
        end
        c:setopt(curl.OPT_URL, url)
        c:setopt(curl.OPT_WRITEDATA, result)
        c:setopt(curl.OPT_WRITEFUNCTION, function(tab, buffer)
            table.insert(tab, buffer)
            return #buffer
        end)
        local ok = c:perform()
        return ok, table.concat(result)
    end
    -- global var
    PHISHTYPE = {["-1"]='UNKNOW'
        ,["0"]='GOOD'
        ,["1"]='PHISH'
        ,["2"]='MAYBE PHISH'
        }
    --ngx.say(PHISHTYPE["2"])
    APPKEY = "k-60666"
    SECRET = "99fc9fdbc6761f7d898ad25762407373"
    ASKHOST = "http://open.pc120.com"
    ASKTYPE = "/phish/?"
    function checkForValidUrl(uri)
        ngx.say("uri:\t",uri)
        crtURI = ngx.encode_base64(uri)
        timestamp = ngx.now()
        signbase = ASKTYPE .. "appkey=" .. APPKEY .. "&q=" .. crtURI .. "&timestamp=" .. timestamp
        sign = ngx.md5(signbase .. SECRET)
        return ASKHOST .. signbase .. "&sign=" .. sign
    end




chk.lua
----------------------

.. code-block:: lua

    -- try openresty easy creat RESTful API srv.
    ngx.req.read_body()
    local method = ngx.var.request_method
    local KSC = require "ksc"

    if method ~= 'POST' then
        ngx.say('pls. only POST chk me;-)')
        local readme = ngx.location.capture("/readme")
        if readme.status == 200 then
            ngx.say(readme.body)
        end
    else
        local data = ngx.req.get_body_data()
        local args = ngx.req.get_post_args()
        local uri = args.uri
        local url = uri --fields[3]
        local chkURI = KSC.checkForValidUrl(url)
        ok, html = KSC._fetch_uri(chkURI)
        if ok then
            local cjson = require "cjson"
            json = cjson.decode(html)
            if 1 == json.success then
                ngx.log(ngx.INFO,"\n\tKCS say:  ",html,"\n\t")
                ngx.say("KCS /phish?:\t", KSC.PHISHTYPE[tostring(json.phish)])
            else
                ngx.say("KCS say:\t",html)
            end
        end
    end


