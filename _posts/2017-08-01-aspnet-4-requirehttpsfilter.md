
## RequireHttpsFilter ##

Add a filter to require HTTPS  - but not in development.

``` csharp 
public class RequireHttpsFilter : IAuthenticationFilter
{
    public bool AllowMultiple
    {
        get { return true; }
    }

    public Task AuthenticateAsync(HttpAuthenticationContext context,
                                        CancellationToken cancellationToken)
    {
        if (context.Request.RequestUri.Scheme != Uri.UriSchemeHttps)
        {
            context.ActionContext.Response = new HttpResponseMessage(
                                    System.Net.HttpStatusCode.Forbidden);
        }
        return Task.FromResult<object>(null);
    }

    public Task ChallengeAsync(HttpAuthenticationChallengeContext context,
                                        CancellationToken cancellationToken)
    {
        return Task.FromResult<object>(null);
    }
}
```

Add this to the WebApiConfig:

``` csharp 
public static class WebApiConfig
{

    public static void Register(HttpConfiguration config)
    {

        // Standard configuration code here...

#if !DEBUG
            ///

            /// See OwinConfig.ConfigureAuth - > AllowInsecureHttp. This may be redundant.
            ///
            config.Filters.Add(new RequireHttpsFilter());
#endif

    }
}
```

OwinConfig.ConfigAuth:

``` csharp 
public class OwinConfig
{
    private const string keyAccessTokenExpireMinutes = "accessTokenExpireMinutes";
    private const string keyAllowInsecureHttp = "allowInsecureHttp";
    private static readonly Logger Logger = LogManager.GetCurrentClassLogger();

    public static OAuthAuthorizationServerOptions OAuthOptions { get; private set; }

    public static string PublicClientId { get; private set; }

    public static IDataProtectionProvider DataProtectionProvider { get; private set; }

    public void Configuration(IAppBuilder app)
    {
        app.UseCors(CorsOptions.AllowAll);
        ConfigureAuth(app);
        ConfigOnAppDisposing(app);
        Logger.Info("Owin configured.");
    }

    public void ConfigureAuth(IAppBuilder app)
    {
        var identityConnectionStringName = ConfigurationManager.ConnectionStrings[Constants.IdentityConnectionStringName].ConnectionString;
        app.CreatePerOwinContext(() => IdentityDataContext.Create(identityConnectionStringName));
        app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
        app.CreatePerOwinContext<ApplicationRoleManager>(ApplicationRoleManager.Create);

        DataProtectionProvider = app.GetDataProtectionProvider();

        var accessTokenExpireMinutes = Convert.ToInt32(ConfigurationManager.AppSettings.Get(keyAccessTokenExpireMinutes) ??
            Common.Constants.DefaultAccessTokenExpireMinutes);
        var allowInsecureHttp = true;
#if !DEBUG

        ///
        /// See WebApiConfig Register filter RequireHttpsFilter. This may be redundant.
        ///
            allowInsecureHttp = !string.IsNullOrEmpty(ConfigurationManager.AppSettings.Get(keyAllowInsecureHttp)) ?
            Convert.ToBoolean(ConfigurationManager.AppSettings.Get(keyAllowInsecureHttp)) : true;
#endif

        // Configure the application for OAuth based flow
        PublicClientId = "self";
        OAuthOptions = new OAuthAuthorizationServerOptions
        {
            TokenEndpointPath = new PathString("/login"),
            Provider = new ApplicationOAuthProvider(PublicClientId),
            AccessTokenExpireTimeSpan = new TimeSpan(0, accessTokenExpireMinutes, 0),
            AllowInsecureHttp = allowInsecureHttp
        };

        // Enable the application to use bearer tokens to authenticate users
        app.UseOAuthBearerTokens(OAuthOptions);
    }

    private void ConfigOnAppDisposing(IAppBuilder app)
    {
        var properties = new AppProperties(app.Properties);
        var token = properties.OnAppDisposing;
        if (token != CancellationToken.None)
        {
            token.Register(() => { Logger.Info("Application disposed."); });
        }
    }
}

```

