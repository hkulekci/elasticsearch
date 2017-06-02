## Elasticsearch ile PDF, Metin veya Word Dosyaları Üzerinde Arama

Bir süre önce bir proje bünyesinde PDF, Metin veya Word dökümanları arama yapma ihtiyacı doğdu. Bu ihtiyaç üzerine bende konu hakkında biraz araştırmalar yaptım ve buradan da süreç içerisinde yaşadıklarımı adım adım paylaşmak istedim.

Proje sahibi ile ihtiyaçları ve neler yapılacağı hakkında biraz konuştuk ve ihtiyacı şu şekilde tanımladık.

> Bir kütüphanede farklı firmalara ait bir takım dökümanlar var ve bu dökümanlar PDF, metin veya word döküman formatında. Kullanıcılarımız bir arama kutusu üzerinden arama yaparak bu dökümanlar içerisinde aramalar yapıp dökümana ulaşmak istiyorlar. Başlangıç olarak döküman içerisindeki yerini önemsemeden, arama sayfasında arama içerisinde eşleşen bölümün bir kısmını da göstermek istiyorlar. (*)

İhtiyacı tam olarak anladıktan sonra çözüm için nasıl bir yol izleyebiliriz diye düşündüm ve konuyu araştırmaya başladım. Biraz araştırmanın ardından Apache Tika’ya ulaştım. Aramayı Elasticsearch üzerinde yapacaktım ve aklıma gelen ilk şey şu oldu. Apache Tika’yı kullanan bir API arayüzü hazırlarım dosyaları bana bu API üzerinden gönderirler, API dosyaların metin çıktılarını Elasticsearch’e kayıt eder ve sonuçta aramak için hazır hale gelir. Daha önce kullanmamıştım ve bu geliştirme hızı açısından benim için sıkıntılı olacaktı. Araştırmaya devam ettim.

Daha sonrasında bunu Elasticsearch eklentisi olarak yazan biri var mı diye bakıyordum ki zaten Elasticsearch içerisinde bu zaten var olduğunu gördüm.(**)

> [Ingest Attachment Processor Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/ingest-attachment.html)

Eklentiyi hemen kurdum:

```
bin/elasticsearch-plugin install ingest-attachment
```

Ve hızlıca denemeye başladım. İlk olarak aramanın yapılacağı dökümanı ve aklımdan geçen veriyi görsel hale getirmek için kendime örnek bir veri kümesi oluşturdum. (***)

```
{
  "filename": "sample.txt",
  "companyId": 1,
  "type": "txt",
  "content": "dosya-içeriği"
}
```

Bu veri örneğinde aramalarımı yaparken şöyle bir arama yapmak istiyorum. İlk olarak şirkete göre ve dosya tipine göre filtreleme yapmak istiyorum. Daha sonrasında bunlarla birlikte içerik ve dosya adına göre hem otomatik tamamlamalı ve hem de terim araması yapmak istiyorum. (!) Burası arama kısmındaki ihtiyaçları halleder nitelikte. Ancak veriyi kaydereken bir sıkıntı dikkat etmem gereken bir durum var. Elasticsearch JSON ile çalışıyor ve benim ikili formattaki olan dosyalarımı oraya atmak için bir yol bulmam lazım. Hemen yine dökümantasyona koştum. “Ingest attachment” eklentisinin kullanımına baktığımda dosya alanını Base64 ile şifreleyip gönderildiğini gördüm. Bu bana“content” olarak açtığım alana gelecek veriler için özel bir çalışma yapmam ve dosyayı o şekilde kaydetmem gerektiğini gösterdi dökümana göre ilerleyip bunun için aşağıdaki gibi bir “ingest pipeline” oluşturdum.

```
PUT _ingest/pipeline/attachment
{
  "description" : "Extract attachment information",
  "processors" : [
    {
      "attachment" : {
        "field" : "content"
      }
    }
  ]
}
```
Artık “content” alanı için gönderdiğimiz verilerimizi “attachment pipeline” üzerinden gönderirsek Base64 ile gönderdiğimiz dosyaların içeriklerini Elasticsearch eklentisi bizim için okunur bir metin formatına getirecek. Şimdi bunu deneyelim ve görelim:

```
# Index Oluşturdum
PUT files
# Ilk Verimi Kaydettim
PUT files/file/1?pipeline=attachment
{
  "filename": "sample.txt",
  "companyId": 1,
  "type": "txt",
  "content": "RWxhc3RpY3NlYXJjaCBoYWtrxLFuZGEgdMO8cmvDp2Ugw6dldmlyaWxlciB2ZSBhw6fEsWtsYW1hbMSxIGFubGF0xLFtbGFyCkdlbGnFn21lbGVyZGVuIGhhYmVyZGFyIG9sbWFrIGnDp2luIGJlbmkgdmUgcmVwbyd5dSB0YWtpcCBlZGluLg=="
}
```

Veriyi geri çektiğimde aşağıdaki gibi bir sonuç aldım.

```
# Istek
GET files/file/1
# Cevap
{
  "_index": "files",
  "_type": "file",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "companyId": 1,
    "filename": "sample.txt",
    "attachment": {
      "content_type": "text/plain; charset=UTF-8",
      "language": "lt",
      "content": "Elasticsearch hakkında türkçe çeviriler ve açıklamalı anlatımlar\nGelişmelerden haberdar olmak için beni ve repo'yu takip edin.",
      "content_length": 127
    },
    "type": "txt",
    "content": "RWxhc3RpY3NlYXJjaCBoYWtrxLFuZGEgdMO8cmvDp2Ugw6dldmlyaWxlciB2ZSBhw6fEsWtsYW1hbMSxIGFubGF0xLFtbGFyCkdlbGnFn21lbGVyZGVuIGhhYmVyZGFyIG9sbWFrIGnDp2luIGJlbmkgdmUgcmVwbyd5dSB0YWtpcCBlZGluLg=="
  }
}
```

Artık veriyi kaydetmiştim. Biraz yol kat ettim ancak daha sona ulaşamadım. Veri kayıt işlemini bitirdikten sonra şimdi sıra arama kısmına gelmişti. Bunun için yukarıda belirtmiştim nasıl bir arama için çalışma yapacağımı. Otomatik tamamlamalı ve terim aramaları yapacaktım. Ayrıca sonuç kümesinde eşleşen kelimeleri ön plana çıkaracaktım. İlk olarak verileri kaydettiğimde tüm veriler standard bir analyzer sürecinden geçti. Ancak bu analyzer benim istediğim sonuçları vermedi haliyle. Benim aramalarımı karşılayacak şekilde index için aşağıdaki gibi bir ayar oluşturdum.

```
PUT files
{
  "analysis": {
    "analyzer": {
      "keywordSearchAnalyzer": {
        "tokenizer": "whitespace",
        "filter": [
          "apostrophe",
          "turkishLowercaseFilter",
          "turkishStopwordsFilter",
          "asciiFilter"
        ]
      },
      "keywordSearchInputAnalyzer": {
        "tokenizer": "whitespace",
        "filter": [
          "apostrophe",
          "turkishLowercaseFilter",
          "turkishStopwordsFilter",
          "asciiFilter"
        ]
      },
      "autocompleteSearchInputAnalyzer": {
        "type": "custom",
        "tokenizer": "whitespace",
        "filter": [
          "apostrophe",
          "turkishLowercaseFilter",
          "asciiFilter"
        ]
      },
      "autocompleteSearchAnalyzer": {
        "type": "custom",
        "tokenizer": "whitespace",
        "filter": [
          "apostrophe",
          "turkishLowercaseFilter",
          "turkishStopwordsFilter",
          "asciiFilter",
          "autocompleteFilter",
          "unique"
        ]
      }
    },
    "filter": {
      "autocompleteFilter": {
        "type": "edge_ngram",
        "min_gram": 3,
        "max_gram": 20
      },
      "turkishStopwordsFilter": {
        "type": "stop",
        "stopwords": "_turkish_"
      },
      "turkishLowercaseFilter": {
        "type": "lowercase",
        "language": "turkish"
      },
      "asciiFilter": {
        "type": "asciifolding",
        "preserve_original": true
      }
    }
  }
}
```

Daha sonra bu bir “file” tipi için aşağıdaki gibi bir “mapping” ile verimin yapısını oluşturmuş oldum:

```
PUT files/file/_mapping
{
  "properties": {
    "filename": {
      "type": "text",
      "fields": {
          "autocomplete": {
            "type": "text",
            "analyzer": "autocompleteSearchAnalyzer",
            "search_analyzer": "autocompleteSearchInputAnalyzer"
          },
          "keyword": {
            "type": "text",
            "analyzer": "keywordSearchAnalyzer",
            "search_analyzer": "keywordSearchInputAnalyzer"
          }
      }
    },
    "companyId": {
      "type": "integer"
    },
    "fileType": {
      "type": "keyword"
    },
    "content": {
      "type": "text"
    },
    "attachment": {
      "properties": {
        "content": {
          "type": "text",
          "fields": {
            "autocomplete": {
              "type": "text",
              "term_vector": "with_positions_offsets",
              "store": true,
              "analyzer": "autocompleteSearchAnalyzer",
              "search_analyzer": "autocompleteSearchInputAnalyzer"
            },
            "keyword": {
              "type": "text",
              "term_vector": "with_positions_offsets",
              "store": true,
              "analyzer": "keywordSearchAnalyzer",
              "search_analyzer": "keywordSearchInputAnalyzer"
            }
          }
        }
      }
    }
  }
}
```

Şimdi bir arama yaparken eğer otomatik tamamlama yapmak istiyorsam “attachment.content.autocomplete” alanında eğer terim araması yapacaksam “attachment.content.keyword” alanında arama yapabilirim. Örnek bir arama çalıştıralım:

```
GET files/_search
{
  "query": {
    "match": {
      "attachment.content.keyword": "hakkinda"
    }
  }
}
```

Kaydetmiş olduğum veride “hakkında” kelimesi geçtiği için sonuç olarak bana döndü. Ancak bu aramada “hakkin” terimini arayınca sonuç vermedi. Otomatik tamamlama için bu aramayı da aşağıdaki gibi değiştirdim:

```
GET files/_search
{
  "query": {
    "match": {
      "attachment.content.autocomplete": "hak"
    }
  }
}
```

Bu değişiklik sonunda “hak” diye aradığımda “hak” ile terimleri içeren tüm dökümanlar dönmüş oldu. Artık arama işleminide tamamladım. Son olarak döküman içerisinde nerede geçtiğini gösterecektik. Yani metni vurgulama işlemi yapacaktık. Onunu için zaten mapping tarafında bir hazırlık yapmıştık.

>"term_vector": "with_positions_offsets",

Bu özellik ile birlikte artık her döküman içerisinden türetilen metinlerin her bir terimi için pozisyon bilgiside bir kenarda tutuluyor olacak. Arama sonucunda eşleşen terim `<em>` etiketleri arasında gelecek. Bu konuda daha fazla bilgi için şu blog yazısına bir göz atabilirsiniz. Sorguma bazı eklemeler yaparak sonuçlar içerisinde vurgulamayı da yaptım:

```
# Istek
GET files/_search
{
  "query": {
    "match": {
      "attachment.content.keyword": "hakkinda"
    }
  },
  "highlight" : {
    "fields" : {
      "attachment.content.keyword" : {}
    }
  }
}
# Cevap
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2947756,
    "hits": [
      {
        "_index": "files",
        "_type": "file",
        "_id": "2",
        "_score": 0.2947756,
        "_source": {
          "companyId": 1,
          "filename": "sample.txt",
          "attachment": {
            "content_type": "text/plain; charset=UTF-8",
            "language": "lt",
            "content": "Elasticsearch hakkında türkçe çeviriler ve açıklamalı anlatımlar\nGelişmelerden haberdar olmak için beni ve repo'yu takip edin.",
            "content_length": 127
          },
          "type": "txt",
          "content": "RWxhc3RpY3NlYXJjaCBoYWtrxLFuZGEgdMO8cmvDp2Ugw6dldmlyaWxlciB2ZSBhw6fEsWtsYW1hbMSxIGFubGF0xLFtbGFyCkdlbGnFn21lbGVyZGVuIGhhYmVyZGFyIG9sbWFrIGnDp2luIGJlbmkgdmUgcmVwbyd5dSB0YWtpcCBlZGluLg=="
        },
        "highlight": {
          "attachment.content.keyword": [
            "Elasticsearch <em>hakkında</em> türkçe çeviriler ve açıklamalı anlatımlar\nGelişmelerden haberdar olmak için beni"
          ]
        }
      }
    ]
  }
}
```

Gördüğünüz gibi “highlight” alanı altında eşleşen kelime `<em>` etiketleri arasında geldi. Burada küçük bir CSS eklemesi ile eşleşen kelimeyi vurgulayabilirsiniz ve aşağıdakine benzer bir görüntü elde edebilirsiniz.



### Son Söz

Bu yazıda PDF, metin ya da word dosyalarını nasıl Elasticsearch ile aranabilir hale getireceğimizi bir problemi çözerken gördük. Bu çözüm yöntemlerinden birisiydi. Bu yöntemin bazı artıları ve bazı da eksileri tabiki var. Hiç ekstra bir API ya da uygulamaya ihtiyaç duymadan bu işlemi yapabilmek bize artı sağlarken, veriniz içerisindeki metin alanlarının çok büyük olması performans açısından bazı sıkıntılar çıkaracak ve bunları çözmeniz gerekecek. Onun dışında Base64'e çevirdiğimiz dosyalarımız boştan yere %30–40 gibi bir oranlarda yer kaplayacak. Bunları ile ilgili nelere yapabiliriz nasıl çözüm üretebiliriz ilerleyen zamanlarda bir yazı ile anlatmaya çalışacağım.

### Duyuru

Elasticsearch ile ilgili kullanımı, veri yapısının oluşturulması ve verilerin aranması konuları hakkında daha detaylı bahsedeceğimiz bir eğitimim var. Katılmak isteyenler [http://datademi.com/index.php/tr/elastic-online-egitim/](http://datademi.com/index.php/tr/elastic-online-egitim/) şu adresten iletişime geçebilirler.

---

Yazı ilk olarak [şu adreste](https://medium.com/@kulekci/elasticsearch-ile-pdf-metin-veya-word-dosyaları-üzerinde-arama-a37f66435bf6) yayınlanmıştır. 