---
layout: post
title: Twilio Whatsapp on dotnetcore
description: using twilio api for Whatsapp messaging
author: bitsmonkey
tags: twilio, whatsapp, api,
---
<br/>
Getting access is to Whatsapp Business API's seems to be tricky, so lets try using twilio whats app sandbox. 

Here is how we get a 
Twilio API keys, Create an account here [Twilio Signup for free](https://www.twilio.com/try-twilio). Create a new project select SMS chatbot and complete that wizard. Now in the project console you would get the Account SID and Auth Token which we would be using for accessing all the twilio API's.

Create an console application using dotnet commands, and use the below code to send a message

`dotnet add package Twilio.AspNet.Core`

```csharp
        public void SendMessage(string body, string phoneNumber)
        {           
            TwilioClient.Init(_whatsAppOptions.Value.TwilioAccountSid, _whatsAppOptions.Value.TwilioAuthToken);

            try
            {
                var returnedMessage = MessageResource.Create(
                    body: body,
                    from: new Twilio.Types.PhoneNumber(_whatsAppOptions.Value.TwilioFromPhoneNumber),
                    to: new Twilio.Types.PhoneNumber(phoneNumber)
                );

            }
            catch (ApiException e)
            {
                throw e;
            }


        }
```

