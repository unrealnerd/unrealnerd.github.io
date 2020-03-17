---
layout: post
title: Consuming SOAP service with dotnet core
description: Consuming SOAP service with dotnet core
author: bitsmonkey
tags: soap, dotnet core, soap service
---

![Image by Merio - from Pixabay](/../img/soapservice-dotnetcore.jpg)

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

-- *Image by Merio - from Pixabay*

