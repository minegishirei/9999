


# Elastic Searchの使い方






# Opensearchの環境構築


### Docker入門 関連記事

- [Docker入門](https://minegishirei.hatenablog.com/entry/2023/09/02/213936)
- [Dockerのダウンロードとインストール(Mac編)](https://minegishirei.hatenablog.com/entry/2023/09/03/143528)
- [Dockerのダウンロードとインストール(Windows編)](https://minegishirei.hatenablog.com/entry/2023/09/04/115946)
- [Dockerのプロキシーの設定](https://minegishirei.hatenablog.com/entry/2023/09/05/120827)
- [Dockerfileの書き方](https://minegishirei.hatenablog.com/entry/2023/09/11/102313)



## 注意点

使用するPCはWindowsでもMacでもLinuxでも構いません

Docker for Desktopを使用しますので、事前にダウンロードをお願いいたします。

## 手っ取り早くOpensearchの環境を作る

Opensearchの実行方法は以下の通りです。
コードを張り付ければ実行されます。

```sh
git clone https://github.com/minegishirei/OpensearchDocker.git
cd OpensearchDocker
docker-compose up
```

<img src="https://github.com/minegishirei/store/blob/main/opensearch/docker/success.png?raw=true">



## docker-compose.ymlの内容

```yml
version: '3'
services:
  opensearch-node1:
    image: opensearchproject/opensearch:latest
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node1
      - discovery.seed_hosts=opensearch-node1
      - cluster.initial_cluster_manager_nodes=opensearch-node1
      - bootstrap.memory_lock=true  # along with the memlock settings below, disables swapping
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m  # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
      - OPENSEARCH_INITIAL_ADMIN_PASSWORD=Ops1234ops!?    # Sets the demo admin user password when using demo configuration, required for OpenSearch 2.12 and higher
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536  # maximum number of open files for the OpenSearch user, set to at least 65536 on modern systems
        hard: 65536
    volumes:
      - ./data:/usr/share/opensearch/data
    ports:
      - 9200:9200
      - 9600:9600  # required for Performance Analyzer
    networks:
      - opensearch-net
    
  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest
    container_name: opensearch-dashboards
    ports:
      - 80:5601
    expose:
      - '80'
    environment:
      OPENSEARCH_HOSTS: '["https://opensearch-node1:9200"]'
    networks:
      - opensearch-net
networks:
  opensearch-net:
```






# Opensearchの用語集

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
[
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
...
]
```


### mapping

Indexの定義。
どのようにデータを格納するかを定義することができます。

Indexのmappingは必ずしも必要なものではなく、データ定義がなくとも「データの追加処理」をすればIndexの作成から自動で対応してくれます。
しかし、例えば`year`の代わりに`string`を使用してほしいことを明示したい時などは、mappingを行い、データ型を明示する必要があります。

明示的なマッピングでは`PUT`リクエストをindexに投げることで、設定の変更が可能です。

```json
PUT sample-index1
{
  "mappings": {
    "properties": {
      "year":    { "type" : "text" },
      "age":     { "type" : "integer" },
      "director":{ "type" : "text" }
    }
  }
}
```

上記は、OpenSearchで新しいインデックスを作成するためのリクエストです。
これにより、"sample-index1"という名前の新しいインデックスが作成され、指定されたマッピングが適用されます。このマッピングでは、"year"フィールドはテキスト型、"age"フィールドは整数型、"director"フィールドもテキスト型として定義されています。


### shard（シャード）

Opensearchで保存されるデータの場所を、`shard`(シャード)と呼ぶ

- `primary shard` : Opensearchで保存されたデータが真っ先に格納される場所。
- `replica shard` : primary shardへ保存された後、次に保存される場所。一つの`primary shard`は複数のレプリカを持つことができ、`primary shard`に何かしらの障害が発生した時に、`replica shard`は`primary shard`へ昇格することができる。


### node（ノード）

opensearchを実行しているインスタンスのこと。







## Indexの作成(インデックステンプレート)


インデックステンプレートは、OpenSearchで新しいインデックスを作成する際に使用される設定のパターンです。
PUT リクエストまたは POST リクエストを使用します。

```json
PUT /users
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

`_clone`エンドポイントに`PUT`メソッドでアクセスすることで、新しいインデックスとしてCloneすることが可能です。
Cloneしたインデックスは元のインデックスとつながっておらず、新規インデックスの変更は元のインデックスには反映されません。

```json
PUT /users/_clone/users_clone
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
  },
  "aliases": {
    "sample-alias1": {}
  }
}
```

jsonボディでは新しいインデックスの設定を定義しています。

- settingsセクションでは、新しいインデックスの設定が定義されています。ここでは、新しいインデックスに2つのシャードと1つのレプリカを設定しています。
- aliasesセクションでは、新しいインデックスに`users_clone`という別名を割り当てるよう指示しています。




## Indexの削除の仕方

Indexの削除の為には`DELETE` メソッドを使用する。
この、`DELETE` メソッドは`GET`や`POST`と同じHTTPメソッドの一種であり、Opensearchもこのリクエスト形式に基づいています。

例えば、`users_clone`インデックスを削除したい場合、コンソール画面から次のように入力します。

```sh
DELETE /users_clone
```


## Indexにdocumentを追加する（データ追加）

Indexにデータを追加するには、`POST`メソッドを使用する。



### bulkインサート

データをまとめて追加したいときは、`bulk`インサートと呼ばれる手法を使う。
エンドポイントの指定は`_bulk`を指定し、jsonの集合を書き込むことで複数データをまとめてインサートできる。

```json
POST /_bulk
{ "index": { "_index": "users", "_id": "1" } }
{
  "id" : 1,
  "name" : "tanaka tarou",
  "description" : "happy new year!"
}
{ "index": { "_index": "users", "_id": "2" } }
{
  "id" : 2,
  "name" : "sakura hanako",
  "description" : "hello world!"
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






































