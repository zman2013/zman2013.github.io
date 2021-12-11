---
title: elasticsearch command
date: 2021-12-11 14:54:06
tags:
---

# 基础命令
1. 创建索引
创建名为`xxx-span-2021-12-11`的索引
```
PUT /xxx-span-2021-12-11
{
  "mappings": {
      "properties": {
        "tags.source.uri": {
          "type":       "keyword"
        }
      }
  }
}
```

2. 查看索引和映射
有了映射才能在 Kibana 中做查询和图标
```
GET /xxx-span-2021-12-11
GET /xxx-span-2021-12-11/_mapping
```

3. 删除索引
```
DELETE /index_*
```
[LINK](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_deleting_an_index.html)

4. 模板操作
```
GET _template/xxx-span_template
DELETE _template/xxx-span_template
```

5. 查询
```
GET /xxx-span-2021-12-11/_search
{
  "query": {
    "match": {
      "traceId": {
        "query": "bf2a201e835028ecae23c324e85b9855"
      }
    }
  }
}
```

# Links 
[Kibana 可视化](https://www.elastic.co/guide/cn/kibana/current/tutorial-visualizing.html)