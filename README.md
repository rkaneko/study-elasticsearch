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
+ RESTful APIを使う

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


## Plugins

+ [Elasticsearch プラグイン](https://medium.com/hello-elasticsearch/elasticsearch-57c321354911)
