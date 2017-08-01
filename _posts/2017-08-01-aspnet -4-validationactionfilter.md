
## ValidationActionFilter ##

Add a filter to validate the model state for any action.

```
public class ValidationActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(HttpActionContext actionContext)
    {
        var modelState = actionContext.ModelState;

        if (!modelState.IsValid)
            actionContext.Response = actionContext.Request
                    .CreateErrorResponse(HttpStatusCode.BadRequest, modelState);

        base.OnActionExecuting(actionContext);
    }

    public override Task OnActionExecutingAsync(HttpActionContext actionContext, CancellationToken cancellationToken)
    {
        var modelState = actionContext.ModelState;

        if (!modelState.IsValid)
            actionContext.Response = actionContext.Request
                    .CreateErrorResponse(HttpStatusCode.BadRequest, modelState);

        return base.OnActionExecutingAsync(actionContext, cancellationToken);
    }
}
```

Add this to the WebApiConfig:

```
public static class WebApiConfig
{

    public static void Register(HttpConfiguration config)
    {

        // Standard configuration code here...

        config.Filters.Add(new ValidationActionFilter());

    }
}
```


