Kibana notes
===

+ Kibana version: 4.1
+ Elasticsearch version: 1.5.2
+ Ubuntu 14.04 LTS

Elasticsearch と一緒に動かすことを前提としている。


### [Getting Kibana Up and Running](https://www.elastic.co/guide/en/kibana/current/setup.html)

#### Preconditions

+ Elasticsearch 1.4.4 or laster

#### Installation

```bash
$ mkdir -p /usr/local/Kibana
$ cd /usr/local/Kibana

$ sudo wget https://download.elastic.co/kibana/kibana/kibana-4.1.1-linux-x64.tar.gz
$ sudo wget https://download.elastic.co/kibana/kibana/kibana-4.1.1-linux-x64.tar.gz.sha1.txt
$ sha1sum --check kibana-4.1.1-linux-x64.tar.gz.sha1.txt

$ sudo tar xvzf kibana-4.1.1-linux-x64.tar.gz
$ sudo mv kibana-4.1.1-linux-x64 4.1.1
$ 4.1.1

$ sudo ./bin/kibana
# That's it! runninig on port 5601
```
