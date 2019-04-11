# 对象数据类型

JSON文档本质上是分层的：文档包含内部对象，内部对象本身还包含内部对象。

```
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{ 	// 1
  "region": "US",
  "manager": { 	// 2
    "age":     30,
    "name": { 	// 3
      "first": "John",
      "last":  "Smith"
    }
  }
}'
```

- 1 外层的文档是JSON对象
- 2 包含称为`manager`的内部对象
- 3 `manager`对象还包含一个内部对象称为`name`

在内部，这个文档被索引为一个简单的、扁平的键值对列表，如下所示：

```
{
  "region":             "US",
  "manager.age":        30,
  "manager.name.first": "John",
  "manager.name.last":  "Smith"
}
```

上面文档的显式映射可以长这样：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": { 	// 1
      "properties": {
        "region": {
          "type": "keyword"
        },
        "manager": { 	// 2
          "properties": {
            "age":  { "type": "integer" },
            "name": { 	// 3
              "properties": {
                "first": { "type": "text" },
                "last":  { "type": "text" }
              }
            }
          }
        }
      }
    }
  }
}'
```

- 1 映射的类型是一个对象类型，具有一个`properties`字段
- 2 `manager`字段是一个内部`object`字段
- 3 `manager.name`字段是`manager`字段中的一个内部`object`字段

不需要显式地将字段类型设置为`object`类型，因为这是默认的类型。

## `object`字段的参数

|参数|说明|
|---|---|
|dynamic|新属性是否应动态添加到现有对象。接受true(默认),false和strict|
|enabled|是否应该对对象字段给出的JSON值进行解析和索引(true，默认)或完全忽略(false)|
|include_in_all|为对象中的所有属性设置默认的`include_in_all`值，对象本身没有添加到_all字段。|
|properties|对象内的字段，可以是任何数据类型，包括对象。可以将新属性添加到现有对象。|

> 如果你需要索引对象的数组而不是单个的对象，可以使用`nested`数据类型。
