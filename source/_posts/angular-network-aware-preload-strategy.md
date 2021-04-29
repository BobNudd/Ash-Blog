---
title: Angular Network Aware Preload Strategy
date: '12-22-2019'
---

### Lazy Loading Routes

As your app becomes larger, with different modules to group feature components, a common approach is to lazy load
each module in your main routing configuration. Angular recommends this from the start, since it's a simple yet scalable 
pattern to load your projects dependencies once the `route` is activated.
<escape><!-- more --></escape>
You have likely seen or implemented this pattern in the `app-routing.module.ts` like below:

{% codeblock lang:typescript %}
import { NgModule } from '@angular/core';
import { RouterModule, Routes }

const routes: Routes = [
            {
                path: 'index',
                loadChildren: 'src/app/modules/home/home.module#HomeModule'
            },
            {
                path: 'shop',
                loadChildren: 'src/app/modules/shop/shop.module#ShopModule'
            }
];
{% endcodeblock %}

or 

{% codeblock lang:typescript %}
import { NgModule } from '@angular/core';
import { RouterModule, Routes }

const routes: Routes = [
            {
                path: 'index',
                loadChildren: () => import('src/app/modules/home/home.module').then(m => m.HomeModule) }
            },
            {
                path: 'users',
                loadChildren: () => import('src/app/modules/user/user.module').then(m => m.UserModule) }
            }
];
{% endcodeblock %}

Note: the second example is new as of Angular 8, and is a [widely supported](https://caniuse.com/#feat=es6-module-dynamic-import) method of importing modules.

The above examples allow the module to be loaded in when they're requested via accessing the `route` tied to the module.

### Eager Loading

The standard method of preloading any specified modules as above is to use the `PreloadAllModules` strategy. This quite simple loads all modules that have been specified in our `routes` array. The advantage of this method is a better user experience since there isn't any time lag between an end user clicking a route e.g. `/users` and the module having to be downloaded.

{% codeblock lang:typescript %}
imports: [
  ... // omitted for brevity
  RouterModule.forRoot(routes,
    { preloadingStrategy: PreloadAllModules })
]
{% endcodeblock %}

