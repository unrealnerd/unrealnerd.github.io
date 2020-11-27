---
layout: post
title: Sending email using flowable workflow engine
description: Sending email using flowable workflow engine
author: bitsmonkey
tags: java, spring, flowable, workflow
---

![flowable-email-task-cover](/img/flowable-email-task-cover.jpg){:.coverimage}

Integrating Email task in Flowable. If doing from flowable-UI follow this.

![flowable-email-task](/img/flowable-email-task.png){:.coverimage}

Once you are done with the filing To, From, Html/HtmlVar, etc,. Configure the email server configuration in your spring application.

```properties
# email config
flowable.mail.server.host=smtp.google.com
flowable.mail.server.port=587
flowable.mail.server.username=arjun
flowable.mail.server.password=arjun
flowable.mail.server.use-tls=true
```

In case you want to add the email task manually paste this node to your workflow bpmn file.

```xml
<serviceTask id="RejectionEmail" name="Rejection Email" flowable:type="mail">
      <extensionElements>
        <flowable:field name="to">
          <flowable:string><![CDATA[arjun@bitmonkey.in]]></flowable:string>
        </flowable:field>
        <flowable:field name="subject">
          <flowable:string><![CDATA[Rejection]]></flowable:string>
        </flowable:field>
        <flowable:field name="html">
          <flowable:string><![CDATA[<a href="http://bitsmonkey.in">Rejection reason</a>
<input type="textarea">Dummy Message</input>]]></flowable:string>
        </flowable:field>
      </extensionElements>
    </serviceTask>
```

Code On!