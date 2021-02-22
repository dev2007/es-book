# 获取索引API | Get Index API

> 通过此调用，获取索引信息

> [官方文档Index Exists API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-get-index.html)

## 获取索引请求

`GetIndexRequest` 要求一到多个索引参数:

```java
GetIndexRequest request = new GetIndexRequest("index");
```

## 可选参数

以下参数可选择提供：

```java
request.includeDefaults(true);
```

- 如果为`true`，未在索引中显示设置的值将返回默认值

```java
request.indicesOptions(IndicesOptions.lenientExpandOpen());
```

- 设置`IndicesOptions`，用于控制如何解析不可用的索引以及如何扩展通配符表达式。可参看其他页面中的介绍[扩展：indicesoptions](apis/index/index_exists?id=扩展：indicesoptions)

## 同步执行

按以下方式执行`GetIndexRequest`，在继续执行代码前，客户端会一直等待`GetIndexResponse`响应：

```java
GetIndexResponse getIndexResponse = client.indices().get(request, RequestOptions.DEFAULT);
```

在高级REST客户端中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行`GetIndexRequest`也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数`request`和`listener`给索引存在方法来指定如何处理响应及潜在失败：

```java
client.indices().getAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的获取索引的监听器示例如下：

```java
ActionListener<GetIndexResponse> listener =
    new ActionListener<GetIndexResponse>() {
        @Override
        public void onResponse(GetIndexResponse getIndexResponse) {
        }

        @Override
        public void onFailure(Exception e) {
        }
    };
```

- `onResponse`：成功完成执行时被调用

- `onFailure`：`GetIndexRequest`执行失败时被调用

## 获取索引响应

返回的`GetIndexResponse`可以获取关于执行操作的信息，示例如下：

```java
MappingMetadata indexMappings = getIndexResponse.getMappings().get("index");
Map<String, Object> indexTypeMappings = indexMappings.getSourceAsMap();
List<AliasMetadata> indexAliases = getIndexResponse.getAliases().get("index");
String numberOfShardsString = getIndexResponse.getSetting("index", "index.number_of_shards");
Settings indexSettings = getIndexResponse.getSettings().get("index");
Integer numberOfShards = indexSettings.getAsInt("index.number_of_shards", null);
TimeValue time = getIndexResponse.getDefaultSettings().get("index").getAsTime("index.refresh_interval", null);
```

- 获取一个不同类型的映射到索引的`MappingMetadata`对象中

- 获取文档类型`doc`的属性映射

- 获取索引别名列表

- 获取索引的分片字符串数值（参数`index.number_of_shards`的值）。如果这个参数未指定，`includeDefault`又为`true`，将返回默认值

- 返回索引所有设置

- 设置对象提供更多灵活性。此处将参数`index.number_of_shards`数值转为了整数

- 获取索引的刷新周期值（参数`index.refresh_interval`的值）。如果`includeDefault`为`true`，返回默认值；如果为`false`，返回空
