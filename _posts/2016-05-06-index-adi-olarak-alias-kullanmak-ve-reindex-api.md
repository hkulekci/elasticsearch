---
layout: post
title: Index adı olarak `Alias` Kullanmak ve Reindex API
categories:
- blog
summary: Index isimleri olarak alias kullanarak index değişikliklerinde veya verileri taşımak zorunda kaldığınızda uygulama katmanınızı değiştirmeden verileriniz taşıyın.
---

Elasticsearch API arayüzü `index` isimleri üzerinden çalışmaktadır ve bir cluster içerisinde 
çoğu zaman bir çok `index`iniz bulunmaktadır. Index Alias API arayüzü ile `index` isimlerinize
takma isimler verebilirsiniz. Bunun önemini bir örnek üzerinden açıklamaya çalışayım. 

Bir kullanıcıların verilerini tuttuğunuz `index`iniz var ve yapısı aşağıdaki gibi:

```
GET users/user/_mapping
{
   "users": {
      "mappings": {
         "user": {
            "properties": {
               "id": {
                  "type": "string"
               },
               "name": {
                  "type": "string"
               },
               "tags": {
                  "type": "string",
                  "index": "not_analyzed"
               }
            }
         }
      }
   }
}
```

Bir takım verileriniz birikti ve sonradan farkettiniz ki `id` alanını integer yerine string olarak
kaydetmişsiniz. Hemen değiştirmek için bir mapping update sorgusu hazırladınız:

```
PUT users/user/_mapping
{
   "properties": {
        "id": {
            "type": "integer"
        }
   }
}
```

Ancak hali hazırda olan bir alanın veri tipini değiştirmeye çalıştığınızda Elasticsearch 
bundan pek hoşlanmayacaktır ve aşağıdaki gibi bir hata verecektir:

```
{
   "error": {
      "root_cause": [
         {
            "type": "illegal_argument_exception",
            "reason": "mapper [id] of different type, current_type [string], merged_type [integer]"
         }
      ],
      "type": "illegal_argument_exception",
      "reason": "mapper [id] of different type, current_type [string], merged_type [integer]"
   },
   "status": 400
}
```

Uygulamanız hali hazırda kullanılıyor olduğu için index'i silip tekrar oluşturamazsınız. Uygulama 
içerisinde de bir çok yerde index ismini kullandığınız için farklı bir isimle index oluşturmanız 
da sizin için zor olacak. Bu durumlarda `alias`lar hayat kurtarıcı olabiliyor. Hemen kısaca 
açıklayalım. 

İlk olarak yeni bir index oluşturup yeni yapımız ile `type`ı oluşturuyoruz. 

```
POST users_20160506

POST users_20160506/user/_mapping
{
   "properties": {
        "id": {
            "type": "integer"
        },
        "name": {
            "type": "string"
        },
        "tags": {
            "type": "string",
            "index": "not_analyzed"
        }
   }
}
```

Daha sonrasında verilerimizi yeni index'imize taşıyoruz. Bunun için Elasticsearch 2.3.1 ile gelen
yeni özelliği `_reindex` API arayüzü kullanabilirsiniz.

```
POST /_reindex
{
  "source": {
    "index": "users"
  },
  "dest": {
    "index": "users_20160506"
  }
}
```
Buradaki örneğimizde tabiki `id` alanımız integer tipine rahatça çevrilebilecek bir alan olduğu 
için rahatça bunu yapabildik. Burada eğer bir alan üzerinde ekstra işlem yaparak taşımak isteseydik 
nasıl olacaktı. Bunun için de script kullanabilirsiniz: 

```
POST /_reindex
{
  "source": {
    "index": "users"
  },
  "dest": {
    "index": "users_20160506"
  },
  "script":{
    "inline": "if (ctx._source.foo == 'bar') { ctx._source.id = parseInt(ctx._source.id) }"
  }
}
```
Şimdi verimizi taşıdık. Şimdi sıra geldi takma isim kısmına. Şimdi yeni verdiğimiz index için bir 
takma isim vereceğiz ve eski index'imizi sileceğiz ya da takma ismini kaldıracağız. Bizim
senaryomuzda `users` adında bir index'imiz vardı ve biz bu index'deki verileri yeni bir index'e 
taşıdık. Bu durumda eski index'imizi kaldırmamız ya da silmemiz gerekmekte. Eğer alias olarak
eski index'imizin adını vereceksek ve index'imizi silmeden bu işlemi yapmaya çalışırsak 
Elasticsearch alias oluşturma sırasında bize hata verecektir. Bu ismi kullanan bir `index` var 
diyecektir. Şimdi bizim adımlarımızı şöyle yapalım. Verilerimizi yeni index'imize taşıdığımıza 
göre eski index'imize ihtiyaç kalmadı. Bunun için o index'i silelim ve yeni oluşturduğumuz 
index'e `users` takma adını verelim.

```
DELETE users

POST _aliases
{
    "actions" : [
        { "add" : { "index" : "users_20160506", "alias" : "users" } }
    ]
}
```

Bu işlemleri tamamladıktan sonra halen `/users/user/_search` sorgusu yaptığınızda verilerinizi
görebilirsiniz. 

```
{
   "took": 9,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 4,
      "max_score": 1,
      "hits": [
         {
            "_index": "users_20160506",
            "_type": "user",
            "_id": "AVSB74LRHQRC9fPqMzsA",
            "_score": 1,
            "_source": {
               "tags": [
                  "php",
                  "python",
                  "html"
               ],
               "name": "Haydar"
            }
         },
         ...
      ]
   }
}
```

Peki olan bir takma ismi bir index'den diğerine taşımak için ne yapmalısını.z Bunun için 
tek bir komut çalıştırmanız yeterli. İki komut birden çalıştırmanız gerekmez. Bir örnek ile 
anlatmaya çalışayım. Gün geldi bu yazımızda oluşturduğumuz `users_20160506` index'ini
yine değiştirmeniz ve taşımanız gerekti. Verileriniz `users_20160720` index'ine taşıdınız ve 
şimdi `users` takma adınızı taşımak istiyorsunuz. İşiniz gayet basit: 

```
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "users_20160506", "alias" : "users" } },
        { "add" : { "index" : "users_20160720", "alias" : "users" } }
    ]
}
```

Sonuç olarak takma isimler (`_aliases`) ve 2.3.1 versiyonu ile gelen `_reindex` API arayüzü 
işlerinizi ve hayatınızı baya kolaylaştırabilir. Bu API arayüzlerini kullanarak verinizi uygulama 
katmanından ayırarak kolay yönetebilir ve rahatlıkla taşıyabilirsiniz. 