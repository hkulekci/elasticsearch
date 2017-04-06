---
layout: post
title: Elasticsearch'te Sayfalama Yapma (From ve Size) - 1
categories:
- blog
summary: Elasticsearch üzerindeki verimiz ile sayfalama nasıl yaparız ne gibi sorunlar bizi bekliyor bir göz atacağız. Örnekler ile bu konuyu açıklamaya çalışacağız.
---

Elasticsearch üzerinde hızlıca arama yapabileceğimizden, veri yapılarından, veri türlerinden, index düzenleme ve takma isimlerinden bolca bahsettik. Bu yazıda çektiğimiz verileri kullanıcıya sayfa sayfa kullanıcıya nasıl ulaştırırız, bunun için nasıl yöntemler var, ondan bahsetmeye çalışacağım.

Verileri sorgulamak için çok fazlaca sorgu çeşitleri mevcut. Bu sorgular ve çeşitleri bu yazının kapsamında değil. Ama siz yazı içerisindeki sorgularda `query` özelliğini rahatça ekleyebilirsiniz. Ekstra özel durumlarda da zaten belirtiyor olacağım.

### `from` ve `size`

İlk olarak en basit yöntem olan ve SQL'de de kullandığımız `from`, `size` yöntemi ile başlayalım. Elasticsearch bize verilerimiz üzerinde dolaşabilmek adına `_search` API üzerinde `from` ve `size` alanlarını sunuyor. Bu alanlarla birlikte bir ofset bir de boyut belirterek sayfalama işlemini kolayca yapabiliyoruz. Aşağıdaki iki sorgudan birincisinde 1. sayfadaki verilerimizi alırken 
ikincisinde ise 2. sayfadaki verileri aldık.

```
GET myindex/_search
{
  "from": 0,
  "size": 10
}

GET myindex/_search
{
  "from": 10,
  "size": 10
}
```

Aslında bu açıdan baktığımızda gayet kolay gibi. Ancak veriniz eğer gerçekten de büyük ise karşınıza çıkacak bir Elasticsearch'te de bazı sınırlar mevcut. Bu sınırlamayı en kısa şöyle ifade edebiliriz => `from + size <= index.max_result_window`. Örnek bir sorgu yapalım. Diyelim ki içerisinde 1M kayıt olan bir index'iniz mevcut. Aşağıdaki gibi bir sorgu rahatça karşılaşabileceğiniz bir sorgu:

```
GET myindex/_search
{
  "from": 20000,
  "size": 20,
  "_source": ["ID", "offset"]
}
```

Ancak alacağınız cevap sizi çok üzecektir. 

```
{
  "error": {
    "root_cause": [
      {
        "type": "query_phase_execution_exception",
        "reason": "Result window is too large, from + size must be less than or equal to: [10000] but was [20020]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level setting."
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "myindex",
        "node": "B1Csa5g8T0uprYLCqPy3kw",
        "reason": {
          "type": "query_phase_execution_exception",
          "reason": "Result window is too large, from + size must be less than or equal to: [10000] but was [20020]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level setting."
        }
      }
    ],
    "caused_by": {
      "type": "query_phase_execution_exception",
      "reason": "Result window is too large, from + size must be less than or equal to: [10000] but was [20020]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level setting."
    }
  },
  "status": 500
}
```

Yani size `index.max_result_window` değerinden daha büyük sıradaki değerlere hiç ulaşamayacaksınız. `index.max_result_window` özelliği dinamik index ayarı olarak geçmektedir. Bu özelliği aşağıdaki gibi bir istek ile değiştirebilirsiniz.

```
PUT myindex/_settings
{
  "index": {
    "max_result_window": 10000
  }
}
```

Ancak, şunuda düşünmeliyiz. Bu değişikliğiniz size `from + size` değeriyle orantılı, bellek kullanımı olarak dönecektir. Burada karar verirken dikkat etmek gerekmektedir.

> [`index.max_result_window`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#dynamic-index-settings) özelliği varsayılan olarak 10000'dir. 

Bir sonraki yazımda bu konuya bir çözüm olarak Elasticsearch'ün sunduğu diğer bir özellik olan "Scroll" özelliğinden bahsedeceğim.

### Kaynaklar 

 - [https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-from-size.html]() 
 - [https://www.elastic.co/guide/en/elasticsearch/guide/master/pagination.html]()