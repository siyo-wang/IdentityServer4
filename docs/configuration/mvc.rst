连接MVC应用
=============================

你可以把identityserver集成到你的MVC应用中，来认证用户和请求访问令牌。

MVC应用一般使用混合流（hybrid flow）使用 :ref:`this <startClientsMVC>` 示例来注册客户.

在MVC应用的startup, 你可以用标准的microsoft ASP.NET OpenID Connect 中间件来连接 identityserver::

    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();//取消微软默认的claims转换，使用原有短名称
            
            app.UseStaticFiles();

            app.UseCookieAuthentication(new CookieAuthenticationOptions
            {
                AuthenticationScheme = "cookies",
                AutomaticAuthenticate = true,
            });

            var oidcOptions = new OpenIdConnectOptions
            {
                AuthenticationScheme = "oidc",
                SignInScheme = "cookies",

                Authority = "https://demo.identityserver.io",
                ClientId = "mvc",
                ClientSecret = "secret",
                ResponseType = "code id_token",
                SaveTokens = true,
                GetClaimsFromUserInfoEndpoint = true,
                 
                TokenValidationParameters = new TokenValidationParameters
                {
                    NameClaimType = "name",
                    RoleClaimType = "role"
                }
            };

            oidcOptions.Scope.Clear();
            oidcOptions.Scope.Add("openid");
            oidcOptions.Scope.Add("profile");
            oidcOptions.Scope.Add("api1");

            app.UseOpenIdConnectAuthentication(oidcOptions);

            app.UseMvcWithDefaultRoutes();
        }
    }
