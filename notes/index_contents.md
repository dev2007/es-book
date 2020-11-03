# 索引相关概念

> 介绍涉及索引属性相关的概念

我们使用如下接口查询索引的相关信息：

```java
GET http://localhost:9200/myindex
```

查询到的映射配置示例如下：

```json
{
    "myindex": {
        "aliases": {},
        "mappings": {
            "properties": {
                "age": {
                    "type": "long"
                },
                "description": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                },
                "name": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                },
                "sex": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                }
            }
        },
        "settings": {
            "index": {
                "creation_date": "1604283614484",
                "number_of_shards": "1",
                "number_of_replicas": "1",
                "uuid": "VoSHcCz6SRuBb9V_oPFI8Q",
                "version": {
                    "created": "7090299"
                },
                "provided_name": "myindex"
            }
        }
    }
}
```

## 映射部分

> 索引映射json示例

```json
"mappings": {
    "properties": {
        "age": {
            "type": "long"
        },
        "description": {
            "type": "text",
            "fields": {
                "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                }
            }
        },
        "name": {
            "type": "text",
            "fields": {
                "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                }
            }
        },
        "sex": {
            "type": "text",
            "fields": {
                "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                }
            }
        }
    }
}
```

可以看到结构为：`mappings` -> `properties` -> `字段` -> `相关属性`

JSON中的`description`字段相关属性示例如下：

```json
"description": {
    "type": "text",
    "fields": {
        "keyword": {
            "type": "keyword",
            "ignore_above": 256
        }
    }
}
```

- type : 字段的数据类型。表明这个字段的数据是什么类型。

> [官方mapping type说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

相关类型见下表：

| 类型所属分类 | 类型 | 说明 |
| ---- | ---- | ---- |
| 普通类型（Common types）| | |
|  | binary | 以Base64编码为字符串的二进制数据 |
|  | boolean | `true`或`false`|
|  | Keywords |  `keywords`一族，包括`keyword`、`constant_keyword`和`wildcard` |
|  | Numbers | 数字类型，如`long`和`double`，用于表示数额 |
|  | Dates | 日期类型，包括`date`和`date_nanos`|
|  | alias | 为现有字段定义一个别名|
| 对象及相关类型（Objects and relational types） | | |
|  | object | JSON对象|
|  | flattened | 完整的JSON对象作为一个字段值 |
|  | nested | 保存了和子字段关系的JSON对象 |
|  | join | 相同索引下的文档定义了父子关系 |
| 体系数据类型（Structured data types）| | |
|  | Range | 范围类型，如`long_rang`、`double_range`、`date_range`和`ip_range` |
|  | ip | IPv4和IPv6地址 |
|  | murmur3 | 计算和存储值的hash值 |
| 合计数据类型（Aggregate data types）| | |
|  | histogram | 预合计的数字值 |
| 文本搜索类型（Text search types）| | |
|  | text | 分析、非体系的文本|
|  | annotated-text | 文本包含特定标识，用于标识命名的实体 |
|  | completion | 用于自动完成建议|
|  | search_as_you_type | 作为`as-you-type`完成的类文本类型 |
|  | token_count | 文本中的标记（token）计数|
| 文档排序类型（Document ranking types）| | |
|  | dense_vector | 记录浮点值的密集向量（dense vectors）|
|  | sparse_vector | 记录浮点值的稀疏向量（sparse vectors）|
|  | rank_feature | 记录一个数字特征以提高查询时的点击率 |
|  | rank_features | 记录多数字特征以提高查询时的点击率 |
| 空间数据类型(Spatial data types）| | |
|  | geo_point | 经纬度点 |
|  | geo_shape | 复杂形状，如多边形|
|  | point | 任意笛卡尔点 |
|  | shape | 任意笛卡尔几何 |
| 其他类型(Other types）| | |
|  | percolator | 写在`Query DSL`中的索引查询|

- fields：将字段映射为其他用途字段。

JSON中的`description`字段相关属性示例如下：

```json
"description": {
    "type": "text",
    "fields": {
        "keyword": {
            "type": "keyword",
            "ignore_above": 256
        }
    }
}
```

`description`字段，类型为`text`，然后映射出一个字段`keyword`，这个字段的类型为`keyword`。通过字段的字段，将数据拆分为多种类型进行搜索。

> 其他一些参数暂不涉及，不作介绍。

## 设置部分

> 索引设置json部分

```json
"index": {
    "creation_date": "1604283614484",
    "number_of_shards": "1",
    "number_of_replicas": "1",
    "uuid": "VoSHcCz6SRuBb9V_oPFI8Q",
    "version": {
        "created": "7090299"
    },
    "provided_name": "myindex"
}
```

其中重要的参数：

- `number_of_shards`：分片数，默认为1

- `number_of_replicas`：备份数，默认为1

> [官方setting参数说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-modules-settings)

索引(index)相关设置参数见下表：

| 分类 | 参数 | 说明 |
| ---- | ----  | ----  |
| 静态索引设置（Static index settings） |  |  |
|  | **number_of_shards** | 索引的主分片数，默认为`1`，上限为`1024`。只能索引创建时设置。 |
|  | number_of_routing_shards | 用于拆分 （`split`）索引路由分片数 |
|  | shard.check_on_startup | 打开前对分片进行检测，损坏会中断打开，默认值为`false`，值有： `true`、`checksum`、`false`|
|  | codec | 默认使用LZ4压缩存储数据，可使用`best_compression` |
|  | routing_partition_size | 自定义可转换的路由分片数。 默认值`1`，必须小于`number_of_shards`（除非它也是1），只能索引创建时设置。|
|  | ~~soft_deletes.enabled~~ | 索引软删除，默认`true`。支持修改版本：[`6.5.0`,`7.6.0`) ，小版本无此配置，大版本不可修改|
|  | soft_deletes.retention_lease.period | 过期前的保留周期，默认`12h` |
|  | load_fixed_bitset_filters_eagerly | 是否为嵌套查询预加载缓存筛选器，默认`true`，值有：`true`、`false` |
| 动态索引设置（Dynamic index setting） |  |  |
|  | hidden | 索引是否默认隐藏，如隐藏，需要在请求带参`expand_wildcards`才可查询。默认值为`false`，值有：`true`、`false` |
|  | **number_of_replicas** | 每个主分片的备份数，默认值为`1` |
|  | auto_expand_replicas | 基于集群中数据节点数量，自动扩展备份数，默认为`false` |
|  | search.idle.after | 搜索空闲前等待时间，默认值`30s` |
|  | refresh_interval | 刷新操作周期，让索引最近变更可被搜索。默认值`1s`，可设置为`-1`表示禁用刷新 |
|  | max_result_window | 从索引搜索数据的最大值（即`from` + `size`的值），默认值`10000` |
|  | max_inner_result_window | 索引内部命中和最高命中聚集最大值（即`from` + `size`的值），默认值`100` |
|  | max_rescore_window | 搜索索引重打分（`rescore`）请求时的`window_size`最大值（默认与`max_result_window`一样默认`10000`） |
|  | max_docvalue_fields_search | 允许查询`docvalue_fields`最大值，默认值`100` |
|  | max_script_fields | 允许查询`script_fields`最大值，默认值`32` |
|  | max_ngram_diff | 用于`NGramTokenizer`和`NGramTokenFilter`的，`min_gram`和`max_gram`之间的最大差值 ，默认值`1` |
|  | max_shingle_diff | 用于`shingle token filter`的，`max_shingle_size`和`min_shingle_size`之间的最大差值，默认值`3` |
|  | max_refresh_listeners | 每个索引分片上的最大刷新监听器数量。 |
|  | analyze.max_token_count | 用于_analyze API的最大标记值，默认值`10000` |
|  | highlight.max_analyzed_offset | 用于高亮请求的可分析最大字符数量，仅对无`offsets`和`term vectors`的文本高亮请求有效。默认值`1000000` |
|  | max_terms_count | 用于词语查询最大词语数量，默认值`65536` |
|  | max_regex_length | 用于正则查询的最大正则长度，默认值`1000` |
|  | routing.allocation.enable | 控制索引的分片分配。默认值`all`，值有：`all`、`primaries`、`new_primaries`、`none` |
|  | routing.rebalance.enable | 允许索引分片重平衡，默认值`all`，值有：`all`、`primaries`、`replicas`、`none` |
|  | gc_deletes | 已删除文档的版本号可用于进一步版本化操作的时长，默认值`60s` |
|  | default_pipeline | 索引的默认摄取节点（ingest node）管道。可使用参数`pipeline`重载。特定参数`_none`表明不会运行摄取节点管道。 |
|  | final_pipeline | 索引的最终摄取节点（ingest node）管道。特定参数`_none`表明不会运行摄取节点管道。 |

> 其他一些参数暂不涉及，不作介绍。

## 别名部分

> 之上示例中别名为空，以下举官方示例：

```json
{
  "aliases": {
    "alias_1": {},
    "alias_2": {
      "filter": {
        "term": { "user.id": "kimchy" }
      },
      "routing": "shard-1"
    }
  }
}
```

`aliases`下的为当前索引的别名，示例中的别名为：`alias_1`和`alias_2`。

别名`alias_2`中还有`filter`和`routing`。`filter`代表用此别名查询时，数据会按此过滤器过滤；`routing`则代表用此别名时，路由到的分片，避免分片操作。

> 其他一些参数暂不涉及，不作介绍。
