##  Enabling HTTPS for ASP.NET 4 ##

Enabling HTTPS for ASP.NET 4

OwinConfig.cs:
``` csharp 
using Microsoft.Owin;
using Microsoft.Owin.BuilderProperties;
using Microsoft.Owin.Cors;
using Microsoft.Owin.Security.DataProtection;
using Microsoft.Owin.Security.OAuth;
using NLog;
using Owin;
using System;
using System.Configuration;
using System.Diagnostics.CodeAnalysis;
using System.Threading;

[assembly: OwinStartup(typeof(OwinConfig))]

namespace FeedlotManagement.Api
{
    [ExcludeFromCodeCoverage]
    public class OwinConfig
    {
        private const string keyAccessTokenExpireMinutes = "accessTokenExpireMinutes";
        private const string keyAllowInsecureHttp = "allowInsecureHttp";
       
        public static OAuthAuthorizationServerOptions OAuthOptions { get; private set; }
        public static string PublicClientId { get; private set; }
        public static IDataProtectionProvider DataProtectionProvider { get; private set; }

        public void Configuration(IAppBuilder app)
        {
            app.UseCors(CorsOptions.AllowAll);
            ConfigureAuth(app);
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

            ///
            /// See WebApiConfig Register filter RequireHttpsFilter. This may be redundant.
            ///
            var allowInsecureHttp = !string.IsNullOrEmpty(ConfigurationManager.AppSettings.Get(keyAllowInsecureHttp)) ?
                Convert.ToBoolean(ConfigurationManager.AppSettings.Get(keyAllowInsecureHttp)) : false;
#if DEBUG
             allowInsecureHttp = true;
#endif

            // Configure the application for OAuth based flow
            PublicClientId = "self";
            OAuthOptions = new OAuthAuthorizationServerOptions
            {
                AllowInsecureHttp = allowInsecureHttp
            };

            // Enable the application to use bearer tokens to authenticate users
            app.UseOAuthBearerTokens(OAuthOptions);
        }

    }
}
```

WebApiConfig.cs:
``` csharp
using Microsoft.AspNet.WebApi.Extensions.Compression.Server;
using Microsoft.Owin.Security.OAuth;
using Newtonsoft.Json.Converters;
using System.Collections.ObjectModel;
using System.Diagnostics.CodeAnalysis;
using System.Net.Http.Extensions.Compression.Core.Compressors;
using System.Web.Http;
using System.Web.Http.ExceptionHandling;
using System.Web.Http.Tracing;

namespace FeedlotManagement.Api
{
    [ExcludeFromCodeCoverage]
    public static class WebApiConfig
    {
        public const string ROUTEDATA_VALUE_VERSION = "version";

        public static Collection<string> SupportedVersions
        {
            get
            {
                return new Collection<string>() { "1" };
            }
        }

        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services
            var xmlFormatter = config.Formatters.XmlFormatter;
            config.Formatters.Remove(xmlFormatter);
            config.Formatters.JsonFormatter.SerializerSettings.Converters.Add(new StringEnumConverter());

            var traceWriter = config.EnableSystemDiagnosticsTracing();
            traceWriter.IsVerbose = true;
            traceWriter.MinimumLevel = TraceLevel.Debug;

            // Configure Web API to use only bearer token authentication.
            config.SuppressDefaultHostAuthentication();
            config.Filters.Add(new HostAuthenticationFilter(OAuthDefaults.AuthenticationType));
            config.Filters.Add(new VersionFilter());
#if !DEBUG

            ///
            /// See OwinConfig.ConfigureAuth - > AllowInsecureHttp. This may be redundant.
            ///
            config.Filters.Add(new RequireHttpsFilter());
#endif
            config.Filters.Add(new ValidationActionFilter());

            // Web API routes
            config.MapHttpAttributeRoutes(new CentralizedRoutePrefixProvider("v{" + ROUTEDATA_VALUE_VERSION + ":int}"));

            config.Routes.MapHttpRoute(
                "DefaultApi",
                "{controller}/{id}",
                new { controller = "system", id = RouteParameter.Optional }
            );

            config.Services.Replace(typeof(IExceptionHandler), new GlobalExceptionHandler());

            #region Compression -- Must be last line

            config.MessageHandlers.Insert(0,
                new ServerCompressionHandler(0, new GZipCompressor(), new DeflateCompressor()));

            #endregion Compression -- Must be last line
        }
    }
}
```


