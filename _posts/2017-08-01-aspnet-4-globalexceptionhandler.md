
## Global Exception Handler ##

Created to catch any unhandled exceptions thrown by ASP.NET.

``` csharp 
public class GlobalExceptionHandler : ExceptionHandler
{
    protected static readonly ILogger Logger = LogManager.GetCurrentClassLogger();

    public override Task HandleAsync(ExceptionHandlerContext context, CancellationToken cancellationToken)
    {
        CreateBadRequestContext(context);

        return Task.FromResult(0);
    }

    public override void Handle(ExceptionHandlerContext context)
    {
        CreateBadRequestContext(context);
    }

    private static void CreateBadRequestContext(ExceptionHandlerContext context)
    {
        Logger.Log(LogLevel.Error, context.Exception, "Unhandled exception thrown");

        var result = new HttpResponseMessage(HttpStatusCode.BadRequest)
        {
            Content = new StringContent("We were unable to complete your request"),
            ReasonPhrase = "An exception occurred."
        };

        context.Result = new BadRequestResult(context.Request, result);
    }
}
```

Related BadRequestResult:

``` csharp 
public class BadRequestResult : IHttpActionResult
{
    private HttpRequestMessage _request;
    private HttpResponseMessage _httpResponseMessage;

    public BadRequestResult(HttpRequestMessage request, HttpResponseMessage httpResponseMessage)
    {
        _request = request;
        _httpResponseMessage = httpResponseMessage;
    }

    public Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        return Task.FromResult(_httpResponseMessage);
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

        config.Services.Replace(typeof(IExceptionHandler), new GlobalExceptionHandler());

    }
}
```
