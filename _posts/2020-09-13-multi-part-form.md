---
layout: post
title: Quick Notes --> http multipart image form-data
description: Quick Notes --> http multipart image form-data
author: bitsmonkey
tags: upload image, multi-part form, http, quicknotes, gitlab
---

Here is how we upload an image using a rest api endpoint. For this example I will be using [gitlab uploads API](https://docs.gitlab.com/ee/api/projects.html#upload-a-file)

This how we post a PNG to an api using postman.

Add header `Content-Type: multipart/form-data` and then add the png image as following

![postman multi part file](/img/Postman_multi-part%20form.png)

Below is what happens on the wire to send. The content-type `multipart/form-data` basically allows us to send a form with multiple attributes of the form seperated with a [boundry](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST) `----WebKitFormBoundary7MA4YWxkTrZu0gW`.

```http
###
POST {{url}}/projects/{{projectId}}/uploads HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Authorization: Bearer {{GitlabAccessToken}}

----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="capture.png"
Content-Type: image/png

(image data)
----WebKitFormBoundary7MA4YWxkTrZu0gW--
```

