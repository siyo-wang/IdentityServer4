自审端点(Introspection Endpoint)
===============================

自审实现了 `RFC 7662 <https://tools.ietf.org/html/rfc7662>`_.

自审端点可以用来验证参考令牌 (或者 客户端无法支持的JWT（没有对应的解码库）).
自审端点需要认证方使用范围密码.

例子
^^^^^^^

::


    POST /connect/introspect
    Authorization: Basic xxxyyy

    token=<token>


成功的响应状态码为200并带有一个活跃或者不活跃令牌::


    {
        "active": true,
        "sub": "123"
    }


未知或者过期令牌将被标记为不活跃::


    {
        "active": false,
    }


无效的请求将返回状态码400或者无权限状态码401。

IdentityModel
^^^^^^^^^^^^^
可以通过 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 库来访问自审端点::

    var introspectionClient = new IntrospectionClient(
        doc.IntrospectionEndpoint,
        "scope_name",
        "scope_secret");

    var response = await introspectionClient.SendAsync(
        new IntrospectionRequest { Token = token });

    var isActice = response.IsActive;
    var claims = response.Claims;
