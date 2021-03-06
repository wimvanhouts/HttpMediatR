# HttpMediatR
A library that combines two awesome things: ASP.NET Core & MediatR, and makes your life easier when you're using them together. :two_hearts:

# Status
![Visual Studio Team services](https://img.shields.io/vso/build/rpm1984/b191ce56-b252-49c8-bba2-23e75b32ab0b/3.svg?style=plastic) [![Codecov](https://img.shields.io/codecov/c/github/RPM1984/HttpMediatR.svg?style=plastic)](https://codecov.io/gh/rpm1984/httpmediatr) [![NuGet](https://img.shields.io/nuget/v/HttpMediatR.svg?style=plastic)](https://www.nuget.org/packages/HttpMediatR/) [![NuGet](https://img.shields.io/nuget/dt/HttpMediatR.svg?style=plastic)](https://www.nuget.org/packages/HttpMediatR/)

# Installation
![Imgur](https://i.imgur.com/s68DUuP.png)

# How to use
- Instead of your input model implementing `IRequest`, implement `IHttpRequest`
- Instead of your handler implementing `IRequestHandler`, inherit from `HttpHandler`
- Implement the `HandleAsync` abstract method in your handler
- Make use of the inherited helpers to return the appropriate responses
- Marvel at your awesome code! :star:

See the [sample project](https://github.com/RPM1984/HttpMediatR/tree/master/samples/HttpMediatR.Samples.AspNetCoreMvc) for a demo on the above.

## What problems does this library solve?
### [DRY (Don't repeat yourself)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
If you're using MediatR with ASP.NET Core, you'd probably be writing a lot of code like this:
```
public async Task<IActionResult> Index(Query query, CancellationToken cancellationToken)
{
    var result = await _mediator.Send(query, cancellationToken);
    if (result == null)
    {
        return NotFound();
    }
    return Ok(result);
}
```

Which is fine! But...with this library we can go one better, and make it look like this:
```
public Task<IActionResult> Index(Query query, CancellationToken cancellationToken)
            => _mediator.Send(query, cancellationToken);
```

So that's even better! :sunglasses:. We no longer need to write the boring "get something, see if it exists, return not found or ok depending on that blah blah", and it's just a 1 liner. In fact, you'll see in the source code there is a file called `Router.cs` instead of many controllers. This is because we don't really have controllers anymore..just a router passing on all the resposibility to handle the HTTP request to your MediatR handler, so there's no real need to seperate into multiple controllers (of course, feel free to do so if you like!)

### [Throwing uneccessary exceptions](http://jonskeet.uk/csharp/exceptions.html)
Exceptions are pretty expensive..so ideally we'd like to avoid throwing them.

Here's an example of where you'd need to throw one in your MediatR handler:
```
public class Handler : IRequestHandler<Query, MyModel>
{
    private readonly SomeDbContext _db;

    public Handler(SomeDbContext db) => _db = db;

    protected async Task<MyModel> Handle(Query query,
										 CancellationToken cancellationToken)
    {
        var result = await _db.Get(query.Id, cancellationToken);
        if (result == null)
        {
            throw new Exception("Record not found.");
        }
        return result;
```

So there's a few problems there:
- We're throwing an _exception_ for a 404/not found (is it really an 'exception'?)
- We have to handle that exception elsewhere (e.g in your ASP.NET Core error handler/some global filter)
- We still have to handle returning the _success_ of the response somewhere else (e.g your controller)

Now, we can do this:
```
public class Handler : HttpHandler<Query, MyModel>
{
    private readonly SomeDbContext _db;

    public Handler(SomeDbContext db) => _db = db;

    protected override async Task<HttpResponse<MyModel>> HandleAsync(Query query,
																	 CancellationToken cancellationToken)
    {
        var result = await _db.Get(query.Id, cancellationToken);
        if (result == null)
        {
            return NotFound();
        }
        return Ok(result));
```

So, we're not throwing exceptions anymore uncessarily, and everything to handle that HTTP request is there in the handler.
