---
layout: post
title: Elasticsearch'te Sayfalama Yapma (Scroll) - 2
categories:
- blog
summary: Bir önceki yazımda `from` ve `size` özelliğinden bahsetmiştim. Bu yazımda daha farklı bir sayfalama yöntemi olan Scroll özelliğinden bahsedeceğim.
---

Bir önceki yazımda `from` ve `size` özelliğinden bahsetmiştim. Bu yazımda daha farklı bir sayfalama yöntemi olan Scroll özelliğinden bahsedeceğim. Scroll özelliği ise bize daha farklı bir sunuş imkanı sağlıyor. Ben Scroll özelliğini Twitter'ın arayüzleri olarak düşünüyorum. Herkes biliyordur, aşağı doğru kaydırdıkça kaldığımız yerden veriler gelmektedir.

Örnek bir sorgu ile konuyu biraz inceleyelim. İlk olarak scroll kullanmak istediğimizi ve ne kadar süre bu sorguyu aklında tutması gerektiğini `?scroll=1m` sorgu parametresi ile gönderiyoruz.

```
POST /myindex/_search?scroll=1m
{
    "size": 100
} 
```

Bu sorguyu çalıştırdığınızda Elasticsearch size yanıt içerisinde bir `scroll_id` değeri ve sonuçları dönecektir.

```
{
  "_scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAcrFkIxQ3NhNWc4VDB1cHJZTENxUHkza3cAAAAAAAAHKBZCMUNzYTVnOFQwdXByWUxDcVB5M2t3AAAAAAAABywWQjFDc2E1ZzhUMHVwcllMQ3FQeTNrdwAAAAAAAAcqFkIxQ3NhNWc4VDB1cHJZTENxUHkza3cAAAAAAAAHKRZCMUNzYTVnOFQwdXByWUxDcVB5M2t3",
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1029761,
    "max_score": 1,
    "hits": [...]
  }
}
```

Bu değer ile birlikte sorgunuzuda aklında tutmaya başlayacaktır. Siz  `scroll_id` ile her istekte bulunduğunuzda aklındaki `offset` bilgisine göre size sorgunuza uyan sonraki kayıtları dönecektir. `scroll_id` ile sorgulama ise aşağıdaki şekildedir:

```
POST  /_search/scroll
{
    "scroll" : "1m", 
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAcrFkIxQ3NhNWc4VDB1cHJZTENxUHkza3cAAAAAAAAHKBZCMUNzYTVnOFQwdXByWUxDcVB5M2t3AAAAAAAABywWQjFDc2E1ZzhUMHVwcllMQ3FQeTNrdwAAAAAAAAcqFkIxQ3NhNWc4VDB1cHJZTENxUHkza3cAAAAAAAAHKRZCMUNzYTVnOFQwdXByWUxDcVB5M2t3" 
}
```

> Sorgu `index` ya da `type` adı içermez. Zaten kalında tuttuğu sorguda mevcuttur.

Bu sorgu sonucu olarak yine aynı `scroll_id` bize geri dönecektir. Siz her sorgu gönderdiğinizde `1m`lik süre sıfırlanacaktır ve tekrar başlayacaktır. Burada süre `1m`lik süre boyunca hiç gelmediğinizde ve bu `id` ile süreniz bittikten sonradan geldiğinizde Elasticsearch aşağıdaki gibi bir yanıt verecektir.

```
{
  "error": {
    "root_cause": [
      {
        "type": "search_context_missing_exception",
        "reason": "No search context found for id [1893]"
      },
     ....
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": -1,
        "index": null,
        "reason": {
          "type": "search_context_missing_exception",
          "reason": "No search context found for id [1893]"
        }
      },
      ...
    ],
    "caused_by": {
      "type": "search_context_missing_exception",
      "reason": "No search context found for id [1895]"
    }
  },
  "status": 404
}
```

Scroll özelliğinde dikkat ettiysek istediğimiz sayfaya bir anda atlayıp gidemiyoruz. Bu özelliğin olmaması kaynak kullanımı açısından bize kazanç sağlamaktadır. Burada hep aklıma şu söz gelir: "her seçim bir kaybediştir". Hız yönünden kazanırken ya da daha fazla veriye erişim imkanı kazanırken deneyim olarak daha farklı bir deneyimle karşı karşıya kalabiliyoruz. 

Bir sonraki yazımda Scroll ile benzer bir yapıda ancak `stateless` bir yapısı olan `search_after` özelliğinden bahsedeceğim.

### Kaynaklar 

 - [https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html]()
 - [https://www.elastic.co/guide/en/elasticsearch/guide/master/pagination.html]()