---
layout: post
title: Integrating Adobe Analytics service in Angular
description: Integrating Adode Analytics in Angular
author: bitsmonkey
tags: quicknotes, adobe-analytics, window, angular
---

![adobe-analytics](../img/adobe-analytics.jpg){:.coverimage}

This notes will hold good for any kind a integration with third-party tools who require our application to include theier scripts and call them to send tracking information. Tried my best keep it as much as angular way to do it.

Lets first create a type that will define the schema of the information that needs to be sent to the thirdparty thorugh the injected scripts. In our case below is the metrics model that will be used to track and analyze on our Adobe Dashboard.

```ts
\\metrics.model.ts
export interface Metrics {
    applicationame: string;
    country: string;
    language: string;    
    pagename: string;    
    userid: string;
    userrole: string;    
}
```

To call a third-party scripts we would need to include the script on the head section. Since we are trying angular way let add this script node dynamically on AppComponent load.

Implement `OnInit` on AppComponent as below

```ts
const scriprtNode = this.document.createElement('script');
scriprtNode.src = environment.adobeAnalyticsUrl;
scriprtNode.type = 'text/javascript';
scriprtNode.async = false;
this.document.getElementsByTagName('head')[0].appendChild(scriprtNode);
```

To use document inside the AppComponent you would have to inject it thorugh the constructor `@Inject(DOCUMENT) private document: Document`.

The Adobe Analytics expects the object to be bound to a predefined custom property of the [window](https://developer.mozilla.org/en-US/docs/Web/API/Window) object. We will create a service that will get reference to the window object and that can be injected in components. Its cleaner that way.

```ts
//window-ref.service.ts
import { Injectable } from '@angular/core';

@Injectable()
export class WindowRef {
   get nativeWindow() : any {
       return window;
   }
}
```

Lets create a create a service that will keep track of the object of type `Metrics`  that we created earlier. This service has to be used to set the object that we will be sending to Analytics service through their included script. 

```ts
//adobe-analytics.service.ts
import { Injectable } from '@angular/core';
import { WindowRef } from './window-ref.service';
import { Metrics } from 'src/models/metrics.model.ts';

@Injectable()
export class AdobeAnalyticsService {
    metrics: Metrics = {} as Metrics;

    constructor(private winRef: WindowRef) {
    }
    updateMetricsObject(deltaMetrics) {
        Object.assign(this.metrics, deltaMetrics)
        const wr = this.winRef.nativeWindow;
        wr.Org = wr.Org || {};
        wr.Org.Metrics = wr.Org.Metrics || {};
        wr.Org.Metrics.sc = this.metrics;
        return wr.Org.Metrics.sc;
    }

    sendToAdobe() {
        const wr = this.winRef.nativeWindow;
        if (wr.metrics_pagenav != undefined) wr.metrics_pagenav('', this.metrics);
    }
}
```

We will try to capture these metrics whenever we browse a new page so that we know which page is most visited and whats the path the user takes and various other insights that we can derive from other attributes like the user details and the page names. In Angular we can use the [`Router` events](https://angular.io/guide/router#router-events) to listen to the route changed event and call the analytics method to push the information.
 
```ts
//app-routing.module.ts
this.router.events.pipe(filter(e => e instanceof RoutesRecognized))
                .subscribe((event) => {
                    const url: string = event['urlAfterRedirects'];
                    const regex = /[a-zA-Z]+[-]*[a-zA-Z]+/gm;
                    var pageName = url.match(regex)[0];
                    this.adobeAnalyticsService.updateMetricsObject({ pagename: `blahblah|${pageName}` });
                    this.adobeAnalyticsService.sendToAdobe();
                    }
                );
```

You can now check your Adobe dashboard to the see tacking informations.