---
layout: post
title: Validating a jwt token and policy-based authorizing
description: Validating a jwt token and policy-based authorizing using jwt token bearer in dotnet core 
author: bitsmonkey
tags: security, dotnet, programming 
---
![token policy based](/img/jwt-policy-based-dotnetcore.jpg){:.coverimage}

## Introduction

There are many ways of securing an API, and using an OAuth token is one among them which is considered to be highly secure. With dotnet core things are tidy and will make implementation easy.

## Generating access token

We will be using service to service authorization use case and grant type as [client credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) to demo validating and authorizing. Now the basic of getting a access token is use your client id and client secret, mash it up and get the base64 version of `clientid:clientsecret` and send it across to the token endpoint as below.

```http
    curl --request POST 
    --url https://tokenprovider/oauth/token
    --header 'authorization: Basic <base64 of clientid:clientsecret>' 
    --header 'content-type: application/x-www-form-urlencoded' 
    --data grant_type=client_credentials 
    --data token_format=jwt 
    --data client_id=27bb5b67-270f-4414-858a-dea819476217 
    --data response_type=token 
    --data 'scope=roles app1.readonly'
```

In the request if you checkout the last param `scope`, we are asking for readonly which we will be using if the client has access to on our API.  
The response you would get will contain `access_token` which will be used when accessing any resource of our API. This is how the response would look like

```json
{
  "access_token": "<token base64 string>",
  "token_type": "bearer",
  "expires_in": 43199,
  "scope": "app1.readonly",
  "jti": "3413de77fcfc4f1f810267df7ff35044"
}
```

## Validating the token

Now that we have the access token lets figure out how to validate this token using [JwtTokenBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/)
Add the below package

`dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 3.1.4`

In you `Startup.cs` add this middleware in `ConfigureServices`

```cs
  services.AddAuthentication(x =>
    {
        x.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        x.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(x =>
    {
        x.Audience = Configuration["Jwt:Audience"];
        x.Authority = Configuration["Jwt:Authority"];
        x.TokenValidationParameters = new TokenValidationParameters
        {                        
            ValidateIssuerSigningKey = true,                                                                        
            ValidIssuer = "https://tokenprovider/oauth/token",
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
        };
    });
```

To get the audience and the authority setting you can check this using [jwt.io token debugger](http://jwt.io). Paste the access token and you will get information about your token on what algorithm is used to sign the issuer address and the audience along with many other info. We will use `aud` and `iss` claims from the token and set the config.

Here is a reminder to add this to `Configure` method. just reminding coz usually I forget and keep wondering.

```cs
  app.UseAuthentication();
  app.UseAuthorization();
```

Now you can use the `[Authorize]` attribute on top of your controller class or action method which makes your API accessible only with valid tokens. Here is an example to try out.

```http
  curl --request 
  GET --url https://localhost:5001/movies
  --header 'authorization: bearer <your access token goes here>'   
```

## Authorizing using policy based approach 

Now that we know how to validate access token, lets extend this and see how to control access over finer methods or classes using [policy-based authorization](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-3.1).

Earlier when retrieving token we had sent a scope saying i need `app1.readonly` which basically meant I need readonly access token to use the api(audience). To limit the access lets add this below code create policies which can be used in any class or method along with `Authorize` attribute.

In `Startup.cs` in `ConfigureServices` method we have create two new policies one with readonly and another with write access.

```cs
  services.AddAuthorization(x =>
      {
          x.AddPolicy("Readonly", 
            policy => policy.RequireClaim("scope", "app1.readonly"));
          x.AddPolicy("Write", 
            policy => policy.RequireClaim("scope", "app1.readwrite"));
      }
  );
```

We can use this policy on top and class or method like this

```cs
  [HttpGet("gotwriteaccess")]
  [Authorize(Policy = "Write")]
  public async Task<ActionResult> Write()
  {
      return Ok("I got Write Access Bro!");
  }
```

## Further reading

You just learnt validating token and policy based access. You can explore about oauth2 and other approaches of securing API below.

- [ASP.Net Core Security](https://docs.microsoft.com/en-us/aspnet/core/security/?view=aspnetcore-3.1)
- [Oauth](https://www.oauth.com/oauth2-servers/accessing-data/)
- [Okta](https://developer.okta.com/docs/guides/validate-access-tokens/go/overview/)
