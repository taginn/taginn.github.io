
## API Version Prefix ##

Add a version prefix to the API.

``` csharp 

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

        // Web API routes
        config.MapHttpAttributeRoutes(new CentralizedRoutePrefixProvider("v{" + ROUTEDATA_VALUE_VERSION + ":int}"));

        config.Routes.MapHttpRoute(
            "DefaultApi",
            "{controller}/{id}",
            new { controller = "system", id = RouteParameter.Optional }
        );

    }
}

```

The provider:

``` csharp

public class CentralizedRoutePrefixProvider : DefaultDirectRouteProvider
{
    private readonly string _centralizedPrefix;

    public CentralizedRoutePrefixProvider(string centralizedPrefix)
    {
        _centralizedPrefix = centralizedPrefix;
    }

    protected override string GetRoutePrefix(HttpControllerDescriptor controllerDescriptor)
    {
        var existingPrefix = base.GetRoutePrefix(controllerDescriptor);
        if (existingPrefix == null) return _centralizedPrefix;

        return string.Format("{0}/{1}", _centralizedPrefix, existingPrefix);
    }
}

```
