发现端点(Discovery Endpoint)
===========================

发现端点用来获取认证服务器的元数据。发现端点返回诸如issuer name, key material, supported scopes 等信息。

发现端点位于认证服务的基础地址的 `/.well-known/openid-configuration` 位置, 比如::

    https://demo.identityserver.io/.well-known/openid-configuration

IdentityModel
^^^^^^^^^^^^^
如果想通过代码访问发现端点获取信息，请使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 库::

    var discoveryClient = new DiscoveryClient("https://demo.identityserver.io");
    var doc = await discoveryClient.GetAsync();

    var tokenEndpoint = doc.TokenEndpoint;
    var keys = doc.KeySet.Keys;

出于安全性考虑， 发现端点客户端有一个可配置的验证策略，默认检查下列规则:

* 必须在发现端点以及其它所有的协议端点上使用HTTPS
* 下载的文档中发布者名称必须和授权指定的名称一样（事实上必须存在于发现规格文档上）
* 协议的端点必须在授权站点的“下面”---不能是一个不同的服务器或者URL（多租户OPs可能会这样干）
* 必须有密钥

如果有特别原因，需要绕过这些限制（比如，开发环境），可以使用下面的代码::

    var client = new DiscoveryClient("http://dev.identityserver.internal");
    client.Policy.RequireHttps = false;
 
    var disco = await client.GetAsync();

顺便说一下： 通过127.0.0.1访问可以不用HTTPS（当然这个也是可配置的）
