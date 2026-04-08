PostgreSQLを選択した理由

tttttttttttttttttttt:company

17179246901374075671




### 「なぜPostgreSQLを選択したのですか?」と質問された時に ビシッと回答できるようになりたい。

でRDBMSを使用しているほとんどのプロダクトは PostgreSQLを採用しています。
DynamoDB等の採用事例もありましたが、 RDBMSを選択する場合はPostgreSQLを選択していました。

「必ずしもPostgreSQLを使用しなくてはならない」と言うルールはないのですが、
各々が意識せずにPostgreSQLを選択している状況です。

「なぜPostgreSQLを選択したのですか?」と質問された時に ビシッと回答できるようになりたい。
そう思いADRとして記しておこうと思ったのが、この記事を書くきっかけでした。


## 結論

我々がPostgreSQLを選択した理由は以下の二つに集約できると思います。

- 機能の豊富さ
- 使い慣れている


## 1. 機能が多い

我々が作成するプロダクトは PoC(仮説検証) としてスタートしたものも少なくありません。
そのため当初要件になかった機能を求められるケースも想定されますが、それを満たす機能が最初から備わっていれば開発工数を削減できます。

**PostgreSQLは他のRDBMSよりも機能が豊富であると考え採用しました。**
その方が予期しない変更にも柔軟に対応できると思ったからです。

> PostgreSQL は MySQL よりも多くの機能を提供するオブジェクトリレーショナルデータベース管理システムです。
> 
> [AWS : MySQL と PostgreSQL はどのように異なりますか?](https://aws.amazon.com/jp/compare/the-difference-between-mysql-vs-postgresql/)


### 例1 : JSONB型（非構造化データ）

スキーマが固まりきっていない初期段階では、一部のデータをJSONB型として保存することで、RDBの良さを活かしつつNoSQLのような柔軟な開発が可能です。
他のデータベースよりも、インデックスの貼り方や演算子が強力な印象があります。

「リレーショナルな枠組みの中で、JSONを効率的に扱う」と言うよりは「ドキュメント型DB（MongoDBなど）に近い使い勝手」 として実装されています。

```sql
-- usersテーブルの idx_full_jsonカラムをjsonとして定義
CREATE INDEX idx_full_json ON users USING GIN (data);

-- 1. ネストした深い階層の検索
SELECT * FROM users WHERE data @> '{"profile": {"location": "Tokyo"}}';
-- 2. 配列の中に特定の要素があるか
SELECT * FROM users WHERE data->'tags' ? 'premium';
-- 3. 「キーが存在するか」の検索
SELECT * FROM users WHERE data ? 'discount_code';
```

### 例2 : ベクトル型

大きな違いの一つはベクトル型についてです。

- PostgreSQLには`pgvector`と呼ばれる拡張機能があり、PostgreSQL にベクトル検索機能を追加できます。
距離計算の方法も多様で、ユークリッド距離、内積、コサイン類似度 などなどの計算方法が提供されています。
- 他のデータベースですと、たとえばMySQLもバージョン 9.0 あたりから本格的なベクトル型のサポートが開始されましたが、長らく実装されていませんでした。


### CREATE EXTENSION の強力さ

データベース内に拡張機能を追加できる PostgreSQLの `CREATE EXTENSION` はとても強力な印象を持ちます。
Amazon RDS for PostgreSQL環境だと、以下の一文で`pg_vector`(ベクトル型) を有効化することができます。

```sql
CREATE EXTENSION vector;
```

そして拡張機能を取り巻くエコシステム/コミュニティがもっとも強力であり、**開発者が「これがあったら便利だな」と思う機能を、本体のリリースサイクルに関係なく配布・導入できます。**

> **PostgreSQL Extension Networkについて**
> 
> PGXN（PostgreSQL Extension Network）は、オープンソースのPostgreSQL拡張ライブラリの中央配布システムです。
> 
> https://manager.pgxn.org/howto

PostgreSQLで使用可能な拡張機能は以下で検索できます。

https://pgxn.org/


### PostgreSQLは「オブジェクト」リレーショナル

PostgreSQLは「オブジェクト・リレーショナル」データベースであり、拡張機能等を活用して、データ型、演算子、インデックスの方法などを後から定義できるように設計されています。

> PostgreSQL はオブジェクトリレーショナルデータベースです。つまり、PostgreSQL では、データをプロパティ付きのオブジェクトとして保存できます。オブジェクトは、Java や.NET などの多くのプログラミング言語で一般的なデータ型です。オブジェクトは、親子関係や継承などのパラダイムをサポートします。
> 
> [AWS : MySQL と PostgreSQL はどのように異なりますか?](https://aws.amazon.com/jp/compare/the-difference-between-mysql-vs-postgresql/)




## 2. 使い慣れているから

開発当初のメンバーがPostgreSQLを多用しており、現在もその流れを引き継いでいるのも一つの要因です。
使用するデータベースを変更する場合、以下の点の違いが学習難易度を上げてしまいます。

### 型の違い

たとえば`CREATE` 文でテーブルを作成しようとした時。
他のデータベースですと、MySQLにおいては `INT UNSIGNED` という型がありますが、 PostgreSQLにはありません。
また、高精度少数型は MySQLにおいては `DECIMAL` ですが PostgreSQLでは `NUMERIC` で表されます。
小さい違いかもしれませんが、**これらの型の違いは開発者にとっては無視できない負荷になります。**


### 接続ツールの違い

ローカル環境でテストデータベースを作成しアクセスしようとした時、さらに厄介な違いが発生します。

- PostgreSQLの場合、 ユーザー名は `-U(大文字)` オプションでの指定を行いますが...

```bash
psql -U ユーザー -d DB名
```

- MySQLの場合、 ユーザー名は `-u(小文字)` での指定を行います。

```bash
mysql -u ユーザー -p DB名
```

このように、データベースにコマンドラインから接続しようとするとオプションの違いが出てきます。
ポート番号の場合は、もっとややこしいです。

- PostgreSQLの場合、ポート番号は `-p` で表されます

```bash
psql -h ホスト名 -p 5432 -U ユーザー名 -d データベース名
```

- MySQLの場合、 ポート番号は `-P` で表されます。 **そして小文字の`-p`はパスワード入力のオプションです。**

```bash
mysql -u ユーザー名 -P 3306
```

あまりにもややこしいので、元々の流れを汲んでPostgreSQLに統一されていったのだと思います。


## 3. 標準のSQLに準拠しているため

多くのRDBMSには独自の「方言」がありますが、
PostgreSQLは標準的な書き方に忠実な印象を持ちます。

> PostgreSQL development aims for conformance with the latest official version of the standard where such conformance does not contradict traditional features or common sense. 
> (PostgreSQLの開発は、従来の機能や常識に反しない限り、最新の公式標準規格に準拠することを目指しています。)
> 
> https://www.postgresql.org/docs/current/features.html

PostgreSQLで書けるようになったエンジニアは別のデータベースに移った際も、「標準的な書き方」を知っているため、スムーズに適応できると考えています。



## まとめ

私たちがPostgreSQLを選んだのは、それが **「今日必要な機能」を満たしているだけでなく、「明日必要になるかもしれない機能」をすでに持っており、かつ「開発者が最もストレスなく扱える」** データベースだからです。












