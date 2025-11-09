---
layout: post
categories: tech
title: Flame (3)\: 使用Flame构建页面抓取工具
---

# Flame (3)\: 使用Flame构建页面抓取工具

随着智能体技术的不断发展，越来越多的企业和用户投入资源研发和部署智能体，以提升业务效率。
然而，受限于训练数据集的时效性，大语言模型往往难以及时获得最新的数据，难以全面满足智能体的实际需求。
为此，智能体常常结合RAG技术，用于存储并检索“新数据”。
这些“新数据”不仅包括企业内部的信息，如IT流程、合同等，还涵盖了互联网上的实时信息，例如股票价格、公司公告等。
对于互联网数据，需通过相应的爬取程序获取并存储到向量数据库中，便于智能体后续检索利用。

借助Flame v0.5版本全新的Python API，我们可以轻松构建一个高并发的网页爬取程序 (Crawler)，高效获取所需的互联网信息。同时，可通过Flame的命令行工具实时监控爬虫任务的执行进度和结果。

## Crawler服务端代码 

在Flame v0.5版本中，新增了`FlameInstance`接口。借助该接口及其Python注解（`FlameInstance.entrypoint`），我们可以明确指定应用的入口函数。当客户端发起请求后，Flame会根据集群当前的资源利用情况，自动将请求（任务）调度至合适的应用实例进行执行，实现高效的资源分配与任务处理。

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

通过 `flamepy.FlameInstance` 创建对象 `ins`，可以借助其实例定义应用的入口函数 (`def crawler(wp: WebPage) -> Answer`)。该入口函数在接收到网页请求后，会下载指定 URL 的内容，并将其转换为 Markdown 格式的文档。后续我们可以将转化后的 Markdown 内容灵活地存储到共享目录或向量数据库中，为智能体的后续检索提供便利。

## Crawler客户端代码

在Flame v0.05中，客户端的Python接口并未做更新。客户端通过`flamepy.create_session`创建与应用的会话，然后通过会话向相应的应用提交请求（任务）。
由于`flamepy`使用了 `asyncio`，客户端可以异步提交多个请求（任务），并通过 `asyncio.gather` 等待所有任务结束。

```python
crawler = await flamepy.create_session("crawler-app")

tasks = []
for url in urls:
    tasks.append(crawler.invoke(WebPage(url=url)))

await asyncio.gather(*tasks)

await crawler.close()
```

## 执行结果



### 启动Flame集群

为了方便演示，Flame代码中提供了`compose.yaml`配置，用户可以在本地启动一个Flame集群，如下所示：

```
k82cn$ docker compose up -d
[+] Running 4/4
 ✔ Network flame_default                     Created                                            0.0s 
 ✔ Container flame-flame-executor-manager-1  Started                                            0.3s 
 ✔ Container flame-flame-console-1           Started                                            0.4s 
 ✔ Container flame-flame-session-manager-1   Started                                            0.3s 
```

Flame集群启动后，可以登录`flame-console`容器以检测集群状态：

```
k82cn$ docker compose exec -it flame-console /bin/bash
root@2818097c3900:/# flmctl list -a
 Name     State    Tags  Created   Shim  Command                              
 flmexec  Enabled        13:37:48  Host  /usr/local/flame/bin/flmexec-service 
 flmping  Enabled        13:37:48  Host  /usr/local/flame/bin/flmping-service 
root@2818097c3900:/# 
```

### 部署Crawler应用

完成应用开发后，可以通过yaml配置部署应用到 Flame 集群中。
在配置中需要指定应用的名称，工作目录和相应的命令行。客户端通过应用的名字创建会话，而Flame通过命令行等配置在资源节点启动相应的应用。

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

创建应用配置后便可以通过`flmctl`将应用部署到Flame集群中。

```
root# flmctl register -f ./crawler-app.yaml
root# flmctl list -a 
 Name         State    Tags  Created   Shim  Command                              
 flmexec      Enabled        13:37:48  Host  /usr/local/flame/bin/flmexec-service 
 flmping      Enabled        13:37:48  Host  /usr/local/flame/bin/flmping-service 
 crawler-app  Enabled        13:51:53  Host  /usr/bin/uv   
```

### 运行

抓取应用部署后便可通过客户端发送请求（任务）。

```
root# uv run client.py api.py
......
```

所有抓取任务完成后，可以通过 `flmctl` 工具查看每个会话和任务的详细状态。在本示例中，Markdown 文档已保存至本地文件系统。

* 查询会话状态

```
root# flmctl list -s
 ID  State   App          Slots  Pending  Running  Succeed  Failed  Created  
 1   Closed  crawler-app  1      0        0        9        0       13:57:03 
```

* 查询任务状态

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

## 总结

Flame v0.5 提供的 Python API 极大地简化了开发流程，并能够与第三方库高效集成。依托 Flame 出色的调度与高并发能力，用户可以更加便捷地构建出能够应对大规模请求的应用程序。


## 引用

* Flame: http://github.com/xflops/flame
* Markitdown: https://github.com/microsoft/markitdown
