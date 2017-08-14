
## Add a token header to the response ##

Add a filter to add a token header on the response of any action. A lazy refresh token hack.

``` csharp 

public class AddTokenHeader : ActionFilterAttribute
{
    private const string authTokenKey = "access_token";
    private const string authIssuedKey = ".issued";
    private const string authExpiresKey = ".expires";

    public override void OnActionExecuted(HttpActionExecutedContext context)
    {
        var currentIdentity = context.ActionContext.RequestContext.Principal.Identity as ClaimsIdentity;
        if (currentIdentity.IsAuthenticated && context.Response != null && context.Response.StatusCode == HttpStatusCode.OK)
        {
            var props = new AuthenticationProperties()
            {
                IssuedUtc = DateTime.UtcNow,
                ExpiresUtc = DateTime.UtcNow.Add(new TimeSpan(24, 0, 0)) //Startup.ExpireTimeSpan)
            };
            var ticket = new AuthenticationTicket(currentIdentity, props);
            var accessToken = Startup.OAuthBearerOptions.AccessTokenFormat.Protect(ticket);
            context.Response.Headers.Add(authTokenKey, accessToken);
            context.Response.Headers.Add(authIssuedKey, props.IssuedUtc.Value.ToString("R"));
            context.Response.Headers.Add(authExpiresKey, props.ExpiresUtc.Value.ToString("R"));
        }
    }
}

```