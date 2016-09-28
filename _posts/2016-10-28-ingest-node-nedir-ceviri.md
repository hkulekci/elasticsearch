---
layout: post
title: Ingest Node Nedir (Çeviri)
categories:
- blog
summary: Elasticsearch 5.0 sürümü ile birlikte yeni gelen yeni bir özelliği nasıl kullanacağımıza kısaca bir göz atalım. "Ingest Node".
---

> Elasticsearch 5.0 sürümü ile birlikte yeni gelen yeni bir özelliği nasıl 
kullanacağımıza kısaca bir göz atalım. "Ingest Node".

## "Ingest Node" Nedir?

"Ingest Node" genel veri dönüştürmeleri ve zenginleştirmeleri için kullanabileceğiniz 
yeni bir Elasticsearch node türüdür. 

Her bir görev bir `processor` olarak temsil edilir. Her bir `processor` bir `pipeline`lar 
ile oluşur. (Buradaki terimlerin çevirilerini yapmadım ancak `processor`'ü veri işleyen 
bir yapı, `pipeline`'ı ise veri işleme hattı olarak düşünebilirsiniz.)

Şu anda hali hazırda dahili olarak 20 tane `processor` bulunmaktadır. Örneğin: 
grok, date, gsub, lowercase/uppercase, remove ve rename. Tma listeye 
[şu adresten](https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest-processors.html)(döküman ingilizcedir) ulaşabilirsiniz.

Bunun yanında `ingest` için ayrıca 3 tane eklenti bulunmaktadır:

 - [Ingest Attachment](https://www.elastic.co/guide/en/elasticsearch/plugins/master/ingest-attachment.html) : Powerpoint, Excel Spreadsheets, ve PDF gibi ikili 
 kodlanmış dökümanları metin ve üstveri(metadata) haline getirir. 
 - [Ingest Geoip](https://www.elastic.co/guide/en/elasticsearch/plugins/master/ingest-geoip.html) : IP adreslerin coğrafi yerine göre metinsel hale dönüştürür. 
 - [Ingest User Agent](https://www.elastic.co/guide/en/elasticsearch/plugins/master/ingest-user-agent.html) : Tarayıcıların ve diğer uygulamaların HTTP'yi kullanırken 
 kullandıkları kullanıcı aracısı(user agent) verisini işler ve bilgilerini yapısal bir hale 
 getirir.
 
### Ingest Pipeline Oluşturma ve Kullanma

`_ingest` arayüzü ile kolayce yeni bir ingest pipeline oluşturabilirsiniz. 

```
PUT _ingest/pipeline/rename_hostname
{
  "processors": [
    {
      "rename": {
        "field": "hostname",
        "target_field": "host",
        "ignore_missing": true
      }
    }
  ]
}
```

Bu örnekte `rename_hostname` adında bir pipeline oluşturduk ve bu sadece 
`hostname` alanını alıp adını değiştirip `host` olarak yazıyor. Eğer `hostname`
yok ise `processor` işlemine hata vermeden devam ediyor.

Pipeline'ı kullanmak için çeşitli yollar mevcut. 

Direk Elasticsearch API üzeriden kullanırken, `pipeline` parametresini url parametresi
olarak göndermeniz gerekir. Örneğin :

```
POST server/values/?pipeline=rename_hostname
{
  "hostname": "myserver"
}
```

Logstash ile kullanırken çıktı ayarlamaları arasına `pipeline` parametresini 
eklemelisiniz:

```
output {
  elasticsearch {
    hosts => "192.168.100.39"
    index => "server"
    pipeline => "rename_hostname"
  }
}
```

Benzer şekilde Beat uygulamasını kullanırken yine çıktı ayarları arasına `pipeline` 
parametresini eklemeniz gerekir:

```
output.elasticsearch:
  hosts: ["192.168.100.39:9200"]
  index: "server"
  pipeline: "convert_value"
```

> Note: Alpha 5.0 sürümünde, Beat ayarlarında `parameters.pipeline` şeklinde 
> kullanmalısınız

### Simulasyon 

Yeni bir pipeline oluşturduğunuzda, gerçek veri üzerinde kullanmadan önce  test 
edebilmek ve bir hata fırlatıp fırlatmayacağını araştırmak  gerçekten çok önemlidir.

Bunun için bir [Simulate API](https://www.elastic.co/guide/en/elasticsearch/reference/master/simulate-pipeline-api.html) mevcuttur:

```
POST _ingest/pipeline/rename_hostname/_simulate
{
  "docs": [
    {
      "_source": {
        "hostname": "myserver"
      }
    }
  ]
}
```

Sunçlar bize alanın başarılı bir şekilde değiştiğini gösterecektir:

```
       [...]
        "_source": {
          "host": "myserver"
        },
        [...]
```

### Gerçek bir örnek : Web Günlükleri

Gerçek dünyadan bir örnek ile devam edelim: Web Günlükleri.

Bu "[Combined Log Format](http://httpd.apache.org/docs/current/mod/mod_log_config.html)" içerisinde Apache httpd ve nginx tarafından desteklenen  girdi günlüğünün bir örneği: 

```
212.87.37.154 - - [12/Sep/2016:16:21:15 +0000] "GET /favicon.ico HTTP/1.1" 200 3638 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36"
```

Gördüğünüz gibi birkaç parça bilgiden oluşam bir metin: IP adres, zaman damgası, 
kullanıcı aracısı ve daha fazlası. Hızlı bir arama ve görselleştirme sunmak için verimizi
parçalara ayırıp Elasticsearch'de kendi alanları içerisine koyacağız. İsteğin nerden
geldiğini bilmemiz de gerçekten çok yararlı olurdu. Bunları hepsini aşağıkdaki 
pipeline ile yapabiliriz.

```
PUT _ingest/pipeline/access_log
{
  "description" : "Ingest pipeline for Combined Log Format",
  "processors" : [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \\[%{HTTPDATE:timestamp}\\] \"%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}\" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int})   %{QS:referrer} %{QS:agent}"]
      }
    },
    {
      "date": {
        "field": "timestamp",
        "formats": [ "dd/MMM/YYYY:HH:mm:ss Z" ]
      }
    },
    {
      "geoip": {
        "field": "clientip"
      }
    },
    {
      "user_agent": {
        "field": "agent"
      }
    }
  ]
}
```
Örnek 4 `processor` içermektedir: 

 - `grok` regular expression ile metin halindeki günlük verisini işleyip kendi alanları olan yapısal bir hale getirmektedir. 
 - `date` dökümanın zaman dangası bilgisini tanımlamaktadır.
 - `geoip` istekte bulunanın IP adresini alıp iç bir veritabanına sorarak coğrafi konumunu belirlemektedir. 
 - `user-agent` kullanıcı aracı bilgisini metin olarak alıp yapısal bir hale getirmektedir. 
 
Son iki `processor` Elasticsearch'e eklenti olarak gelmektedir. Onları öncelikli olarak 
kurmalıyız:

```
bin/elasticsearch-plugin install ingest-geoip
bin/elasticsearch-plugin install ingest-user-agent
```

Pipeline'ı test etmek için Simutale API'ı kullanabiliriz :

```
POST _ingest/pipeline/httpd_weblogs/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "212.87.37.154 - - [12/Sep/2016:16:21:15 +0000] \"GET /favicon.ico HTTP/1.1\" 200 3638 \"-\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36\""
      }
    }
  ]
}
```

Sonuç bize çalıştığını göstercektir:

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_type": "_type",
        "_id": "_id",
        "_source": {
          "request": "/favicon.ico",
          "agent": "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36\"",
          "geoip": {
            "continent_name": "Europe",
            "city_name": null,
            "country_iso_code": "DE",
            "region_name": null,
            "location": {
              "lon": 9,
              "lat": 51
            }
          },
          "auth": "-",
          "ident": "-",
          "verb": "GET",
          "message": "212.87.37.154 - - [12/Sep/2016:16:21:15 +0000] \"GET /favicon.ico HTTP/1.1\" 200 3638 \"-\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36\"",
          "referrer": "\"-\"",
          "@timestamp": "2016-09-12T16:21:15.000Z",
          "response": 200,
          "bytes": 3638,
          "clientip": "212.87.37.154",
          "httpversion": "1.1",
          "user_agent": {
            "patch": "2743",
            "major": "52",
            "minor": "0",
            "os": "Mac OS X 10.11.6",
            "os_minor": "11",
            "os_major": "10",
            "name": "Chrome",
            "os_name": "Mac OS X",
            "device": "Other"
          },
          "timestamp": "12/Sep/2016:16:21:15 +0000"
        },
        "_ingest": {
          "timestamp": "2016-09-13T14:35:58.746+0000"
        }
      }
    }
  ]
}
```

### Sonraki 

İkinici kısımda Filebeat kullanarak ingest pipeline nasıl oluşturulur göstereceğiz ve
Elasticsearch ve Kibana ile günlükleri görselleştireceğiz.

> Note : Bazı terimlerin çevirisinde anlama sıkıntısı ortaya çıkaracağı için çevirisini 
yapmadım. 