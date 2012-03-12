.. include:: ../LINKS.rst


好了,已经完成了预期的 :

- 使用 `OpenResty`_ 
- 包装 `金山网址云安全开放API <http://code.ijinshan.com/api/devmore4.html#md1>`_ 为接口服务
- 只要 `curl -d "uri=http://sina.com" localhost:9090/=/chk` 即可返回,金山云的数据!


.. sidebar:: 推荐
    :subtitle: `agentzh <http://agentzh.org/>`_ 妙吼

    - `命名的艺术 <http://agentzh.org/misc/slides/naming/#2>`_
    - `“命名”课回顾 <http://agentzh.org/misc/slides/naming/naming_recap.html#2>`_

    

**但是** :

- 代码很集中,业务逻辑离头部很远,很难看
- 当没有进行 `POST` 请求时,以及各种其它意外请求时,并没智能的捕获,并提示
- 变量名设计的都很挫,需要重构
...

所以,继续折腾...

