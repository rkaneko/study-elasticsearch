Elasticsearch + Kibana 調査メモ
===

# Elasticsearch

## はじめに

+ [Elasticsearchチュートリアル](http://code46.hatenablog.com/entry/2014/01/21/115620) 主に機能説明
+ [Elasticsearch入門](http://sssslide.com/speakerdeck.com/johtani/elasticsearchru-men) 構成と機能説明、注意事項
+ [実践Elasticsearch](http://engineer.wantedly.com/2014/02/25/elasticsearch-at-wantedly-1.html)
+ [新卒エンジニア研修で「Elasticsearch の歩き方」について話した](http://blog.inouetakuya.info/entry/2014/12/11/180106)
+ [N-gramと形態素解析](http://gihyo.jp/dev/serial/01/make-findspot/0006)


## Installation

+ [Official doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)

**TODO automation with ansible**

### Marvel (Kibana?)

マネジメント・モニタリングツール。開発利用はフリー。Senseっているインタラクティブツールがついてくる。ブラウザからElasticsearchとおしゃべりできる。

```bash
# install marvel
$ ./bin/plugin -i elasticsearch/marvel/latest
```
http://localhost:9200/_plugin/marvel/  
http://localhost:9200/_plugin/marvel/sense/ にアクセスすると見れる。

Ref: [Installing Marvel](https://www.elastic.co/guide/en/elasticsearch/guide/current/_installing_elasticsearch.html#marvel)

### Talking to Elasticsearch

+ Java を使う
l RESTful APIを使う

２つの手段がある。様々な言語でクライアントがある。
汎用的に学習したいので、RESTfulを試してみる。

Ref: [Talking to Elasticsearch]( https://www.elastic.co/guide/en/elasticsearch/guide/current/_talking_to_elasticsearch.html)

公式ではcurlを使用しているが、ここでは[httpie](https://github.com/jakubroztocil/httpie) を使う。

#### cluster内のdocumentの数をcountする

```bash
$ http GET 127.0.0.1:9200/_count < json/to_count_the_number_of_documents_in_the_cluster.json

HTTP/1.1 200 OK
Content-Length: 63
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0,
        "successful": 3,
        "total": 3
    },
    "count": 44432
}
```

### Document Oriented

### [Finding Your Feet](https://www.elastic.co/guide/en/elasticsearch/guide/current/_finding_your_feet.html)

Elasticsearchでできることをおおよそ理解するために、

Employee Directoryを構築してみる

ビジネス要件は以下の通り

+ 複数のタグや数字、文字列を含むことができるデータ
+ その従業員の完全な詳細を取得できる
+ 構造化された検索(ex: 30歳以上の従業員)
+ フルテキスト検索、複雑なフレーズによる検索
+ マッチしたドキュメントにおける部分的なテキストをハイライトして返す
+ データを分析できるダッシュボードを構築できる管理ができる

### [Indexing Exployee Documents](https://www.elastic.co/guide/en/elasticsearch/guide/current/_indexing_employee_documents.html)

まずは、従業員データのストア。

一つのドキュメント(レコード)は一人の従業員データを表す。

Elasticsearchでは、**データのストアすることをindexingと呼ぶ**。

|          |          |          |          |         |
|:---------|:---------|:---------|:---------|:--------|
|RDB       | Databases|Tables    |Rows      |Columns  |
|Elasticsearch|Indices|Types     |Documents |Fields   |

Clusterは複数のIndicesを含める。

#### ElasticsearchではIndexという言葉が複数定義されている。

+ Index (noun)
  - RDBでいうdatabase
+ Index (verb)
  - SQLでいうinsert
+ Inverted index
  - data検索のためにB-treeのようなindexを作ること

サンプルを整理すると、

Elasticsearch clusterに`megacorp` index (database)を作り、このindexに
`employee` type (table)を作る。

```bash
# 暗黙的に、indexとtypeも作られる /index/type/id のようにpathを並べる
$ http PUT 127.0.0.1:9200/megacorp/employee/1 <json/to_index_a_sample_data.json
```

### [Retrieving a Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/_retrieving_a_document.html)



## Plugins

+ [Elasticsearch プラグイン](https://medium.com/hello-elasticsearch/elasticsearch-57c321354911)
