---
layout: post
title: Enabling CORS in Dotnet Core
description: Enabling CORS forin sptnet core application across all the controllers globally
author: bitsmonkey
tags: CORS, dotnetcore, webapi
---

<br/>

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) - Cross Origin Resource Sharing, basically helps our application from Cross site scripting attacks. This will restrict a website from access or sending data to another origin( web app).

![Photo by Banter Snaps on Unsplash](/../img/corsdotnetcore.jpg){:.coverimage}

This post will help look into enabling CORS in dotnet core application. Every POST request before posting the actual requests will send an [Preflighted requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Preflighted_requests) using the OPTION method like the below sample.

```
            OPTIONS {{apiendpoint}}/api/web/incoming HTTP/1.1
            Access-Control-Request-Method: POST
            Origin: http://localhost:4200
```

This is to check if the actual request method in this case POST is allowed or not.

So to enable this in dotnet core we use [AddCors](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvccorsmvccorebuilderextensions.addcors?view=aspnetcore-2.2) of the IServiceCollection in `StartUp.cs` like below.

```csharp
            services.AddCors(options =>
            {
                options.AddPolicy("CorsPolicy",
                    builder => builder.AllowAnyOrigin()
                    .AllowAnyMethod()
                    .AllowAnyHeader()
                    .AllowCredentials());
            });
```

We have created the policy as per our requirements now need to use this in our MVC application, make sure you add `UseCors` before `UseMvc`.

```csharp
            app.UseCors("CorsPolicy");
            app.UseMvc();
```

Now your application CORS enabled. This approach is to add CorsPolicy globally. you also refer this [guide](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-2.2#enable-cors-with-attributes) to enable CORS using attributes.

- Photo by Banter Snaps on Unsplash