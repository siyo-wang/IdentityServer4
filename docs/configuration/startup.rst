启动
=======

IdentityServer 是一系列中间件和服务的组合.
所有这些配置工作都在startup类中。

配置服务
^^^^^^^^^^^^^^^^^^^^
首先，你要把IdentityServer加入到你的依赖注入系统中::

    public void ConfigureServices(IServiceCollection services)
    {
        var builder = services.AddIdentityServer();
    }
在调用这个配置的时候，你也可以传人一些选项，具体请参看 :ref:`here <refOptions>` 了解选项的具体信息。

这个方法调用会返回一个builder对象，builder对象有很多方便的扩展方法来添加更多的附加服务。

**密钥证书**

* ``AddSigningCredential``
    添加一个密钥签名服务，用于给各种令牌生产，验证服务提供证书信息。
    你可以传入 ``X509Certificate2``, ``SigningCredential``或者证书仓库中一个证书的引用信息。
* ``AddTemporarySigningCredential``
    启动时创建临时证书，这个主要用于开发阶段。避免在开发早期就必须去申请证书，降低开发门槛。
* ``AddValidationKeys``
    为验证令牌服务提供一些密钥。主要用于内部令牌验证服务，这个密钥会出现在发现文档(discovery document)中。
    主要用于密钥更换的阶段，旧密钥已经过期了，但是可能还有一些客户使用旧密钥处理令牌，所以旧密钥还是要在令牌验证服务中使用。

**配置信息内存存储**

为了方便开发，identityserver提供了各种“硬编码(in-memory)”配置函数，用于配置客户，用户，资源，范围，声明等数据的。
这些“硬编码(in-memory)”配置信息可以直接写在identityserver的托管应用中，也可以写在配置文件或者数据库中，用代码一次性读取。
按照设计，这些配置信息都是在identityserver的托管应用启动的时候一次性创建的，换句话说，你可以认为这些配置信息是准静态的，在运行时，是不会变化的。

设计这些配置函数主要目的是用于功能验证，开发或者技术选型，这些情况下不需要动态和数据库交互来获取，创建，维护配置信息（如 用户，客户信息），
使用静态配置信息又快又好。如果某些配置信息在生产环境下基本不会变化，比如我们就支持一个固定的MVC客户，那么这种“硬编码”配置也可以用于生产环境。
或者我们生产环境重启影响很小，某些配置发生改变，比如需要加一个客户，我们重启生产环境就好；这种情况下也可以用“硬编码”配置函数。

* ``AddInMemoryClients``
    注册 ``IClientStore`` 和 ``ICorsPolicyService`` 基于内存数据集合的``Client``配置数据的实现类。
* ``AddInMemoryIdentityResources``
    注册 ``IResourceStore`` 基于内存数据集合的``IdentityResource``配置数据的实现类。
* ``AddInMemoryApiResources``
    注册``IResourceStore`` 基于内存数据集合的``ApiResource``配置数据的实现类。

**Test stores**

这个 ``TestUser`` 类在IdentityServer中构建了一个用户， 他们的用户名密码信息和对应的声明。
这个``TestUser``设计目的和“硬编码”信息一样，主要用户功能验证，开发或者技术选项。
``TestUser`` 这任何情况下都不建议在生产环境中使用。

* ``AddTestUsers``
    用``TestUser``对象集合注册 ``TestUserStore`` 。
    ``TestUserStore`` 也用户快速入门程序。
    也会注册 ``IProfileService`` 和 ``IResourceOwnerPasswordValidator``的实现.

**附加服务**

* ``AddExtensionGrantValidator``
    添加 ``IExtensionGrantValidator`` 用来来扩展授权(grant)的实现。

* ``AddSecretParser``
    添加 ``ISecretParser`` 用于分析客户或者API资源凭据的实现。

* ``AddSecretValidator``
    添加 ``ISecretValidator`` 用于基于存储的凭据来验证客户或者API资源凭据的实现。
    
* ``AddResourceOwnerValidator``
    添加 ``IResourceOwnerPasswordValidator`` 在资源拥有者（用户）密码凭据授权类型下，验证用户凭据的实现。
    
* ``AddProfileService``
    添加 ``IProfileService`` 连接自定义用户简介(profile)信息存储的实现。
    这个 ``DefaultProfileService`` 类提供了仅依赖认证cookie作为唯一声明来源来发布令牌的默认实现。
    
* ``AddAuthorizeInteractionResponseGenerator``
    添加 ``IAuthorizeInteractionResponseGenerator``一个实现类，可以实现自定义的授权逻辑来显示自定义的错误界面，登录界面，授权页面或者任何其他自定义页面。
    ``AuthorizeInteractionResponseGenerator`` 提供类默认实现。建议扩展这个默认实现来快速扩充你要的功能。
    
* ``AddCustomAuthorizeRequestValidator``
    添加 ``ICustomAuthorizeRequestValidator``实现类，来自定义验证授权请求参数。

* ``AddCustomTokenRequestValidator``
    添加 ``ICustomTokenRequestValidator``实现类，来自定义验证令牌请求参数。
    
**缓存**

IdentityServer需要频繁使用客户和资源数据，如果这些数据需要从数据库或者其它外部存储中读取，反复读取这些数据代价可能太大了。缓存就是一个提高性能的银弹。
在使用“硬编码(in-memory)”服务保存这些数据时，缓存没有什么用。

* ``AddClientStoreCache``
    注册``IClientStore`` 装饰类， 用于在内存中维护``客户``信息。缓存时间由``Caching``的配置选项``IdentityServerOptions``确定。

* ``AddResourceStoreCache``
    注册``IResourceStore``装饰类，用于在内存中维护``IdentityResource`` 和 ``ApiResource`` 信息。缓存时间由``Caching``的配置选项``IdentityServerOptions``确定。

如有必要，可以进一步扩展缓存:

默认缓存依赖于``ICache<T>`` 实现.
如果需要为特定配置对象自定义缓存行为，可以在依赖注入系统替换对应的``ICache<T>``实现。
 
默认的``ICache<T>``实现依赖于.NET提供的``IMemoryCache`` 接口 (以及 ``MemoryCache`` 实现)。
如果需要自定义内存缓存行为，可以在依赖注入系统替换对应的``IMemoryCache``实现。

管道中的配置
^^^^^^^^^^^^^^^^^^^^^^^^
你需要把IdentityServer加到管道中::

    public void Configure(IApplicationBuilder app)
    {
        app.UseIdentityServer();
    }

这个中间件没有附加的配置选项。

注意管道中的配置顺序很重要。
如果你要在界面中实现登录窗口，那么你必须在UI 框架前添加IdentityServer。
