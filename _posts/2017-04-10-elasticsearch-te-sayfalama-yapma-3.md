---
layout: post
title: Elasticsearch'te Sayfalama Yapma (`start_after`) - 3
categories:
- blog
summary: Bir önceki yazımda `from`,`size` ve Scroll özelliklerinden bahsetmiştim. Bu yazımda Scroll özelliği ile benzer ancak bir durum bilgisi barındırmayan daha farklı bir sayfalama yöntemi olan `start_after` özelliğinden bahsedeceğim.
---

Scroll özelliği ile sorgularımızın sonuçlarını rahat bir şekilde sayfalama yapabiliyoruz. Ancak scroll özelliğinde sorgumuz `context` içerisinde tutulduğu için sayfalama sırasında sorgumuzu değiştiremiyoruz. Bu durum bizi gerçek zamanlı uygulamalarda sıkıntılı durumlara sokacaktır. Her seferinde sorgularınızı tekrar tekrar bir `context`e eklemeniz gerekecektir. 

"Search After" özelliği ile bu soruna da bir nebze çözüm bulabiliriz. Bu özellik ile canlı bir işaretçi oluşturabiliriz. İlk olarak örnek verimizdeki değerlere hızlıca bir bakalım:

```
Offset   | ID
262444821  | 10093220
262444572  | 10093219
262444331  | 10093218
262444090  | 10093216
```

Şimdi bu veriseti üzerinde bir sorgu ile sayfalama için nasıl bir yol izleyeceğimize bir göz atalım.

```
GET myindex/_search
{
  "from": 0,
  "size": 2,
  "sort": [
    {
      "offset": {
        "order": "desc"
      }
    }
  ],
  "_source": ["ID", "offset"]
}
```

Yukarıdaki bizim için ilk sorgumuz. 0'dan başlayıp 2 tane veri çekiyoruz. Şimdi bu örneğimizde tabiki basit gideceğiz. İlerleyen aşamada daha büyük sayılar ile tekrar örneklendirebiliriz. Farkettiysek sorgumuzda sıralama mevcut. Diğer örneklerde bir sıralama yoktu. "Search After" özelliği için bir sıralama yapmamız **zorunludur**. Çünkü sıralama yaptığımız alana göre bir sonraki sayfada çekeceğimiz veriyi seçeceğiz. Burada bize dönen veriye hızlıca bir göz atalım. 

```
{
  "took": 26,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1029761,
    "max_score": null,
    "hits": [
      {
        "_index": "myindex",
        "_type": "mytype",
        "_id": "AVreenrAIMPgDpuvzEzN",
        "_score": null,
        "_source": {
          "offset": 262444821,
          "ID": "10093220"
        },
        "sort": [
          262444821
        ]
      },
      {
        "_index": "myindex",
        "_type": "mytype",
        "_id": "AVreenrAIMPgDpuvzEzM",
        "_score": null,
        "_source": {
          "offset": 262444572,
          "ID": "10093219"
        },
        "sort": [
          262444572
        ]
      }
    ]
  }
}
```

Şimdi Bir sonraki sayfayı çekelim:

```
GET myindex/_search
{
  "size": 2,
  "search_after": [262444572],
  "sort": [
    {
      "offset": {
        "order": "desc"
      }
    }
  ],
  "_source": ["ID", "offset"]
}
```

Bu sorgu sonucunda dönen ID değerlerinin `10093218` ve `10093216` olduğunu görebilirsiniz. Yukarıdaki tablomuzda da zaten sırasıyla bunların gelmesi gerektiğini biliyorduk. Dikkat ettiysek ikinci sorgumuzda `"search_after": [262444572]` parametresini kullanarak aslında sayfalamayı yapmış olduk. 

```
{
  "took": 18,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1029761,
    "max_score": null,
    "hits": [
      {
        "_index": "myindex",
        "_type": "mytype",
        "_id": "AVreenrAIMPgDpuvzEzL",
        "_score": null,
        "_source": {
          "offset": 262444331,
          "ID": "10093218"
        },
        "sort": [
          262444331
        ]
      },
      {
        "_index": "myindex",
        "_type": "mytype",
        "_id": "AVreenrAIMPgDpuvzEzK",
        "_score": null,
        "_source": {
          "offset": 262444090,
          "ID": "10093216"
        },
        "sort": [
          262444090
        ]
      }
    ]
  }
}
```

Scroll özelliğinden farklı olarak `search_after` değerini sorgumuzu değiştirerek ilerletebiliyor olmamızdır. Scroll özelliğinde ise sorguyu bir kez gönderdikten sonra `scroll_id` ile ilerleyebiliyorduk. Buradaki sorgudan da göreceğimiz üzere rasgele erişimler için bu seçenekte çözüm değil. Ancak `from & size` özelliğini kullanarak erişemediğimiz verilerimize bu özellik sayesinde erişebileceğiz.

### Kaynaklar 

 - [https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-search-after.html]()
 - [https://www.elastic.co/guide/en/elasticsearch/guide/master/pagination.html]()