##  Enabling CORS for ASP.NET 4 and Owin ##

Enabling CORS for ASP.NET 4 and Owin 

Packages:
  <package id="Microsoft.AspNet.Cors" version="5.2.3" targetFramework="net461" />
  <package id="Microsoft.Owin.Cors" version="3.1.0" targetFramework="net461" />


OwinConfig.cs:
``` csharp 
using Microsoft.Owin;
using Microsoft.Owin.Cors;
using Owin;


[assembly: OwinStartup(typeof(OwinConfig))]

namespace FeedlotManagement.Api
{
    [ExcludeFromCodeCoverage]
    public class OwinConfig
    {

        public void Configuration(IAppBuilder app)
        {
            app.UseCors(CorsOptions.AllowAll);
        }

    }
}
```