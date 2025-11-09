---
layout: post
categories: tech
title: Flame (3)&#58; Building a Web Crawler with Flame
---

As agent technology continues to evolve, enterprises and users are increasingly investing in developing and deploying agents to improve business efficiency.
However, constrained by the timeliness of training datasets, large language models often struggle to obtain the latest data in a timely manner, making it difficult to fully meet the actual needs of agents.
To address this, agents often leverage RAG (Retrieval-Augmented Generation) technology to store and retrieve "new data."
This "new data" includes not only internal enterprise information, such as IT processes and contracts, but also real-time information from the internet, such as stock prices and company announcements.
For internet data, web crawlers are needed to fetch and store it in vector databases, facilitating subsequent retrieval and utilization by agents.

With Flame v0.5's new Python API, we can easily build a high-concurrency web crawler to efficiently obtain the required internet information. Additionally, we can use Flame's command-line tools to monitor the execution progress and results of crawling tasks in real-time.

## Crawler Server Code

In Flame v0.5, a new `FlameInstance` interface has been added. Using this interface and its Python decorator (`FlameInstance.entrypoint`), we can explicitly specify the application's entry function. When a client initiates a request, Flame automatically schedules the request (task) to an appropriate application instance based on the cluster's current resource utilization, achieving efficient resource allocation and task processing.

```python
ins = flamepy.FlameInstance()

@ins.entrypoint
def crawler(wp: WebPage) -> Answer:
    text = requests.get(wp.url, headers=headers).text

    md = markitdown.MarkItDown()
    stream = io.BytesIO(text.encode("utf-8"))
    result = md.convert(stream).text_content

    ...
    
    return Answer(answer=f"Crawled {wp.url}")
```

By creating an instance `ins` with `flamepy.FlameInstance`, we can use it to define the application's entry function (`def crawler(wp: WebPage) -> Answer`). This entry function, upon receiving a web page request, downloads the content of the specified URL and converts it into a Markdown-formatted document. The converted Markdown content can then be stored in a shared directory or vector database, facilitating subsequent retrieval by agents.

## Crawler Client Code

In Flame v0.5, the client's Python interface remains unchanged. The client creates a session with the application using `flamepy.create_session`, then submits requests (tasks) to the application through the session.
Since `flamepy` uses `asyncio`, the client can asynchronously submit multiple requests (tasks) and wait for all tasks to complete using `asyncio.gather`.

```python
crawler = await flamepy.create_session("crawler-app")

tasks = []
for url in urls:
    tasks.append(crawler.invoke(WebPage(url=url)))

await asyncio.gather(*tasks)

await crawler.close()
```

## Execution Results

### Starting the Flame Cluster

For demonstration purposes, Flame's codebase includes a `compose.yaml` configuration file that allows users to start a Flame cluster locally, as shown below:

```
k82cn$ docker compose up -d
[+] Running 4/4
 ✔ Network flame_default                     Created                                            0.0s 
 ✔ Container flame-flame-executor-manager-1  Started                                            0.3s 
 ✔ Container flame-flame-console-1           Started                                            0.4s 
 ✔ Container flame-flame-session-manager-1   Started                                            0.3s 
```

Once the Flame cluster is running, you can log into the `flame-console` container to check the cluster status:

```
k82cn$ docker compose exec -it flame-console /bin/bash
root@2818097c3900:/# flmctl list -a
 Name     State    Tags  Created   Shim  Command                              
 flmexec  Enabled        13:37:48  Host  /usr/local/flame/bin/flmexec-service 
 flmping  Enabled        13:37:48  Host  /usr/local/flame/bin/flmping-service 
root@2818097c3900:/# 
```

### Deploying the Crawler Application

Once the application is developed, you can deploy it to the Flame cluster using a YAML configuration file.
The configuration must specify the application name, working directory, and command line. Clients create sessions using the application name, while Flame starts the application on resource nodes using the command line and other configuration parameters.

```
root# cat crawler-app.yaml 
metadata:
  name: crawler-app
spec:
  working_directory: /opt/examples/crawler/
  environments:
    FLAME_LOG_LEVEL: DEBUG
  command: /usr/bin/uv
  arguments:
    - run
    - crawler.py
    - api.py
```

Once the application configuration is created, you can deploy the application to the Flame cluster using `flmctl`.

```
root# flmctl register -f ./crawler-app.yaml
root# flmctl list -a 
 Name         State    Tags  Created   Shim  Command                              
 flmexec      Enabled        13:37:48  Host  /usr/local/flame/bin/flmexec-service 
 flmping      Enabled        13:37:48  Host  /usr/local/flame/bin/flmping-service 
 crawler-app  Enabled        13:51:53  Host  /usr/bin/uv   
```

### Running

Once the crawler application is deployed, clients can send requests (tasks).

```
root# uv run client.py api.py
......
```

Once all crawling tasks are completed, you can use the `flmctl` tool to view the detailed status of each session and task. In this example, the Markdown documents have been saved to the local file system.

* Query session status

```
root# flmctl list -s
 ID  State   App          Slots  Pending  Running  Succeed  Failed  Created  
 1   Closed  crawler-app  1      0        0        9        0       13:57:03 
```

* Query task status

```
root# flmctl view -s 1 -t 1
Task:          1
Session:       1
Application:   crawler-app
State:         Succeed
Events:        
  13:57:08.549: Running task on host <78f3468f8bc8>. (1)
  13:57:09.187: Task completed successfully on host <78f3468f8bc8>. (2)
```

```
root# flmctl view -s 1 -t 2
Task:          2
Session:       1
Application:   crawler-app
State:         Succeed
Events:        
  13:57:08.676: Running task on host <78f3468f8bc8>. (1)
  13:57:09.735: Task completed successfully on host <78f3468f8bc8>. (2)
```

## Summary

Flame v0.5's Python API greatly simplifies the development process and enables efficient integration with third-party libraries. By leveraging Flame's excellent scheduling and high-concurrency capabilities, users can easily build applications capable of handling large-scale requests.

## References

* Flame: [http://github.com/xflops/flame](http://github.com/xflops/flame)
* Markitdown: [https://github.com/microsoft/markitdown](https://github.com/microsoft/markitdown)

