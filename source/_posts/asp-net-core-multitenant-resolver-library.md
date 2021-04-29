---
  title: Micro Multi Tenanted Value Resolver Library
  date: 2020-06-07 19:07:00
  tags: c#, multitenant
---

I've been recently dealing with enabling multi tenancy in a full stack application, the back end being ASP.NET Core. When looking into various multi tenant libraries I found them to have one or more characteristics:

* The apis were quite complex, and required lots of documentation parsing
* Difficult to plug in different value resolutions
* Require lots of boilerplate code and implementation of services

Many of these libraries are designed to use configuration files or resolution databases in order to load credentials. I **don't** like this, why?

The reason is simple, *most* credentials don't belong in files such as `appsettings.json`. Instead, these should be stored in a Key Vault which has a host of benefits such as, permission management hardware level encryption and so on. Services such as [Azure Key Vault](https://azure.microsoft.com/en-gb/services/key-vault/) have these benefits.

<escape><!-- more --></escape>

So, more to the point I've written a very, very small library which implements a `Middleware` pattern to essentially push specific credentials based upon a tenant resolution pattern into the current `HttpContext`. The NuGet package can be found [here](https://www.nuget.org/packages/BobNudd.MicroMultiTenant)

The nice thing about Azure Key Vault is all secrets (depending upon access rights) can be loaded into `IConfiguration` on application build.

More information and the source can be found on GitHub:

[https://github.com/BobNudd/MicroMultiTenant](https://github.com/BobNudd/MicroMultiTenant)

