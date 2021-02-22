# 索引存在API | Index Exists API

> 通过此调用，查询索引是否存在

> [官方文档Index Exists API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-indices-exists.html)

## 索引存在请求

REST高级客户端，将`GetIndexRequest`用于索引存在API。索引名称是必需参数。

```java
GetIndexRequest request = new GetIndexRequest("twitter");
```

## 可选参数

索引存在API通过`GetIndexRequest`也接受以下可选参数：

```java
request.local(false);
request.humanReadable(true);
request.includeDefaults(false);
request.indicesOptions(indicesOptions);
```

- 是返回本地信息还是从主节点检索状态

- 以适配人类的格式返回结果（可读性友好）

- 是否返回每个索引的所有默认设置

- 控制不可用索引的处理方式以及通配符表达式的展开方式

### 扩展：IndicesOptions

构造函数为：

```java
public IndicesOptions(EnumSet<Option> options, EnumSet<WildcardStates> expandWildcards)
```

`IndicesOptions`预置几种默认配置，如下：

```java
public static final IndicesOptions STRICT_EXPAND_OPEN = new IndicesOptions(EnumSet.of(Option.ALLOW_NO_INDICES), EnumSet.of(WildcardStates.OPEN));
public static final IndicesOptions LENIENT_EXPAND_OPEN = new IndicesOptions(EnumSet.of(Option.ALLOW_NO_INDICES, Option.IGNORE_UNAVAILABLE), EnumSet.of(WildcardStates.OPEN));
public static final IndicesOptions LENIENT_EXPAND_OPEN_CLOSED = new IndicesOptions(EnumSet.of(Option.ALLOW_NO_INDICES, Option.IGNORE_UNAVAILABLE), EnumSet.of(WildcardStates.OPEN, WildcardStates.CLOSED));
public static final IndicesOptions STRICT_EXPAND_OPEN_CLOSED = new IndicesOptions(EnumSet.of(Option.ALLOW_NO_INDICES), EnumSet.of(WildcardStates.OPEN, WildcardStates.CLOSED));
public static final IndicesOptions STRICT_EXPAND_OPEN_FORBID_CLOSED = new IndicesOptions(EnumSet.of(Option.ALLOW_NO_INDICES, Option.FORBID_CLOSED_INDICES), EnumSet.of(WildcardStates.OPEN));
public static final IndicesOptions STRICT_EXPAND_OPEN_FORBID_CLOSED_IGNORE_THROTTLED = new IndicesOptions(EnumSet.of(Option.ALLOW_NO_INDICES, Option.FORBID_CLOSED_INDICES, Option.IGNORE_THROTTLED), EnumSet.of(WildcardStates.OPEN));
public static final IndicesOptions STRICT_SINGLE_INDEX_NO_EXPAND_FORBID_CLOSED = new IndicesOptions(EnumSet.of(Option.FORBID_ALIASES_TO_MULTIPLE_INDICES, Option.FORBID_CLOSED_INDICES), EnumSet.noneOf(WildcardStates.class));
```

- `Option`为不可用索引的处理方式，如下：

```java
public enum Option {
    IGNORE_UNAVAILABLE,
    IGNORE_ALIASES,
    ALLOW_NO_INDICES,
    FORBID_ALIASES_TO_MULTIPLE_INDICES,
    FORBID_CLOSED_INDICES,
    IGNORE_THROTTLED;

    public static final EnumSet<Option> NONE = EnumSet.noneOf(Option.class);
}
```

- `WildcardStates`为通配符表达式展开方式，如下：

```java
public enum WildcardStates {
    OPEN,
    CLOSED;

    public static final EnumSet<WildcardStates> NONE = EnumSet.noneOf(WildcardStates.class);

    public static EnumSet<WildcardStates> parseParameter(Object value, EnumSet<WildcardStates> defaultStates) {
        if (value == null) {
            return defaultStates;
        }

        Set<WildcardStates> states = new HashSet<>();
        String[] wildcards = nodeStringArrayValue(value);
        for (String wildcard : wildcards) {
            if ("open".equals(wildcard)) {
                states.add(OPEN);
            } else if ("closed".equals(wildcard)) {
                states.add(CLOSED);
            } else if ("none".equals(wildcard)) {
                states.clear();
            } else if ("all".equals(wildcard)) {
                states.add(OPEN);
                states.add(CLOSED);
            } else {
                throw new IllegalArgumentException("No valid expand wildcard value [" + wildcard + "]");
            }
        }

        return states.isEmpty() ? NONE : EnumSet.copyOf(states);
    }
}
```

## 同步执行

按以下方式执行`GetIndexRequest`，在继续执行代码前，客户端会一直等待`DeleteIndexResponse`响应：

```java
boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
```

在高级REST客户端中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行`GetIndexRequest`也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数`request`和`listener`给索引存在方法来指定如何处理响应及潜在失败：

```java
client.indices().existsAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的索引存在的监听器示例如下：

```java
ActionListener<Boolean> listener = new ActionListener<Boolean>() {
    @Override
    public void onResponse(Boolean exists) {
    }

    @Override
    public void onFailure(Exception e) {
    }
};
```

- `onResponse`：成功完成执行时被调用

- `onFailure`：`GetIndexRequest`执行失败时被调用

## 响应

响应结果为`boolean`值，表明索引是否存在
