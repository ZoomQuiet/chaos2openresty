.. include:: ../LINKS.rst


整备
==============
只要明确一些 `Lua`_ 的`基本形式` 80% 的实际编程就可以混进去了 `!-)` :


.. sidebar:: 提示
    :subtitle: 教程

    lua-users.org `维基 <http://lua-users.org/wiki/FindPage>`_ 

    中搜索 `tutorial`
    
    大约几十篇,有空的话,真心应该看,,



- 基本语法

.. code-block:: lua

    -- 单行注释
    --[[
        多行
        注释
    ]]
    a="hollo"
    b= a .. 1   -- 字串连接,连接数字会自动转换类型
    b,a = a,b   -- 巨爽直的变量交换 

    c=0
    if 1 ~= c   -- 不等于?
        print "Yes"
    else
        print "No"
    end


    function d(e,f)
        return e,f,e*f
    end

    A1,A2,result = d(2,3)
    print(A1,A2,result)


- 基本数据

    - 数字,字串,布尔 基本和其它脚本语言类同
    - `nil` ~ 空值
    - 特殊的是 `function` 也是基本数据类型!
    - 关系表,嗯嗯嗯,就是Python 里的字典吼


.. code-block:: lua

    T1 ={ 10,  -- 相当于 [1] = 10
        ,[100] = 40,
        ,John={  -- 如果你原意，你还可以写成：["John"] =
            Age=27   -- 如果你原意，你还可以写成：["Age"] =27
            ,Gender=Male   -- 如果你原意，你还可以写成：["Gender"] =Male
            }
        ,20  -- 相当于 [2] = 20
    }






