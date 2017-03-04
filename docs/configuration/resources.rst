资源数据配置
==================

在系统中，你首先需要决定那些资源你想保护。
这可能是用户的身份信息，比如：简介信息，邮件地址等， 或者要访问的APIs

..注意:: 你可以使用C#对象模型声明资源 - 或者从数据库中装载这些资源.  ``IResourceStore`` 的具体实现处理技术细节. 本教程我们使用简化的内存实现类来声明资源.

添加一个身份资源
^^^^^^^^^^^^^^^^^^^^^^^^^^^
身份资源是诸如用户ID，姓名或者用户邮件地址这样的数据。
一个身份资源有系统唯一名字，你可以给这个名字任意指定声明类型。 这些声明会被包括在身份令牌中， 传给用户。
客户将使用  ``scope`` 参数来请求访问一个身份资源.

OpenID 连接规格指定了一系列 `标准 <https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims>`_ 身份资源.
最小需求是，你需要给每个用户提供一个唯一编号 - 也被称为 subject id.
这是通过暴露叫做``openid``的标准身份资源来实现::

    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        return new List<IdentityResource>
        {
            new IdentityResources.OpenId()
        };
    }

这个 `IdentityResources` 类支持所有规格中定义的范围（资源）比如(openid, email, profile, telephone, and address).
如果想支持所有的标准资源，你可以把这些标准资源加入到身份资源列表中::

    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        return new List<IdentityResource>
        {
            new IdentityResources.OpenId(), 
            new IdentityResources.Email(),
            new IdentityResources.Profile(),
            new IdentityResources.Telephone(),
            new IdentityResources.Address()
        };
    }

添加用户自定义身份资源
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
你也可以定义自己的身份资源. 创建一个 `IdentityResource` 类, 提供一个身份资源名称，愿意的话还可以提供显示名称及描述，以及定义请求这个资源的时候，那些用户声明需要包括在身份令牌中::
 
    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        var customProfile = new IdentityResource(
            name: "custom.profile",
            displayName: "Custom profile",//可以不提供，这个不是必须的
            claimTypes: new[] { "name", "email", "status" });//用户请求自定义的身份资源的时候，将返回这些声明

        return new List<IdentityResource>
        {
            new IdentityResources.OpenId(),
            new IdentityResources.Profile(),
            customProfile
        };
    }

请参见 :ref:`reference <refIdentityResource>` 节了解更多的身份资源的设置。

添加API资源
^^^^^^^^^^^^^^^^^^^^^^
为了让客户申请APIs的访问令牌，你需要定义一个API资源, 比如::

为了得到APIs的访问令牌，你也需要把他们注册成一个范围(scope)。 这个范围(scope)的类型是`Resource`::

    public static IEnumerable<ApiResource> GetApis()
    {
        return new[]
        {
            // simple API with a single scope (in this case the scope name is the same as the api name)
            new ApiResource("api1", "Some API 1"),
            
            // expanded version if more control is needed
            new ApiResource
            {
                Name = "api2",
                
                // secret for using introspection endpoint
                ApiSecrets =
                {
                    new Secret("secret".Sha256())
                },

                // include the following using claims in access token (in addition to subject id)
                UserClaims = { JwtClaimTypes.Name, JwtClaimTypes.Email }
                },

                // this API defines two scopes
                Scopes =
                {
                    new Scope()
                    {
                        Name = "api2.full_access",
                        DisplayName = "Full access to API 2",
                    },
                    new Scope
                    {
                        Name = "api2.read_only",
                        DisplayName = "Read only access to API 2"
                    }
                }
            }
        };
    }

请参看 :ref:`reference <refApiResource>` 节关于API资源的设置细节.
