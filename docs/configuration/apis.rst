保护APIs（Protecting APIs）
===============
使用IdentityServer的访问令牌可以保护APIs接口的访问。非常简单-只需要把令牌验证中间件加入到Asp.net Core的管道中，并且配置IdentityServer的地址和访问范围即可::

    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            app.UseIdentityServerAuthentication(new IdentityServerAuthenticationOptions
            {
                Authority = "https://demo.identityserver.io",//指定identityserver的地址
                AllowedScopes = { "api1" },//指定允许访问的范围(api1)
            });

            app.UseMvc();
        }
    }

中间件可以通过 `nuget <https://www.nuget.org/packages/IdentityServer4.AccessTokenValidation/>`_ 
或者 `github <https://github.com/IdentityServer/IdentityServer4.AccessTokenValidation>`_ 得到.

**如果要用于非Asp.net core的早期.Net框架**

由于早期.net框架(如：NET452）的默认的HTTPS 通讯配置原因，在早期的.Net框架(如：NET452）使用这个中间件在配置令牌验证的metadata endpoint,可能会有运行时的SSL/TLS失败异常。如果遇到这样的异常，可以通过在ServicePointManager的安全协议配置里启用最新版本的TLS避免, 把下面的示例代码放到你API工程的Startup.cs类中来解决::

    #if NET452
        System.Net.ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12 | SecurityProtocolType.Tls11 | SecurityProtocolType.Tls;
    #endif
    
最外层的异常会是类似于下面:
    
    System.InvalidOperationException: IDX10803: Unable to obtain configuration from: 'https://MYWEBSITE.LOCAL/.well-known/openid-configuration'.

内部原始异常(inner exception)会类似于下面的信息:

    System.Security.Authentication.AuthenticationException: A call to SSPI failed, see inner exception. ---> System.ComponentModel.Win32Exception: The client and server cannot communicate, because they do not possess a common algorithm

 