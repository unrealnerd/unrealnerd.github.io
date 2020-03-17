---
layout: post
title: Consuming SOAP service with dotnet core
description: Consuming SOAP service with dotnet core
author: bitsmonkey
tags: soap, dotnet core, soap service
---

![Image by <a href="https://pixabay.com/users/Merio-1480566/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=990304">Merio</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=990304">Pixabay</a>](/../img/soapservice-dotnetcore.jpg)

Here is how we consume a SOAP service in dotnet core.

Once you have a created a new web api project using Visual Studio. In Solution Explorer follow this,
Connected Services-->Add Connected Service then fill the the Url and and space which will generate the required classes just like the SvcUtil.exe does.

```csharp
var Uri = "http://service.com/service1.asmx";

BasicHttpBinding binding = new BasicHttpBinding();
EndpointAddress address = new EndpointAddress(Uri);

binding.Security.Transport.ClientCredentialType = HttpClientCredentialType.Ntlm;
binding.Security.Transport.ProxyCredentialType = HttpProxyCredentialType.None;

binding.Security.Mode = BasicHttpSecurityMode.TransportCredentialOnly;

soapClient soapClient = new soapClient(
    binding,
    address);

soapClient.Method1();
```

