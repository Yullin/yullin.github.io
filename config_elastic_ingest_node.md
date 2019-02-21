Title: configure elasticsearch ingest node
Date: 2018-08-20 18:08
Modified: 2018-08-20 18:08
Category: TECH SKILLS
Tags: elk, elasticsearch, logstash, kibana, ingest node
Slug: config-elasticsearch-ingest-node
Authors: Yullin

在elastic 5.*版本开始支持ingest node功能，他可以在一定程度上替代Logstash的处理功能，只要是熟悉了其支持的格式之后，配置还是比较简单的。  
### 1.测试可以用如下命令
>curl  -v -H 'Content-Type: application/json' -X POST 'http://10.2.4.34:9200/_ingest/pipeline/_simulate' -d@filebeat.test.json

filebeat.test.json的内容如下：  
```
{
  "pipeline" : {
    "description" : "nginx access log",
    "processors": [
      {
        "grok": {
          "field": "message",
          "patterns": ["%{IP:client_ip} %{TIMESTAMP_ISO8601:iso_time} %{BASE10NUM:timestamp} %{BASE10NUM:request_time} %{BASE10NUM:request_length} %{INT:connection_id} %{INT:connection_requests} %{PATH:uri} \"%{WORD:request_method} %
{NOTSPACE:request} HTTP/%{NUMBER:httpversion}\" %{NUMBER:response_status} (?:%{NUMBER:bytes}|-) (?:%{NUMBER:error_code}|-) \"(?:%{WORD:error_msg}|-)\" %{QS:referrer} %{QS:agent} \"%{DATA:x_forwarded_for}\" \"%{DATA:x_is_ip}\" %{HOSTN
AME:server_host} %{URIHOST:upstream_addr} %{BASE10NUM:upstream_response_time} %{NUMBER:upstream_status} \"(?:%{IP:x_is_client_ip}|-)\""],
          "ignore_missing": true
        }
      }
    ]
  },
  "docs" : [
    { "_source": {
        "message": "101.95.128.162 2018-08-09T19:26:41+08:00 1533814001.040 0.061 407 27500839 1 /wp-includes/js/jquery/jquery.js \"GET /wp-includes/js/jquery/jquery.js?ver=1.12.4 HTTP/1.1\" 200 97184 - \"-\" \"http://blog.zdao.com/\
" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36\" \"-\" \"-\" blog.zdao.com 106.75.223.130:80 0.061 200 \"101.95.128.162\"" 
      }
    }
  ]
}
```

配置文件filebeat.yml如下：
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /data/front/webcc_online/*/*/*/10.2.0.247/5032/*.log
  fields:
    type: "access-log"

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 3

setup.template.name: "ignore-this"
setup.template.pattern: "ignore-this"

setup.kibana:
  host: "localhost:5601"
  
output.elasticsearch:
  hosts: ["10.2.4.34:9200"]
  indices:
    - index: "filebeat-nginx-access-%{+yyy.MM.dd}"
      when.equals:
        fields.type: "access-log"
  pipelines:
    - pipeline: "pipeline-nginx-access"
      when.equals:
        fields.type: "access-log"
```
