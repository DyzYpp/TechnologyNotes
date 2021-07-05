```gherkin
# 最小切分
GET _analyze
{
  "analyzer": "ik_smart",
  "text": ["段宇哲是帅哥"]
}

# 最小粒度划分
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": ["段宇哲是帅哥"]
}

# /索引名/~类型名~/文档id
PUT /test1/type/1
{
  "name": "Dyz",
  "unit": "Java后端开发工程师"
}

PUT /test2
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "age":{
        "type": "long"
      },
      "birthday":{
        "type": "date"
      }
    }
  }
}

GET test2


PUT /test3/_doc/1
{
  "name": "段宇哲1号",
  "age": 24,
  "birth": "1996-08-04"
}

GET test3

# 查看健康值
GET _cat/health

GET _cat/indices?v

# 修改 PUT 直接覆盖  是修改的话，版本号会增加  老方发，不腿甲能使用

POST /test3/_doc/1/_update
{
  "doc":{
    "name": "段"
  }
}
```

