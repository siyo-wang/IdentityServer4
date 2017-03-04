客户配置(Defining Clients)
================

客户就是那些需要从identityserver申请令牌的应用程序。

认证服务的客户的细节有很多不同的解释，一般来说，我们认为客户有下面共同的特点（设置）:

* 一个唯一的客户ID
* 一个密钥（如果需要）
* 允许和令牌服务交互的方式(也叫授权类型)
* 一个网络地址，用于接收身份(identity)或者访问令牌信息(也叫重定向URI)
* 这个客户允许访问的范围(也被称为资源)列表


..注意:: 在运行时，客户是通过``IClientStore``的实现来获取的. 客户信息可以从任意的数据源获取，如配置文件或者数据库。本教程我们使用内存版的客户信息存储实现.你可以通过``AddInMemoryClients``de ``AddInMemoryClients``扩展方法来应用内存版的客户存储实现  


用于服务器到服务器通讯用的客户
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
这种场景中没有交换用户参与 - 一个服务（也就是客户）请求和一个API(也叫范围）通讯::

    public class Clients
    {
        public static IEnumerable<Client> Get()
        {
            return new List<Client>
            {
                new Client
                {
                    ClientId = "service.client",                    
                    ClientSecrets = { new Secret("secret".Sha256()) },//密码

                    AllowedGrantTypes = GrantTypes.ClientCredentials,//客户身份信息登录，也就是用客户id和secret认证客户。
                    AllowedScopes = { "api1", "api2.read_only" }
                }
            };
        }
    }

基于浏览器的Javascript客户（如：单页面应用） 请求用户认证、委托访问及API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
这种客户使用所谓的隐式流(implicit flow)，从javascript请求身份信息和访问令牌::

    var jsClient = new Client
    {
        ClientId = "js",
        ClientName = "JavaScript Client",
        ClientUri = "http://identityserver.io",

        AllowedGrantTypes = GrantTypes.Implicit,
        AllowAccessTokensViaBrowser = true,

        RedirectUris =           { "http://localhost:7017/index.html" },
        PostLogoutRedirectUris = { "http://localhost:7017/index.html" },
        AllowedCorsOrigins =     { "http://localhost:7017" },

        AllowedScopes = 
        {
            IdentityServerConstants.StandardScopes.OpenId,
            IdentityServerConstants.StandardScopes.Profile,
            IdentityServerConstants.StandardScopes.Email,
            
            "api1", "api2.read_only"
        }
    };

.. _startClientsMVC:

服务端Web应用（如：MVC）客户，请求认证和代理API访问
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
交互式的服务端客户（或者 原生的桌面/移动应用）使用混合流(hybrid flow)。
这种流处理提供最好的安全性，因为访问令牌只在后台对后台的调用中出现，访问令牌不会出现在最终用户的电脑上。（这种方式也提供刷新令牌）::
 
    var mvcClient = new Client
    {
        ClientId = "mvc",
        ClientName = "MVC Client",
        ClientUri = "http://identityserver.io",

        AllowedGrantTypes = GrantTypes.Hybrid,
        AllowOfflineAccess = true,
        ClientSecrets = { new Secret("secret".Sha256()) },
        
        RedirectUris =           { "http://localhost:21402/signin-oidc" },
        PostLogoutRedirectUris = { "http://localhost:21402/" },
        LogoutUri =                "http://localhost:21402/signout-oidc",

        AllowedScopes = 
        {
            IdentityServerConstants.StandardScopes.OpenId,
            IdentityServerConstants.StandardScopes.Profile,
            IdentityServerConstants.StandardScopes.Email,

            "api1", "api2.read_only"
        },
    };
