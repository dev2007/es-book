# 分析API | Analyze API

> 通过此调用，对文本进行分析

> [官方文档 Analyze API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-analyze.html)

## 分析请求

一个 `AnalyzeRequest` 包含待分析的文本，以及指定如何进行分析的几个选项之一。

使用内置分词器的简单版本：

```java
AnalyzeRequest request = AnalyzeRequest.withGlobalAnalyzer("english","Some text to analyze", "Some more text to analyze");
```

- 第一个参数指定内置分词器；其他参数的多个字符串被视为多值字段。

你可以配置自定义的分词器：

```java
Map<String, Object> stopFilter = new HashMap<>();
stopFilter.put("type", "stop");
stopFilter.put("stopwords", new String[]{ "to" });  
AnalyzeRequest request = AnalyzeRequest.buildCustomAnalyzer("standard")
    .addCharFilter("html_strip")
    .addTokenFilter("lowercase")
    .addTokenFilter(stopFilter)
    .build("<b>Some text to analyze</b>");
```

- `stopFilter.put("stopwords", new String[]{ "to" })` 配置一个标记过滤器(`tokenfilter`)
- `AnalyzeRequest.buildCustomAnalyzer("standard")` 配置标记器(`tokenizer`)
- `addCharFilter("html_strip")` 配置字符过滤器
- `addTokenFilter("lowercase")` 添加内置标记过滤器
- `addTokenFilter(stopFilter)` 添加自定义标记过滤器

你也可以通过仅包括字符过滤器（`charfilter`）和标记过滤器（`tokenfilter`） 创建一个自定义的标准化器（`normalizer`）：

```java
AnalyzeRequest request = AnalyzeRequest.buildCustomNormalizer()
    .addTokenFilter("lowercase")
    .build("<b>BaR</b>");
```

你可以使用在现有索引中定义的分析器来分析文本：

```java
AnalyzeRequest request = AnalyzeRequest.withIndexAnalyzer(
    "my_index",
    "my_analyzer",
    "some text to analyze"
);
```

或者你也可以使用一个标准化器：

```java
AnalyzeRequest request = AnalyzeRequest.withNormalizer(
    "my_index",
    "my_normalizer",
    "some text to analyze"
);
```

你可以使用索引中特定字段的映射来分析文本：

```java
AnalyzeRequest request = AnalyzeRequest.withField("my_index", "my_field", "some text to analyze");
```

## 可选参数

以下参数可选择提供：

```java
request.explain(true);
request.attributes("keyword", "type");
```

- 设置 `explain` 为 `true`，将为响应增加更多的细节
- 设置 `attributes`，允许只返回你感兴趣的标记（`token`）属性

## 同步执行

按以下方式执行 `AnalyzeRequest`，在继续执行代码前，客户端会一直等待 `AnalyzeResponse` 响应：

```java
AnalyzeResponse response = client.indices().analyze(request, RequestOptions.DEFAULT);
```

在高级REST客户端中的同步调用，由于处理REST响应失败，可能抛出异常`IOException`。比如请求超时，或者类似原因的服务器无响应。

当服务器返回`4xx`或者`5xx`的的错误码时，高级客户端（high-level client）会试图处理响应体的错误详情，并将其替换，抛出一个通用的异常`ElasticsearchException`，同时将原生的异常`ResponseException`作为它的抑制异常（suppressed exception）。

## 异步执行

执行 `AnalyzeReques` 也可以异步的形式进行，这样客户端可以直接返回。用户需要通过传递参数 `request` 和 `listener` 给索引存在方法来指定如何处理响应及潜在失败：

```java
client.indices().analyzeAsync(request, RequestOptions.DEFAULT, listener);
```

异步方法不会阻塞，会直接返回。一旦它执行完成时，如果成功，`ActionListener`中的方法`onResponse`将会被回调；如果失败，`onFailure`将会被回调。失败情形和期待的异常与同步执行时一样。

一种典型的分析的监听器示例如下：

```java
ActionListener<AnalyzeResponse> listener = new ActionListener<AnalyzeResponse>() {
    @Override
    public void onResponse(AnalyzeResponse analyzeTokens) {
    }

    @Override
    public void onFailure(Exception e) {
    }
};
```

- `onResponse`：成功完成执行时被调用
- `onFailure`：全部 `AnalyzeRequest` 执行失败时被调用

## 分析响应

返回的 `AnalyzeResponse` 可以获取关于执行操作的信息，示例如下：

```java
List<AnalyzeResponse.AnalyzeToken> tokens = response.getTokens();
```

- `AnalyzeToken` 保存有关分析生成的单个标记的信息

如果 `explain` 设置为 `true`，则改为从 `detail()` 方法返回信息：

```java
DetailAnalyzeResponse detail = response.detail();
```

- `DetailAnalyzeResponse` 保存有关分析链中各个子步骤生成的标记的更详细信息
