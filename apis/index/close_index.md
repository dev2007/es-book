# 关闭索引API | Close Index API

> 通过此调用，关闭开启的索引

> [官方文档 Close Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-close-index.html)

## 关闭索引请求

一个 `CloseIndexRequest` 要求一个索引参数:

```java
CloseIndexRequest request = new CloseIndexRequest("index");
```

## 可选参数

以下参数可选择提供：

```java
request.setTimeout(TimeValue.timeValueMinutes(2));
```

- 超时值，用于等待所有节点确认索引开启，`TimeValue` 类型

```java
request.setMasterTimeout(TimeValue.timeValueMinutes(1));
```

- 连接主节点超时值，`TimeValue` 类型

```java
request.indicesOptions(IndicesOptions.lenientExpandOpen());
```

- 设置 `IndicesOptions`，用于控制如何解析不可用的索引以及如何扩展通配符表达式。可参看其他页面中的介绍[扩展：indicesoptions](apis/index/index_exists?id=扩展：indicesoptions)

## 同步执行

按以下方式执行 `CloseIndexRequest`，在继续执行代码前，客户端会一直等待 `CloseIndexResponse` 响应：

```java
AcknowledgedResponse closeIndexResponse = client.indices().close(request, RequestOptions.DEFAULT);
```

在高级REST客户端中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行 `CloseIndexRequest` 也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数 `request` 和 `listener` 给索引存在方法来指定如何处理响应及潜在失败：

```java
client.indices().closeAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的关闭索引的监听器示例如下：

```java
ActionListener<CloseIndexResponse> listener = new ActionListener<CloseIndexResponse>() {
    @Override
    public void onResponse(CloseIndexResponse closeIndexResponse) {
    }

    @Override
    public void onFailure(Exception e) {
    }
};
```

- `onResponse`：成功完成执行时被调用

- `onFailure`：`GetIndexRequest`执行失败时被调用

## 关闭索引响应

返回的 `CloseIndexResponse` 可以获取关于执行操作的信息，示例如下：

```java
boolean acknowledged = closeIndexResponse.isAcknowledged();
```

- `isAcknowledged`显示是否所有的节点(node)已响应此请求
