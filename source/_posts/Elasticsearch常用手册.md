---
title: Elasticsearch常用手册
date: 2019-03-20 22:26:06
urlname: elasticsearch
categories: ["技术"]
tags: ["Elasticsearch"]
toc: true
---

目前大部分的前后端协作方式仍然是后端根据业务需求给前端提供接口返回特定数据，但是Elasticsearch却为我们提供了可能来自己根据需要搜索筛选以及更新数据。比如查询商品列表，我们可以自己根据品牌、入库时间、颜色等条件筛选获取数据。

### Elasticsearch是什么

Elasticsearch是一个基于Lucene的搜索服务器，简称ES。它提供了一个分布式多用户能力的全文搜索引擎，每一个字段都可以被搜索到，且可以结构化检索，简单地使用JSON就可以通过HTTP请求来索引数据。由于本文并非从零开始的Elasticsearch教学，所以关于安装配置等问题这里按下不表，我们直接进入正题，去看看它都可以怎样使用。

### 文档及元数据

在Elasticsearch中的每一个基础单元我们称之为文档，文档中的数据是由键值对组成的JSON对象。和我们从前接触的JSON对象一样，值可以是字符串、数字、布尔类型、对象或数组。

``` json
{
  type:"store",
  store_id:330001,
  name:"独行侠有限公司",
  mobile:"18888888888",
  address:"浙江省杭州市",
  created:1505721167953,
  modified:1552965411756
}
```

除数据外，文档中还有一些元数据用以帮我们检索到数据，三个必须的元数据如下:

| 元数据 | 说明                     |
| ------ | ------------------------ |
| _index | 文档存储的地方(索引)     |
| _type  | 文档代表的对象的类(类型) |
| _id    | 文档的唯一标识           |

其实这有点像我们在电脑中存储文件，`_index`就好像磁盘，`_type`则是分门别类的文件夹，而`_id`就是文件名，三者结合一定能确定文档，就好像根据文件路径查找一样。

### 查询

Elasticsearch提供了强大的全文搜索能力，从简单的文档查询、模糊匹配到组合条件结构化查询，完全可以满足我们的各种需求。

#### 查询索引及类型

`/_search`:在所有索引的所有类型中搜索

`/gb/_search`:在索引`gb`的所有类型中搜索

`/gb,us/_search`:在索引`gb`和`us`的所有类型中搜索

`/g*,u*/_search`:在以`g`或`u`开头的索引的所有类型中搜索

`/gb/user/_search`:在索引`gb`的`user`类型中搜索

`/gb,us/user,tweet/_search`:在索引`gb`和`us`的`user`和`tweet`类型中搜索

`/_all/user,tweet/_search`:在所有索引的`user`和`tweet`类型中搜索

#### 查询子句

##### term

用以精确匹配值:

``` json
{ "term": { "store_id": 330001 }}
{ "term": { "mobile": "18888888888" }}
```

当文档的数据结构存在多层，例如:

```json
{
  type: "team",
  name: "Mavericks",
  member:{
    coach: "Rick Carlisle",
    players: ["Dirk","Barea","Terry"]
  }
}
```

我们想在众多NBA球队中查到主教练是卡莱尔的球队就可以这样写:

``` json
{ "term": { "member.coach": "Rick Carlisle" }}
```

层级再多就继续追加，后面的match语句也是一样。

##### terms

精确匹配多个值，terms的扩展，相当于多个term合并:

``` json
{
  "terms": {
    "tag": [ "JS", "Vue", "ES" ]
  }
}
```

##### range

按照一定范围查询数据，下面的例子就是查询年龄20~30（含20）的数据:

``` json
{
  "range": {
    "age": {
      "gte":  20,
      "lt":   30
    }
  }
}
```

> `gt`:大于
>
> `gte`:大于等于
>
> `lt`:小于
>
> `lte`:小于等于

##### exists和missing

用于查找文档中是否包含指定字段或没有某个字段:

```json
{
  "exists": {
    "field": "title"
  }
}
```

##### match_all

查询到所有文档，是没有查询条件下的默认语句:

``` json
{
  "match_all": {}
}
```

##### match

match是一个查询语句，可以模糊查询关键结果，Elasticsearch会根据搜索的相关度给文档定分值，相关度越高的分值越高，结果越靠前。

``` json
{
  "match": {
    "title": "BROWN DOG"
  }
}
```

该语句可以查询到所有包含`BROWN`或者`DOG`的文档，如果要提高精度则可以使用`operator`参数，默认值为`or`，还可以修改为`and`:

``` json
{
  "match": {
    "title": {
      "query": "BROWN DOG",
      "operator": "and"
    }
  }
}
```

如此，文档中就必须同时包含`BROWN`和`DOG`。还可以通过`minimum_should_match`参数控制精度，值可以设置为数字或百分比表示匹配度。

#### 结构化语句及合并多子句

##### 查询与过滤

我们在使用Elasticsearch时通常会在JSON接口中使用结构化语句。结构化语句分两种:结构化查询（Query DSL）和结构化过滤（Filter DSL）。 查询与过滤语句非常相似，但是它们由于使用目的不同而稍有差异。

过滤语句是为了确定是或者不是，非黑即白，有就是有。而查询语句为了找到文档会去询问匹配度如何，更适合全文搜索。由此可以看出，过滤语句是为了缩小匹配结果，只找最准确的，由于不需要额外计算相关性所以过滤语句性能更高。

结构化查询和结构化过滤需要对应的参数分别为`query`和`filter`，这使得语句创建了上下文关系，`query`和`filter`可以相互包含，相互辅助，这一点我们稍后举例。

##### 合并多子句

出于特定化的需求，我们要将多个条件结合起来得到搜索结果，这就需要`bool`语句了。`bool`语句可以用来合并多个条件查询结果的布尔逻辑。

``` json
{
  "query": {
    "bool": {
      "must": {"term": { "type": "team" }},
      "must_not": {"term": { "name": "Lakers" }},
      "should": [
        {"term": { "area": "West" }},
        {"range": { "rank": { "lte": 8 }}},
      ]
    }
  },
  "_source": ["name","rank"],
  "size": 20
}
```

> `must`:多个查询条件的完全匹配,相当于 `and`，必须全部满足。
>
> `must_not`:多个查询条件的相反匹配，相当于 `not`，必须全部满足。
>
> `should`:至少有一个查询条件匹配, 相当于 `or`，至少满足一个条件。

`must`、`must_not`、`should`可以接受一个条件或一个条件数组。上面的例子就是查询出所有球队中不包含湖人的西部球队或排名前八的球队。由于Elasticsearch默认只返回最多10条结果，我们可以通过`size`参数进行修改。`_source`参数可以配置文档只返回特定字段。

为了优化性能我们可以在查询语句中包含过滤语句:

``` json
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "member.players": "Harris"
        }
      },
      "filter": {
        "term": {
          "area": "West"
        }
      }
    }
  }
}
```

即过滤出西部队中球员名字含有`Harris`的球队。

#### 嵌套对象查询

``` json
{
  "title": "Nest eggs",
  "body":  "Making your money work...",
  "tags":  [ "cash", "shares" ],
  "comments": [
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}
```

Elasticsearch中，对象数组会在索引中被扁平化为键值对形式:

``` json
{
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ],
  "comments.name":    [ alice, john, smith, white ],
  "comments.comment": [ article, great, like, more, please, this ],
  "comments.age":     [ 28, 31 ],
  "comments.stars":   [ 4, 5 ],
  "comments.date":    [ 2014-09-01, 2014-10-22 ]
}
```

因此`comments`中各字段之间的关联消失了，这会导致查询结果不够精确。如果想匹配同一`comments`的多个字段需要借助`nested`查询:

``` json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }},
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "must": [
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age": 28 }}
                ]
              }
            }
          }
        }
      ]
    }
  }
}
```

### 创建文档

借助[elasticsearch.js](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/index.html)这个工具库，我们可以很方便的对文档进行增删改查。

```Javascript
client.create({
  index: 'myindex',
  type: 'mytype',
  id: '1',
  body: {
    title: 'Test 1',
    tags: ['y', 'z'],
    published: true,
    published_at: '2013-01-01',
    counter: 1
  }
});
```

### 删除文档

``` javascript
client.delete({
  index: 'myindex',
  type: 'mytype',
  id: '1'
});
```

### 更新文档

```js
client.update({
  index: 'myindex',
  type: 'mytype',
  id: '1',
  body: {
    // put the partial document under the `doc` key
    doc: {
      title: 'Updated'
    }
  }
})
```

如果是更新整个文档那就把修改后的数据全部放进body中，如果只是需要更新某些字段那就加上`doc`这个参数。

### 批量操作文档

```javascript
client.bulk({
  body: [
    // action description
    { index:  { _index: 'myindex', _type: 'mytype', _id: 1 } },
     // the document to index
    { title: 'foo' },
    // action description
    { update: { _index: 'myindex', _type: 'mytype', _id: 2 } },
    // the document to update
    { doc: { title: 'foo' } },
    // action description
    { delete: { _index: 'myindex', _type: 'mytype', _id: 3 } },
    // no document needed for this delete
  ]
}, function (err, resp) {
  // ...
});
```

| 参数     | 说明                     |
| -------- | ------------------------ |
| `create` | 当文档不存在时创建文档   |
| `index`  | 创建新文档或替换已有文档 |
| `update` | 局部更新文档             |
| `delete` | 删除一个文档             |

批量操作文档需要指定`_index`、`_type`和`_id`，并且在操作后面紧跟`_source`请求体(`delete`操作除外)。

### 结语

到这里，有关Elasticsearch的常见用法就都介绍完了，但强大的Elasticsearch功能绝不止于此，掌握的越多我们在今后处理数据的时候就会更加得心应手。

<br>

*<font color="#bbb" size="3">部分内容参考自以下：</font>*
[https://es.xiaoleilu.com/](https://es.xiaoleilu.com/)(***特别感谢该团队的详尽翻译***)
[https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html)