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

#### Version確認

```bash
$ http GET 127.0.0.1:9200/
```

```json
{
    "cluster_name": "elasticsearch",
    "name": "Black Fox",
    "status": 200,
    "tagline": "You Know, for Search",
    "version": {
        "build_hash": "62ff9868b4c8a0c45860bebb259e21980778ab1c",
        "build_snapshot": false,
        "build_timestamp": "2015-04-27T09:21:06Z",
        "lucene_version": "4.10.4",
        "number": "1.5.2"
    }
}
```

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
$ http 127.0.0.1:9200/_count < json/to_count_the_number_of_documents_in_the_cluster.json

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

次はデータ(Documents)をRetrieve。

```bash
$ http 127.0.0.1:9200/megacorp/employee/1
```

```json
{
    "id": "1",
    "_index": "megacorp",
    "_source": {
        "about": "I love to go rock climbing",
        "age": 25,
        "first_name": "John",
        "intersts": [
            "sports",
            "music"
        ],
        "last_name": "Smith"
    },
    "_type": "employee",
    "_version": 1,
    "found": true
}
```

`PUT`, `GET`, `DELETE`, `HEAD` を使用することで、index, retrieve, delete, check existence ができる。

### [Search Lite](https://www.elastic.co/guide/en/elasticsearch/guide/current/_search_lite.html)

GETでDocumentsを取得できることはわかった。もっと発展したシンプルな検索を試してみる。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search
```

Documentsが全件取得できる。

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            }
        ],
        "max_score": 1.0,
        "total": 1
    },
    "timed_out": false,
    "took": 2
}
```

query検索も。


```bash
$ http 127.0.0.1:9200/megacorp/employee/_search q==last_name:Smith
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 0.30685282,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            }
        ],
        "max_score": 0.30685282,
        "total": 1
    },
    "timed_out": false,
    "took": 4
}
```

### [Search with Query DSL](https://www.elastic.co/guide/en/elasticsearch/guide/current/_search_with_query_dsl.html)

リッチでフレキシブルな検索のために、QueryDSLがある。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_search_with_query_dsl.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 0.30685282,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            }
        ],
        "max_score": 0.30685282,
        "total": 1
    },
    "timed_out": false,
    "took": 2
}
```

### [More-Complicated Searches](https://www.elastic.co/guide/en/elasticsearch/guide/current/_more_complicated_searches.html)

ex)last_nameがSmithで、24歳以上の従業員を検索したい。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_search_with_query_dsl_more_complicated.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 0.30685282,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            }
        ],
        "max_score": 0.30685282,
        "total": 1
    },
    "timed_out": false,
    "took": 56
}
```

### [Full-Text Search](https://www.elastic.co/guide/en/elasticsearch/guide/current/_full_text_search.html)

RDBでは困るような検索をやってみる。

ただ、DSLのmatch構造に入れるだけ。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_search_full_text.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 0.16273327,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            },
            {
                "_id": "2",
                "_index": "megacorp",
                "_score": 0.016878016,
                "_source": {
                    "about": "I like to collect rock albums",
                    "age": 32,
                    "first_name": "Jane",
                    "intersts": [
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            }
        ],
        "max_score": 0.16273327,
        "total": 2
    },
    "timed_out": false,
    "took": 2
}
```

+ match構造の中に記述した値はor検索のよう。
+ 検索結果はmatchしたscore順にソートされる。

### [Phrase Search](https://www.elastic.co/guide/en/elasticsearch/guide/current/_phrase_search.html)

先ほどの例のように、ひとつひとつの語を検索するときは便利。しかし、フレーズ単位で検索したいときもある。

`match_phrase`という構造を使う。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_search_phrase.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 0.23013961,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            }
        ],
        "max_score": 0.23013961,
        "total": 1
    },
    "timed_out": false,
    "took": 37
}
```

先ほどの`match`フレーズで指定したときとことなり、AND検索になっている。


### [Highlighting Our Searches](https://www.elastic.co/guide/en/elasticsearch/guide/current/highlighting-intro.html)

検索結果ユーザがなぜこの検索結果が返ってきたかを認識させるために、マッチした部分をハイライトしたい。

`highlight`構造を使う。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_highlight.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 0.23013961,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "intersts": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee",
                "highlight": {
                    "about": [
                        "I love to go <em>rock</em> <em>climbing</em>"
                    ]
                }
            }
        ],
        "max_score": 0.23013961,
        "total": 1
    },
    "timed_out": false,
    "took": 208
}
```

match部分が`<em>`Tagで囲まれれた結果が返ってくる。
もっと詳しく知りたい場合は、[Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html)の章で。


### [Analytics](https://www.elastic.co/guide/en/elasticsearch/guide/current/_analytics.html)

分析のための、SQLの`GROUP BY`に似ているが、もっと強力な`aggregations`機能を見ていく。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_use_aggregations.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "aggregations": {
        "all_interests": {
            "buckets": [
                {
                    "doc_count": 2,
                    "key": "music"
                },
                {
                    "doc_count": 1,
                    "key": "forestry"
                },
                {
                    "doc_count": 1,
                    "key": "sports"
                }
            ],
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0
        }
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "interests": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            },
            {
                "_id": "2",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I like to collect rock albums",
                    "age": 32,
                    "first_name": "Jane",
                    "interests": [
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            },
            {
                "_id": "3",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I like to build cabinets",
                    "age": 35,
                    "first_name": "Douglas",
                    "interests": [
                        "forestry"
                    ],
                    "last_name": "Fir"
                },
                "_type": "employee"
            }
        ],
        "max_score": 1.0,
        "total": 3
    },
    "timed_out": false,
    "took": 332
}
```

`doc_count`がカテゴリごとに計算された構造が返ってくる。


また、`aggregations`は入れ子にもできる。

```bash
$ http 127.0.0.1:9200/megacorp/employee/_search <json/to_use_hierarchical_aggregations.json
```

```json
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "aggregations": {
        "all_interests": {
            "buckets": [
                {
                    "avg_age": {
                        "value": 28.5
                    },
                    "doc_count": 2,
                    "key": "music"
                },
                {
                    "avg_age": {
                        "value": 35.0
                    },
                    "doc_count": 1,
                    "key": "forestry"
                },
                {
                    "avg_age": {
                        "value": 25.0
                    },
                    "doc_count": 1,
                    "key": "sports"
                }
            ],
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0
        }
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I love to go rock climbing",
                    "age": 25,
                    "first_name": "John",
                    "interests": [
                        "sports",
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            },
            {
                "_id": "2",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I like to collect rock albums",
                    "age": 32,
                    "first_name": "Jane",
                    "interests": [
                        "music"
                    ],
                    "last_name": "Smith"
                },
                "_type": "employee"
            },
            {
                "_id": "3",
                "_index": "megacorp",
                "_score": 1.0,
                "_source": {
                    "about": "I like to build cabinets",
                    "age": 35,
                    "first_name": "Douglas",
                    "interests": [
                        "forestry"
                    ],
                    "last_name": "Fir"
                },
                "_type": "employee"
            }
        ],
        "max_score": 1.0,
        "total": 3
    },
    "timed_out": false,
    "took": 31
}
```

`interests`fieldでカテゴリ分けされた、ドキュメントにおける`age`fieldの平均値が得られる。
この機能の限界は空のようだよ。

### [Tutorial Conclusion](https://www.elastic.co/guide/en/elasticsearch/guide/current/_tutorial_conclusion.html)

まだまだ紹介しきれない機能がある。

+ suggestions
+ geolocation
+ percolation
+ fuzzy and partial matching

それからパフォーマンスについても。

### [Distributed Nature](https://www.elastic.co/guide/en/elasticsearch/guide/current/_distributed_nature.html)

数百のサーバにスケールするとき、ペタバイトのデータを扱うときについて。
Elasticsearchは分散を意識して生み出されている。分散における複雑さを隠ぺいしようと試みている。

Elasticsearchの中で自動的に行われる操作は次の通り。

+ Documentsの異なるContainers, shardsへのパーティショニング。単一もしくは複数のnodeに保存される。
+ インデクシングや検索負荷を分散するために、クラスターの中でnodeをまたいでshardsのバランスをとる。
+ データロスやハードの故障に備えて、データの冗長化(shardsの複製)。
+ クラスター内のすべてのnodeからのリクエストをあるべきデータを保持するnodeにルーティング。
+ nodeロスからリカバリのためにshardsの再分配や、clusterをより大きくするために新しいnodeをシームレスに統合する。

つぎの章に詳しく書いてある。

+ [Life Inside a Cluster](https://www.elastic.co/guide/en/elasticsearch/guide/current/distributed-cluster.html)
+ [Distributed Document Store](https://www.elastic.co/guide/en/elasticsearch/guide/current/distributed-docs.html)
+ [Distributed Search Execution](https://www.elastic.co/guide/en/elasticsearch/guide/current/distributed-search.html)
+ [Inside a Shard](https://www.elastic.co/guide/en/elasticsearch/guide/current/inside-a-shard.html)

---

### [Modeling Your Data](https://www.elastic.co/guide/en/elasticsearch/guide/current/modeling-your-data.html)

Elasticsearchは違う野獣だ。とくに、SQL界隈から期待人ならそう感じる。多くの利点が生まれる。

+ scale
+ real-timeに近い検索
+ 大量のデータにまたがった分析

しかし、マジックではない。Elasticsearchから多くを得るには、どのように動くか、どのように必要なことを実現するかを理解する必要がある。

エンティティ間のリレーションを扱うことは、専用のリレーショナルストアを利用するほど明確ではない。RDBの黄金ルール:正規化はElasticsearchには
適用できない。

利用できるアプローチの賛否両論を議論していく。

スケールについて議論するうえで、システムにおけるデータの流れを考える必要があり、それに従ってデータモデルをデザインする必要がある。
ログイベントやソーシャルネットワークストリームのような時間軸ベースのデータは静的なドキュメントコレクションとは異なるアプローチが必要となる。


### [Handling Relationships](https://www.elastic.co/guide/en/elasticsearch/guide/current/relations.html)

RDBはリレーションマネージをすることに特化してデザインされている。

+ PKによってエンティティ(or row) はユニークとなる。
+ エンティティは正規化される。ユニークエンティティのデータは一度だけストアされ、関連テーブルはPKだけをストアする。エンティティデータの修正はひとつの場所ないで行われなければならない。
+ クロスエンティティ検索のためにクエリ発行時にエンティth値はジョインされる。
+ 1つのエンティティの変更はACID。
+ ほとんどのRDBは複数エンティティにまたがったACIDトランザクションをサポートしている。

しかし、RDBはフルテキストサーチのサポートが弱い。JOINのコストも高い。異なるハードウェアにおけるJOINはとても高価で実用的でない。これはひとつのサーバに
ストアできるデータ量のリミットを生む。

ElasticsearchはNOSQLのように、フラットなものとして扱う。一つのdocumentは検索にマッチするかどうかを決めるのに必要な情報をすべて含むべき。

**Elasticsearchにおける一つのdocumentの変更はACID。しかし、複数ドキュメントを含むトランザクションではACIDではない。一つのトランザクションが失敗したときに、ひとつ前の状態に戻すロールバックの方法はない**。

FlatWorldの利点

+ インデクシングが高速でロックがない。
+ 検索が高速でロックがない。
+ それぞれのdocumentは独立しているので、大量のデータの複数ノードへの拡張ができる。

しかし、リレーションには問題がある、FlatWorkdと実世界のギャップをうめるためにbridgeが必要。Elasticsearchにおけるリレーショナルデータを扱うための4つの共通したテクニックは以下。

+ Application-side joins
+ Data denormalization
+ Nested objects
+ Parent/child relationships

### [Application-side Joins](https://www.elastic.co/guide/en/elasticsearch/guide/current/application-joins.html)

アプリケーション内でjoinを実装することでrelational databaseをemulateできる。

```bash
$ http PUT 127.0.0.1:9200/my_index/user/1 <json/join_user.json
HTTP/1.1 201 Created
Content-Length: 74
Content-Type: application/json; charset=UTF-8

{
    "_id": "1",
    "_index": "my_index",
    "_type": "user",
    "_version": 1,
    "created": true
}

$ http PUT 127.0.0.1:9200/my_index/blogpost/2 <json/join_blog_post.json
HTTP/1.1 201 Created
Content-Length: 78
Content-Type: application/json; charset=UTF-8

{
    "_id": "2",
    "_index": "my_index",
    "_type": "blogpost",
    "_version": 1,
    "created": true
}
```

ユーザIDが1のユーザによってポストされたBlogを検索するときは、次のようにする。

```bash
$ http 127.0.0.1:9200/my_index/blogpost/_search <json/search_emulate_join.json                                                                  [14/98]HTTP/1.1 200 OK
Content-Length: 273
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "2",
                "_index": "my_index",
                "_score": 1.0,
                "_source": {
                    "body": "It's complicated...",
                    "title": "Relationships",
                    "user": 1
                },
                "_type": "blogpost"
            }
        ],
        "max_score": 1.0,
        "total": 1
    },
    "timed_out": false,
    "took": 50
}
```

Johnという名前のユーザによってポストされたBlogを検索するには、2つのクエリを必要とする。

```bash
$ http 127.0.0.1:9200/my_index/user/_search <json/search_by_name_john_1st_query.json                                                           [14/184]HTTP/1.1 200 OK
Content-Length: 284
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "my_index",
                "_score": 0.19178301,
                "_source": {
                    "dob": "1970/10/24",
                    "email": "john@smith.com",
                    "name": "John Smith"
                },
                "_type": "user"
            }
        ],
        "max_score": 0.19178301,
        "total": 1
    },
    "timed_out": false,
    "took": 9
}

$ http 127.0.0.1:9200/my_index/blogpost/_search <json/search_by_name_john_2nd_query.json                                                       [14/218]HTTP/1.1 200 OK
Content-Length: 272
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "2",
                "_index": "my_index",
                "_score": 1.0,
                "_source": {
                    "body": "It's complicated...",
                    "title": "Relationships",
                    "user": 1
                },
                "_type": "blogpost"
            }
        ],
        "max_score": 1.0,
        "total": 1
    },
    "timed_out": false,
    "took": 3
}
```

この手法のメリットは、データが正規化されていること、デメリットはJOIN　documentsを検索する度に余分なクエリが走ること。

この例では、Johnというユーザが1人しかいなかったが、現実世界のアプリでは、複数人いることが考えられる。よってJohnによるBlogポストを
検索するのに、非常に多くの時間がかかりうる。

このアプローチで向いているのは、ユーザエンティティのdocumentが少なく、めったに変更されないようなとき。このようなときは、キャッシングして1つ目クエリを走らせるのを避けられる。


### [Denormalizing Your Data](https://www.elastic.co/guide/en/elasticsearch/guide/current/denormalization.html)

Elasticsearchから検索をもっともよくする方法は、データをindexするときに、denormalizeすること。
データの冗長なコピーを各documentに持たせることは、joinの手間を排除する。

例えば、指定したユーザ名のユーザがポストしたBlogを検索したいときは、Blog　documentにユーザ名を持たせることで検索することができる。

```bash
$ http 127.0.0.1:9200/denormalize_index/user/1 <json/denormalizing-your-data/put_user.json
HTTP/1.1 201 Created
Content-Length: 83
Content-Type: application/json; charset=UTF-8

{
    "_id": "1",
    "_index": "denormalize_index",
    "_type": "user",
    "_version": 1,
    "created": true
}

$ http 127.0.0.1:9200/denormalize_index/blogpost/2 <json/denormalizing-your-data/put_blogpost_with_user.json
HTTP/1.1 201 Created
Content-Length: 87
Content-Type: application/json; charset=UTF-8

{
    "_id": "2",
    "_index": "denormalize_index",
    "_type": "blogpost",
    "_version": 1,
    "created": true
}
```

Johnと呼ばれるユーザによってポストされたrelationshipsについてのBlogを検索する。

```bash
$ http 127.0.0.1:9200/denormalize_index/blogpost/_search <json/denormalizing-your-data/search_by_user_name.json
HTTP/1.1 200 OK
Content-Length: 337
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "2",
                "_index": "denormalize_index",
                "_score": 0.35258877,
                "_source": {
                    "body": "It's complicated...",
                    "title": "Relationships",
                    "user": {
                        "id": 1,
                        "name": "John Smith"
                    }
                },
                "_type": "blogpost"
            }
        ],
        "max_score": 0.35258877,
        "total": 1
    },
    "timed_out": false,
    "took": 7
}
```

denormalizeの利点はスピード。コスト高なJOINが不要。

### [Field Collapsing](https://www.elastic.co/guide/en/elasticsearch/guide/current/top-hits.html)




---
Next?
+ [Mapping and Analysis](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-analysis.html)


## Plugins

+ [Elasticsearch プラグイン](https://medium.com/hello-elasticsearch/elasticsearch-57c321354911)
