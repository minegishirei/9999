


# Elastic Searchの使い方



## Opensearchの用語集

Opensearchでは扱う言葉が紛らわしいという性質があり、
「MySQLなどのほかのデータベースを知っているし大体同じだろ」と考えて用語の理解をおろそかにすると逆に理解に苦しむことがあります。
他のデータベースシステムを知っていたとしても、Opensearchの用語は押さえておきましょう。

### `Document`

  - Opensearchでは**格納するデータの一行を`document`と呼んでいます。**
  - RDSでの「行」に該当する言葉です。
  - Opensearchでは、辞書形式でデータを格納していきます。

```json
{
  "id": 1,
  "name": "こころ",
  "body": "私はその人を常に先生と呼んでいた。だからここでもただ先生と書くだけで本名は打ち明けない。これは世間を憚かる遠慮というよりも、その方が私にとって自然だからである。私はその人の記憶を呼び起すごとに、すぐ「先生」といいたくなる。筆を執っても心持は同じ事である。よそよそしい頭文字などはとても使う気にならない。",
  "author": "夏目漱石"
}
```

from https://qiita.com/hkano/items/a097d6c2fa103752dce9



### `Field` 

- Opensearchでは、**`Document`を構成する、keyとvalueのペアのことを`Field`と呼んでいます。**

```json
  "author": "夏目漱石"
```


### `Index`

- Opensearchでは、一つのテーブルのことを`Index`と呼んでいます。
- 一つの`Index`には、大量の`Document`が格納されます。

```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1.0708848,
    "hits": [
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "2",
        "_score": 1.0708848,
        "_source": {
          "id": 2,
          "name": "羅生門",
          "body": "ある日の暮方の事である。一人の下人が、羅生門の下で雨やみを待っていた。広い門の下には、この男のほかに誰もいない。ただ、所々丹塗の剥げた、大きな円柱に、蟋蟀が一匹とまっている。羅生門が、朱雀大路にある以上は、この男のほかにも、雨やみをする市女笠や揉烏帽子が、もう二三人はありそうなものである。それが、この男のほかには誰もいない。",
          "author": "芥川龍之介"
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "4",
        "_score": 0.9264894,
        "_source": {
          "id": 4,
          "name": "坊っちゃん",
          "body": "親譲りの無鉄砲で小供の時から損ばかりしている。小学校に居る時分学校の二階から飛び降りて一週間ほど腰を抜かした事がある。なぜそんな無闇をしたと聞く人があるかも知れぬ。別段深い理由でもない。新築の二階から首を出していたら、同級生の一人が冗談に、いくら威張っても、そこから飛び降りる事は出来まい。弱虫やーい。と囃したからである。小使に負ぶさって帰って来た時、おやじが大きな眼をして二階ぐらいから飛び降りて腰を抜かす奴があるかと云ったから、この次は抜かさずに飛んで見せますと答えた。",
          "author": "夏目漱石"
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "3",
        "_score": 0.85215265,
        "_source": {
          "id": 3,
          "name": "走れメロス",
          "body": "メロスは激怒した。必ず、かの邪智暴虐の王を除かなければならぬと決意した。メロスには政治がわからぬ。メロスは、村の牧人である。笛を吹き、羊と遊んで暮して来た。けれども邪悪に対しては、人一倍に敏感であった。きょう未明メロスは村を出発し、野を越え山越え、十里はなれた此のシラクスの市にやって来た。",
          "author": "太宰治"
        }
      }
    ]
  }
}
```










## Indexの作成(インデックステンプレート)


インデックステンプレートは、OpenSearchで新しいインデックスを作成する際に使用される設定のパターンです。
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


## Indexの複製


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






































