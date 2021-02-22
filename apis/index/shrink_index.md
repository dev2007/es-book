# 收缩索引API | Shrink Index API

> 通过此调用，对索引进行收缩

> [官方文档 Shrink Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-shrink-index.html)

## 缩容请求

收缩API要求一个 `ResizeRequest` 实体。 一个 `ResizeRequest` 要求两个字符串参数：

```java
ResizeRequest request = new ResizeRequest("target_index","source_index");
```

- 将源索引（第二个参数）收缩到目标索引（第一个参数）

## 可选参数

以下参数可选择提供：

```java
request.timeout(TimeValue.timeValueMinutes(2));
request.timeout("2m");
```

- 等待所有节点确认打开的索引超时时长，使用`TimeValue`配置
- 等待所有节点确认打开的索引超时时长，使用`字符串`配置

```java
request.masterNodeTimeout(TimeValue.timeValueMinutes(1));
request.masterNodeTimeout("1m");
```

- 连接主节点的超时时长，使用`TimeValue`配置
- 连接主节点的超时时长，使用`字符串`配置

```java
request.setWaitForActiveShards(2);
request.setWaitForActiveShards(ActiveShardCount.DEFAULT);
```

- 收缩索引API返回响应之前要等待的活动分片数，以 `int` 表示
- 收缩索引API返回响应之前要等待的活动分片数，以 `ActiveShardCount` 表示

```java
request.getTargetIndexRequest().settings(Settings.builder()
        .put("index.number_of_shards", 2)
        .putNull("index.routing.allocation.require._name"));
```

- `index.number_of_shards` 配置收缩索引请求的目标分片数值
- `index.routing.allocation.require._name` 删除从源索引复制的分配要求

```java
request.getTargetIndexRequest().alias(new Alias("target_alias"));
```

- 关联目标索引的别名

## 同步执行

按以下方式执行 `ResizeRequest`，在继续执行代码前，客户端会一直等待 `ResizeResponse` 响应：

```java
ResizeResponse resizeResponse = client.indices().shrink(request, RequestOptions.DEFAULT);
```

在高级REST客户端中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行 `ResizeRequest` 也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数 `request` 和 `listener` 给索引存在方法来指定如何处理响应及潜在失败：

```java
client.indices().shrinkAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的分析的监听器示例如下：

```java
ActionListener<ResizeResponse> listener = new ActionListener<ResizeResponse>() {
    @Override
    public void onResponse(ResizeResponse resizeResponse) {
    }

    @Override
    public void onFailure(Exception e) {
    }
};
```

- `onResponse`：成功完成执行时被调用

- `onFailure`：全部 `ResizeRequest` 执行失败时被调用

## 收缩索引响应

返回的 `ResizeResponse` 可以获取关于执行操作的信息，示例如下：

```java
boolean acknowledged = resizeResponse.isAcknowledged();
boolean shardsAcked = resizeResponse.isShardsAcknowledged();
```

- `isAcknowledged`显示是否所有的节点(node)已响应此请求

- `isShardsAcknowledged`显示在超时前，是否为索引中的每个分片（shard）开启了必需数量的分片复本(shard copies)