---
layout: post
title: Streaming api
description: Streaming api using java springboot
author: bitsmonkey
tags: api, streaming, java
---

![streaming-java](/img/stream-java.jpg){:.coverimage}

Streaming API's are something we need when we have a large set of data to be sent to the client. The analogy that I can think of is this.If I wanted to water my garden I have two options fetch water in a bucket and then get to my garden and plants water one by one. This is similar a rest api where garden is the client app and I need to wait on the server(water source) to fill my bucket and then return that filled data set to the client. Streaming API lets me connect a pipe from the water source to the garden so that I get water in realtime and I need not wait until the complete data is retreived. It also save a lot server memory since there is no need to hold huge data volume in memory.

In this example I am trying to pull data from a database, since the volume of data is huge I will pull data in subsequent calls to the DB. Well there is trade off in this scenario since we have to make multiple calls to DB.

Lets create a streaming controller just like any rest api.

```java
//Controller.java

@Autowired
private TaskExecutor taskExecutor;

//input query: SELECT * FROM TABLE1 WHERE ID_COLUMN > $INDEX LIMIT $BACTHSIZE
@PostMapping("/stream/query")
public ResponseBodyEmitter executeMdx(@RequestBody Input input) {

    final ResponseBodyEmitter emitter = new ResponseBodyEmitter();

    taskExecutor.execute(() -> {
        try {

            final String batchSize = "10";

            for (int i = 0; i < 10; i++) {
                input.setInputQuery(input.getInputQuery().replace("$INDEX", String.valueOf(i + 1))
                        .replace("$BATCHSIZE", batchSize));

                Connector c = _applicationContext.getBean(Connector.class, input);

                String[][] result = c.executeQuery(input);
                if (result == null)
                    break;

                for (String[] x : result) {
                    emitter.send(String.join(",", x));
                }
            }
            emitter.complete();
        } catch (Exception e) {
            emitter.completeWithError(e);
            e.printStackTrace();
        }
        emitter.complete();

    });

        return emitter;
}
```

Our use case client applications asks for one row at a time with column comma seperated. This is very crude way. You can send binary of json as well using the emitter. Also note batchsize calculations can be manipulated dynamically depending on the requirement.

Taskexecutor bean is configured like this,

```java
//ThreadConfig.java
@Configuration
public class ThreadConfig {
    @Bean
    public TaskExecutor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(4);        
        executor.setThreadNamePrefix("task_executor_thread");
        executor.initialize();
        return executor;
    }
}
```

Make sure teh beans you use are thread ready I mean in our case Connector implements Runnable and so is accessible through the Task Executor.

```java
//Connector.java
@Service
@Scope("prototype")
public class Connector implements Runnable {

    @Autowired
    private Logger log;

    @Autowired
    private Grid grid;

    private Input _input;

    public Connector(Input input) {
        _input = input;
    }

    public String[][] executeQuery(Input input) {
        try {
            return grid.executeQuery(input);

        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }

        return null;
    }

    @Override
    public void run() {        
        executeQuery(_input);
    }
}
```

To pass values to the thread we are using constructor arguments.

This sample should be able to atleast guide you with the things required to acheive the streaming API. Might not be the right way for all the usecases.

Now that we have created the server side streaming API we will need to consume it in our UI this way,

```js
async function postData(url = '', data = {}) {
            
    var request = {
        method: 'POST',
        mode: 'cors',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
    }

    await fetch('http://localhost:8080/stream/query', request)
        .then(response => {
            const reader = response.body.getReader()
            const utf8Decoder = new TextDecoder("utf-8");
            const stream = new ReadableStream({
                start(controller) {
                    function push() {
                        reader.read().then(({
                            done,
                            value
                        }) => {
                            if (done) {
                                controller.close();
                                return;
                            }

                            controller.enqueue(value);
                            let listItem = document.createElement('li');
                            listItem.textContent = utf8Decoder.decode(value);
                            console.log(listItem.textContent);
                            document.body.appendChild(listItem);
                            push();
                        });
                    };

                    push();
                }
            });

            return new Response(stream, {
                headers: {
                    "Content-Type": "text/html"
                }
            });
        });
}

postData('http://localhost:8080/stream/query', {    
    "inputQuery": "SELECT {[AUGW120]:[AUGW420],[AUG20],[Q3FY20]} ON COLUMNS, NON EMPTY CrossJoin ({[Fulfillment Region].children}, CrossJoin ( SUBSET({Descendants ([Product_Org],3)},$INDEX,$BATCHSIZE), {[CSR_Unit_Sales],[UNIT_Ships]}))ON ROWS FROM [LIBASOD2.NEW_DM] WHERE ([Version].[&CVersion],[Site].[Site],[Customer].[Customer], [Geo].[Geo],[Channel].[Channel],[Config].[Config])"
}).then(data => {
    console.log(data);
});
```


Photo by [Joshua Sortino](https://unsplash.com/@sortino?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)