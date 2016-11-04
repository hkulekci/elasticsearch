---
layout: post
title: Arama Sonucunda Metni Vurgulamak
categories:
- blog
summary: Arama sonuçlarınızda aramalarda eşleşen sonuçları vurgulamak için Elasticsearch hali hazırda bir özellik sunmaktadır. Bu yazıda bu özelliğin nasıl çalıştığını ve en efektif yöntem nedir öğreneceğiz.
---

Bir ya da birden fazla alanda sonuçlardaki metinleri vurgulamak için kullanılan özelliktir. Lucene `highlighter`, `fast-vector-highlighter` veya `postings-highlighter` uyarlamalarını kullanmaktadır. Aşağıda bir arama örneğini bulabilirsiniz: 

```
{
    "query" : {...},
    "highlight" : {
        "fields" : {
            "content" : {}
        }
    }
}
```

Yukarıdaki örnekte, `content` alanı her bir arama için vurgulanacak alandır. `highlight` parametresi içerisinde çağrılan herhangi başka bir alan da olabilir. Alan isimleri joker semboller ile de belirtilebilir. Örneğin `comment_*` ile `comment_` ile başlayan alanlar seçilmiş olacaktır. 

### Plain Highlighter

Varsayılan seçenek olarak `plain` vurgulama türü kullanılmaktadır ve Lucene Highlighter kullanılır. Terimlerin yerlerini saptamak ve biraz yavaşlatacaktır. 

### Posting Highlighter

Eğer mapping oluşturulurken arama yapmak istediğiniz alanların `index_options` parametresine `offsets` değerini verdiğinizde varsayılanın yerine bu tip vurgulama yöntemi kullanılacaktır. Daha hızlıdır çünkü metni tekrar analiz etmeniz gerekmez ve büyük dökümanlar için daha performanslıdır. `term_vectors`'a göre daha az alan(disk alanı) harcar. doğal dillerde gerçekten iyidir ancak html içeren metinlerde o kadar iyi değildir.

Mapping ayarlarını yaparken aşağıdaki gibi bir konfigürasyon yapabilirsiniz:

```
{
    "type_name" : {
        "content" : {"index_options" : "offsets"}
    }
}
```

### Fast Vector Highlighter

Eğer mapping oluşuturulurken arama yapmak istediğiniz alanların `term_vector` parametresine `with_positions_offsets` verdiğinizde varsayılanın yerine bu Fast Vector vurgulama yöntemi kullanılmaktadır. Özellikle büyük (> 1MB) alanlar için daha hızlıdır. Index boyutunu diğerlerine göre daha fazla büyütecektir. `matched_fields` özelliği ile birden fazla alandaki eşleşen sonuçları birleştirebilmektedir. Aşağıda bir `content` alanı için Fast Vector vurgulama yöntemini uygulayalım:

```
{
    "type_name" : {
        "content" : {"term_vector" : "with_positions_offsets"}
    }
}
```

### Vurgulama Tipini Seçmek

Sorgulama yaparken vurgulama yapmak istediğiniz alan için vurgu türünü seçebilirsiniz. Örneğin:

```
{
    "query" : {...},
    "highlight" : {
        "fields" : {
            "content" : {"type" : "plain"}
        }
    }
}
```

### Vurgulama için Etiket Seçimi

Vurgulamak istediğiniz sonuçları belli bir html etiketleri arasında göstermek istiyorsak bunun için `pre_tags` and `post_tags` parametrelerini kullanabiliriz. Bunlar için varsayılan değerler `<em>` ve `</em>`dir. Farklı bir etiket vermenin örneği aşağıdaki gibidir:

```
{
    "query" : {...},
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "_all" : {}
        }
    }
}
```

### Örnek Sorgu ve Sonuç

Örnek Sorgu aşağıdaki gibidir ve vurgulama türü olarak `fvh` kullanılmıştır. 

```
GET test/type/_search
{
   "query": {
      "match": {
         "text": "şarkı"
      }
   },
    "highlight": {
        "fields" : {
        "pre_tags":["<em>"],
        "post_tags":["<\/em>"],
            "text" : {}
        }
    }
}
```

Sorgu sonucu olarak aşağıdaki gibi bir cevap alınacaktır:

```
{
   "took": 47,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 1,
      "max_score": 0.11506981,
      "hits": [
         {
            "_index": "test",
            "_type": "type",
            "_id": "2",
            "_score": 0.11506981,
            "_source": {
               "id": 2,
               "text": "Güzel söyleri olan şarkılar gerçektende beni derinden etkiler"
            },
            "highlight": {
               "text": [
                  "Güzel söyleri olan <em>şarkılar</em> gerçektende beni derinden etkiler"
               ]
            }
         }
      ]
   }
}
```