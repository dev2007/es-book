# 创建索引API | Create Index API

> 通过这个调用，可以创建一个索引（index）

> [官方文档Create Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-create-index.html)

## 创建请求对象

`CreateIndexRequest`必需一个参数，用于指定索引名字

```java
CreateIndexRequest request = new CreateIndexRequest("twitter");
```

## 索引配置（settings）

每个索引创建时可以指定相关的索引配置（settings）

```java
request.settings(Settings.builder() 
    .put("index.number_of_shards", 3)
    .put("index.number_of_replicas", 2)
);
```

## 索引映射（mappings）

索引可以创建时通过映射（mappings）配置文档（document）类型

```java
request.mapping(
        "{\n" +
        "  \"properties\": {\n" +
        "    \"message\": {\n" +
        "      \"type\": \"text\"\n" +
        "    }\n" +
        "  }\n" +
        "}",
        XContentType.JSON);
```

映射（mappings）源可以用不同的方式添加，以上示例为JSON字符串设置的映射属性。

```java
Map<String, Object> message = new HashMap<>();
message.put("type", "text");
Map<String, Object> properties = new HashMap<>();
properties.put("message", message);
Map<String, Object> mapping = new HashMap<>();
mapping.put("properties", properties);
request.mapping(mapping);
```

以上使用`Map`配置映射源，会自动转为JSON格式。

```java
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.startObject("properties");
    {
        builder.startObject("message");
        {
            builder.field("type", "text");
        }
        builder.endObject();
    }
    builder.endObject();
}
builder.endObject();
request.mapping(builder);
```

以上通过Elasticsearch内置的帮助器，以`XContentBuilder`对象的方式为映射源生成JSON配置

## 索引别名 | Index aliases

别名（aliases）可以在索引（index）创建时设置

```java
request.alias(new Alias("twitter_alias").filter(QueryBuilders.termQuery("user", "kimchy")));  
```

## 全部配置源一次性配置

全部配置源，包括映射（mappings）、设置(settings)和别名(aliases)，可以一次性配置：

```java
request.source("{\n" +
        "    \"settings\" : {\n" +
        "        \"number_of_shards\" : 1,\n" +
        "        \"number_of_replicas\" : 0\n" +
        "    },\n" +
        "    \"mappings\" : {\n" +
        "        \"properties\" : {\n" +
        "            \"message\" : { \"type\" : \"text\" }\n" +
        "        }\n" +
        "    },\n" +
        "    \"aliases\" : {\n" +
        "        \"twitter_alias\" : {}\n" +
        "    }\n" +
        "}", XContentType.JSON); 
```

以上是按JSON字符串的形式进行配置。也可以用`Map` 或`XContentBuilder`来配置。

## 可选参数

以下参数可选进行配置：

1. 所有节点(nodes)响应索引创建的超时时长，使用`TimeValue`配置

```java
request.setTimeout(TimeValue.timeValueMinutes(2)); 
```

2. 连接主节点的超时时长，使用`TimeValue`配置

```java
request.setMasterTimeout(TimeValue.timeValueMinutes(1));
```

3. 在创建索引API响应前，激活的分片复本(shard copies)数,使用`int`或`ActiveShardCount`配置

```java
request.waitForActiveShards(ActiveShardCount.from(2)); 
request.waitForActiveShards(ActiveShardCount.DEFAULT); 
```

## 同步执行

按以下方式执行`CreateIndexRequest`，在继续执行代码前，客户端会一直等待`CreateIndexResponse`响应：

```java
CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
```

在高级REST客户端（high-level REST client）中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行`CreateIndexRequest`也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数`request`和`listener`给异步方法来指定如何处理响应及潜在失败：

```java
client.indices().createAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的创建索引（index）的监听器示例如下：

```java
ActionListener<CreateIndexResponse> listener =
        new ActionListener<CreateIndexResponse>() {

    @Override
    public void onResponse(CreateIndexResponse createIndexResponse) {
    }

    @Override
    public void onFailure(Exception e) {
    }
};
```

- `onResponse`：成功完成执行时被调用

- `onFailure`：`CreateIndexRequest`执行失败时被调用

## 创建索引响应

返回的`CreateIndexResponse`可以获取关于执行操作的信息，示例如下：

```java
boolean acknowledged = createIndexResponse.isAcknowledged();
boolean shardsAcknowledged = createIndexResponse.isShardsAcknowledged();
```

- `isAcknowledged`显示是否所有的节点(node)已响应此请求

- `isShardsAcknowledged`显示在超时前，是否为索引中的每个分片（shard）开启了必需数量的分片复本(shard copies)

## 扩展

- 索引的相关介绍，见：[索引相关概念](/notes/index_contents?id=索引相关概念)