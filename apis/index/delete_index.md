# 删除索引API | Delete Index API

> 通过这个调用，删除索引

> [官方文档Delete Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-delete-index.html)

## 删除索引请求

`DeleteIndexRequest`必需一个参数，用于指定索引名字

```java
DeleteIndexRequest request = new DeleteIndexRequest("posts");
```

## 可选参数

以下参数可选配：

- 所有节点(nodes)响应索引删除的超时时长，使用`TimeValue`配置
- 所有节点(nodes)响应索引删除的超时时长，使用`字符串`配置

```java
request.timeout(TimeValue.timeValueMinutes(2));
request.timeout("2m");
```

- 连接主节点的超时时长，使用`TimeValue`配置
- 连接主节点的超时时长，使用`字符串`配置

```java
request.masterNodeTimeout(TimeValue.timeValueMinutes(1));
request.masterNodeTimeout("1m");
```

- 设置`IndicesOptions`用于控制，如何解析不可用索引和如何展开通配符表达式

```java
request.indicesOptions(IndicesOptions.lenientExpandOpen());
```

## 同步执行

按以下方式执行`DeleteIndexRequest`，在继续执行代码前，客户端会一直等待`DeleteIndexResponse`响应：

```java
AcknowledgedResponse deleteIndexResponse = client.indices().delete(request, RequestOptions.DEFAULT);
```

在高级REST客户端（high-level REST client）中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行`DeleteIndexRequest`也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数`request`和`listener`给异步删除索引方法来指定如何处理响应及潜在失败：

```java
client.indices().deleteAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的删除索引的监听器示例如下：

```java
ActionListener<AcknowledgedResponse> listener =
        new ActionListener<AcknowledgedResponse>() {
    @Override
    public void onResponse(AcknowledgedResponse deleteIndexResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
};
```

> `onResponse`：成功完成执行时被调用

> `onFailure`：`CreateIndexRequest`执行失败时被调用

## 删除索引响应

返回的`DeleteIndexResponse`可以获取关于执行操作的信息，示例如下：

```java
boolean acknowledged = deleteIndexResponse.isAcknowledged();
```

> `isAcknowledged`显示是否所有的节点(node)已响应此请求

如果索引不存在，`ElasticsearchException`异常会被抛出：

```java
try {
    DeleteIndexRequest request = new DeleteIndexRequest("does_not_exist");
    client.indices().delete(request, RequestOptions.DEFAULT);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.NOT_FOUND) {
        
    }
}
```

> 状态`RestStatus.NOT_FOUND`，即索引不存在时，可在代码中进行相应的操作
