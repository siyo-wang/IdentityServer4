授权端点(Authorize Endpoint)
==================

浏览器可以从授权端点申请令牌或者授权码。
一般来说，主要涉及最终用户的登录认证（包括用户许可(consent)授权范围）

..许可(consent):: 在用户登录后，显示浏览器应用这次申请的权限范围，最终用户可以同意授权或者取消。许可这个动作是可配置的，如果IdentityServer只用于公司项目，这一步一般会省略。

..要点:: IdentityServer支持OpenID Connect和OAuth2 授权请求参数的子集。完整的列表请参看 `这里 <https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest>`_。

``client_id``
    客户标识(必须)。
``scope``
    一个或者多个注册的范围(必须)。
``redirect_uri`` 
    必须是相关客户的允许重定向URI列表中的一个。需要完全匹配。(必须)
``response_type`` 
    ``id_token`` 请求一个身份令牌 (只能获得身份范围(scope))

    ``token`` 请求一个访问令牌 (只能获得资源范围(scope))

    ``id_token token`` 同时请求身份令牌和访问令牌

    ``code`` 请求一个授权码

    ``code id_token`` 同时请求授权码和身份令牌

    ``code id_token token`` 同时请求授权码，身份令牌和访问令牌
    
``response_mode``
    ``form_post`` 以表单POST方式回传令牌,默认是一个编码片段重定向（可选）
``state`` 
    identityserver 将在令牌响应中回传收到的状态(state),
    主要目的就是把请求(客户)和响应(授权服务器)关联起来，以便对跨站请求伪造(CSRF)/重放攻击进行保护。(推荐）
``nonce`` 
    identityserver 将把nonce的值放在身份令牌中回传，用于对重放攻击进行保护。 

    *隐式授权必须要把nonce包含在身份令牌中。*
``prompt``
    ``none`` 在授权时，不显示界面。如果做不到（比如用户必须登录或者授权），那么显示一个错误信息
    
    ``login`` 即使用户已经登录，会话还在有效期内， 也显示登录界面。
``code_challenge``
    发送授权码争议(详情请查阅Proof Key for Code Exchange(PKCE)
``code_challenge_method``
    ``plain`` 使用明码发送授权码争议。（不推荐）
    ``S256`` 使用SHA256散列授权码争议。
``login_hint``
    can be used to pre-fill the username field on the login page
``ui_locales``
    gives a hint about the desired display language of the login UI
``max_age``
    if the user's logon session exceeds the max age (in seconds), the login UI will be shown
``acr_values``
    allows passing in additional authentication related information - identityserver special cases the following proprietary acr_values:
        
        ``idp:name_of_idp`` bypasses the login/home realm screen and forwards the user directly to the selected identity provider (if allowed per client configuration)
        
        ``tenant:name_of_tenant`` can be used to pass a tenant name to the login UI

**Example**

::

    GET /connect/authorize?
        client_id=client1&
        scope=openid email api1&
        response_type=id_token token&
        redirect_uri=https://myapp/callback&
        state=abc&
        nonce=xyz 

(URL encoding removed, and line breaks added for readability)


IdentityModel
^^^^^^^^^^^^^
You can programmatically create URLs for the authorize endpoint using the `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ library::

    var request = new AuthorizeRequest(doc.AuthorizeEndpoint);
    var url = request.CreateAuthorizeUrl(
        clientId:     "client",
        responseType: OidcConstants.ResponseTypes.CodeIdToken,
        responseMode: OidcConstants.ResponseModes.FormPost,
        redirectUri: "https://myapp.com/callback",
        state:       CryptoRandom.CreateUniqueId(),
        nonce:       CryptoRandom.CreateUniqueId());

..and parse the response::

    var response = new AuthorizeResponse(url);

    var accessToken = response.AccessToken;
    var idToken = response.IdentityToken;
    var state = response.State;