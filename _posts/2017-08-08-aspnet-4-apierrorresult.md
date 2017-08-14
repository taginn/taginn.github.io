
## API Error Result ##

Add a predictable error response to the API.

``` csharp 


public class ApiErrorResult : IHttpActionResult
{
    ApiError _value;
    HttpRequestMessage _request;

    public ApiErrorResult(string value, HttpRequestMessage request)
    {
        _value = new ApiError(value);
        _request = request;
    }
    public ApiErrorResult(IEnumerable<string> value, HttpRequestMessage request)
    {
        _value = new ApiError(string.Join(",", value));
        _request = request;
    }
    public Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        var response = new HttpResponseMessage()
        {
            StatusCode = HttpStatusCode.BadRequest,
            Content =  new ObjectContent(typeof(ApiError), _value, _request.GetConfiguration().Formatters.JsonFormatter),
            RequestMessage = _request
        };
        return Task.FromResult(response);
    }
}

```

The error:

``` csharp

public class ApiError
{
    public string Message { get; set; }

    public ApiError()
    {

    }
    public ApiError(string error)
    {
        this.Message = error;
    }
}

```