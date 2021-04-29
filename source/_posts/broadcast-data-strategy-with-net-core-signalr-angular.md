---
title: Broadcast data strategy with .NET Core, SignalR & Angular
date: 2019-05-06 15:40:00
tags: angular, c#, c-sharp, signalr
---


In today’s post, we’re going to discuss an increasingly common requirement of tasks running within a web application. Depending upon your specialism you can think of these as service works, or simply background tasks. For this demo we’re going to focus on our web application continuously running a task which emits I/O data for the duration of the app’s life time. This data can then be consumed by another service, since our task needs to continuously run, it’s suited for more of a observable pattern on the client side, so we can subscribe and handle new data accordingly. In this scenario, we’re going to use SignalR to trigger invocation to any clients connected.

First, you’ll need to do a small amount of configuration, I won’t elaborate on this since it’s fairly straight forward
<escape><!-- more --></escape>

### Startup.cs 

Add the following to your `ConfigureServices(IServiceCollection services)`

`services.AddSignalR();`
Note: In Core 3, you won’t need this call, as the Core Team are [tidying up some of the service extensions](https://twitter.com/davidfowl/status/1123603857831890944)

Next, you’ll want to add an endpoint for your hub in your `Configure(IApplicationBuilder app, IHostingEnvironment env)`

{% codeblock lang:csharp %}
app.UseSignalR(routes =>
{
    routes.MapHub<ChangeDataHub>("/changes");
});
{% endcodeblock %}
With the addition of the `ChangeDataHub` you’ll need to create a class with inherits from the base class Hub, we’ll leave this blank.

{% codeblock lang:csharp %}
public class ChangeDataHub: Hub
{
}
{% endcodeblock %}

Good so far? Next we’ll need a Listener service which will manage the `HubManager` allowing us to send invocation messages to anything which has subscribed to our service.

But, before this we’ll need to think about how we’re going to use our listener service. With .NET Core 1 & 2, Microsoft provided a `IWebHost` however, with the addition of 2.1. a new interface IHost was introduced. `IHost` is similar to `IWebHost` allowing hosted services, DI etc, however it’s more lightweight by the fact it’s designed for background tasks that aren’t used over HTTP. Since we’ll be transmitting our data via web sockets, this is ideal and the right fit for our project.

Registering a host is easy in .NET Core, within our `Startup.cs`, we’ll want to create a singleton instance of our host, we’ll be implementing the `IHostedService` which is a great interface provide just two method of type `Task` to implement, below is the interface:

{% codeblock lang:csharp %}
namespace Microsoft.Extensions.Hosting
{
    //
    // Summary:
    //     Defines methods for objects that are managed by the host.
    public interface IHostedService
    {
        //
        // Summary:
        // Triggered when the application host is ready to start the service.
        Task StartAsync(CancellationToken cancellationToken);
        //
        // Summary:
        // Triggered when the application host is performing a graceful shutdown.
        Task StopAsync(CancellationToken cancellationToken);
    }
}
{% endcodeblock %}

Plugging in our implementation of the IHostedService as described above:

{% codeblock lang:csharp %}
public void ConfigureServices(IServiceCollection services)
{
      services.AddSingleton<Microsoft.Extensions.Hosting.IHostedService, ChangeDataListenerService>();
}
{% endcodeblock %}

Now it’s time to implement our interface, like most tasks we don’t want to simply run this once, but on an incremental basis. To achieve this we’ll use the static method Task.Delay which will take our delay value.

{% codeblock lang:csharp %}
private readonly HubLifetimeManager<ChangeDataHub> _hubManager;
private readonly ILogger<DefaultLogger> _logger;
private readonly TimeSpan _delay = TimeSpan.FromSeconds(1);
public ChartListenerService(
    HubLifetimeManager<ChangeDataHub> hubManager,
    ILogger<DefaultLogger> logger)
{
    _hubManager = hubManager;
    _logger = logger;
}
public async Task StartAsync(CancellationToken cts)
{
    _logger.LogInformation("Listener is starting..");
    while (!cts.IsCancellationRequested)
    {
        _logger.LogInformation("Fetching updates..");
        var data = DataManager.Get();
        await _hubManager.SendAllAsync("TransferData", new object[] { data });
        await Task.Delay(_delay, cts);
    }
    _logger.LogInformation("Listener has completed..");
}
{% endcodeblock %}

That’s our basic implementation completed. It should be noted that if you’re using a repository which for instance uses a `DbContext` this won’t work, since we’ll be creating a DbContext instance that can become a singleton, which can lead to a whole host of problems.

The solution to this problem is injecting a `IServiceScopeFactory` instance and use when we need a scoped instance of a service.

{% codeblock lang:csharp %}
private readonly HubLifetimeManager<ChangeDataHub> _hubManager;
private readonly ILogger<DefaultLogger> _logger;
private readonly TimeSpan _delay = TimeSpan.FromSeconds(1);
private readonly IServiceScopeFactory _scopeFactory;
public ChartListenerService(
    HubLifetimeManager<ChangeDataHub> hubManager,
    ILogger<DefaultLogger> logger,
  IServiceScopeFactory scopeFactory)
{
    _hubManager = hubManager;
    _logger = logger;
  _scopeFactory = scopeFactory;
}
public async Task StartAsync(CancellationToken cts)
{
    _logger.LogInformation("Listener is starting..");
    while (!cts.IsCancellationRequested)
    {
  // Scoped service generation, resolve and dependencies in here
    using (var scope = _scopeFactory.CreateScope())
    {
  
        _logger.LogInformation("Fetching updates..");
    var dataManager = scope.ServiceProvider.GetRequiredService<DataManager>();
    
        await _hubManager.SendAllAsync("TransferData", new object[] { dataManager.Get() });
  }
        await Task.Delay(_delay, cts);
    }
    _logger.LogInformation("Listener has completed..");
}
{% endcodeblock %}

This also means we don’t have to constructor DI all our service dependencies, a good thing in a larger, real world example.

In the next post, I’ll go over how to connect and consume this information on the client side.