---
layout: post
title: Mapping Nedir?
categories:
- blog
---

Bir önceki yazımda daha çok `Type` kavramından bahsetmiştim. `Mapping` hakkında da 
kısa bir kaç cümle söylemiştim. Elasticsearch'de `mapping` verilerin index'lerde nasıl 
tutulacağını belirleyen yapılarıdır. Normalde Lucene'de `mapping` yapısı yoktur. Lucene'de
veriler `field-value` olarak tutulur ve değerlerin sayı, `string` ya da tarih mi olduğunu 
önemsemez. Tüm değerler `opaque byte`lar halinde tutulur. 

Elasticsearch'ün en güzel özelliklerinden birisi hızlıca verilerini oluşturabilrmenizi ve 
erişebilmenizi sağlar. Bir dökümanı index'lemek için bir index oluşturmanıza ya da bir 
`mapping type` belirlemenize, alanlarınızı tanımlamanıza gerek yoktur. 

```
PUT core_index/user/1
{
	"name": "Haydar KULEKCI",
	"age" : 1
}
```

ES otomatik olarak anlayacaktır ve bir index bir `type` ve alanlarınız otomatik olarak 
tiplerinize göre oluşturacaktır. Yukarıdaki sorguyu çalıştırdığınızda aşağıdaki gibi bir 
mapping otomatik olarak oluşacaktır.

```
{
   "core_index": {
      "mappings": {
         "user": {
            "properties": {
               "age": {
                  "type": "long"
               },
               "name": {
                  "type": "string"
               }
            }
         }
      }
   }
}
```

Gördüğünüz gbi `name` alanı `string` olarak `age` alanı da `long` olarak tanımlandı. 
ES'de yeni bir veriyi içeri almak aslında bu kadar kolay. İşler her zaman ub kadar da
kolay olmayabiliyor. Bazı veriler sayı gelse dahi string olarak tutmamız gerekebilir. 
Ya da ilk değer sayı denk gelen bir index oluşturmuş olabiliriz. Ilk olarak `long` olarak 
otomatik oluşturulmuş alana daha sonrasında `string` geldiğinde sıkıntı yaşayabiliriz.

Her bir index'te bir veya daha fazla `type` vardır. Bu `type`lar index'leri belirli gruplara 
böler. Her bir `type`'daki dökümanlarda dökümanı ilişkilendirmek için kullanılan 
`_index`, `_type`, `_id` ve `_source` gibi `meta` alanlar(field) vardır. Bunlar dışında 
kendi belirleyeceğiniz alanlarınız yani `properties`ler vardır. 

Daha iyi anlamak için bir örnek belirleyelim ve bunun üzerinden devam edelim. Aşağıda 
aynı index içerisinde iki `type` tanımlayalım ve bunların `mapping` bilgilerini yani index'te 
verilerin nasıl tutulacağını biz belirleyelim.

```
POST core_index 
{
  "index": {
    "number_of_replicas": "1",
    "number_of_shards": "2"
  },
  "mappings": {
    "user": {
      "_all": {
        "enabled": false
      },
      "properties": {
        "title": {
          "type": "string"
        },
        "name": {
          "type": "string"
        },
        "age": {
          "type": "integer"
        }
      }
    },
    "post": {
      "properties": {
        "title": {
          "type": "string"
        },
        "body": {
          "type": "string"
        },
        "user_id": {
          "type": "string",
          "index": "not_analyzed"
        },
        "created": {
          "type": "date"
        }
      }
    }
  }
}
```

Yukarıdaki index'i oluştururken görüldüğü üzere `user` ve `post` adında iki tane `type` 
oluşturuyoruz ve `mapping` bilgilerini yazıyoruz. `Type`larda alanları `properties` alanı
altında belirtiyoruz. `Properties` altındaki yapı aşağıdaki gibidir:

```
        "field-name": {
          "type": "type-name",
          "other-attribute-name": "value"
        },
```

`field-name` kısmını siz alana vereceğiniz ismi yazıyorsunuz ve `type-name` kısmına da o 
alanın tipini yazıyorsunuz. Yukarıda göreceğiniz gibi `user` `type`'ı için `title`, `name`, `age` 
alanları var ve sırasıyla `string`, `string` ve `integer` tipinde. Aynı index'teki `post` `type`ınde 
ise `title`, `body`, `user_id` ve `created` alanları bulunuyor ve bu alanlar sırasıyla `string`,
`string`, `string` ve `date` tipindedir. 

Elasticsearch'te  `string`, `date`, `long`, `double`, `boolean` ve `ip` basit tipleri mevcuttur.
Bunlar dışında `object` ve `nested` tipleri bulunmaktadır. Daha özelleşmiş olan `geo_point`, 
`geo_shape`, ve `completion` tipleri de bulunmaktadır. Tİpleri ilerleyen zamanlarda 
kullandıkça açıklayacağız. Yine de erkenden bilgi sahibi olmak isterseniz 
[buradan](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)
bilgi alabilirsiniz. 

Yukarıdaki örnekte göreceğiniz `post` type'ındaki `user_id` alanında `index` diye ayrı bir 
özellikte mevcut. Bu özellik alandaki değerin nasıl index'e alınacağını belirler. Eğer bu alan 
hiç yazılmaz ise `standart analyzer` ile analiz edilerek index'e alınır. Buraya kendi 
oluşturduğunuz `analyzer`ları ya da varsayılanları kullanabilirsiniz. 

