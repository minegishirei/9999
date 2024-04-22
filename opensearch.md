


# Elastic Searchの使い方



## Indexの削除の仕方

Indexの削除の為には`DELETE` メソッドを使用する。
この、`DELETE` メソッドは`GET`や`POST`と同じHTTPメソッドの一種であり、Opensearchもこのリクエスト形式に基づいています。

例えば、`pages`インデックスを削除したい場合、コンソール画面から次のように入力します。

```sh
DELETE /pages
```


## 変更の方法

```
PUT /products
```



## Indexの作成

PUT リクエストまたは POST リクエストを使用します。

```json
PUT _index_template/daily_logs
{
  "index_patterns": [
    "logs-2020-01-*"
  ],
  "template": {
    "aliases": {
      "my_logs": {}
    },
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "value": {
          "type": "double"
        }
      }
    }
  }
}
```


## Indexのプライマリーキー


```json
PUT /your_index_name/_mapping
{
  "properties": {
    "_id": {
      "type": "keyword",
      "script": {
        "source": "ctx._id = ctx['_source']['sentence_id']",
        "lang": "painless"
      }
    }
  }
}
```





## データ定義 : mapping

mappingとは

https://stackoverflow.com/questions/21461269/how-to-create-unique-constraint-in-elasticsearch-database



https://dev.classmethod.jp/articles/elasticsearch-getting-started-03/






































