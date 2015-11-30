---
layout: post
title: Elasticsearch ve Geliştirme Araçları - Part 1
categories:
- blog
---

Elasticsearch'e yeni başlayanlar için hızlı ve rahat bir başlangıç yapmalarını 
sağlayacak araçlardan biraz bahsetmek istiyorum ve bu arada da kısaca 
Elasticsearch (ES kısaltması ile devam edeceğim) hakkında bilgi vermeye 
çalışacağım.

[Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch) wikipedia'da 
bahsedildiği üzere [Lucene](https://lucene.apache.org/core/) tabanlı bir arama 
sunucusudur. Genel bilinmesi gerekenler:

 - Full-text arama yapabilmenizi sağlar
 - Java ile geliştirilmiştir.
 - Apache Lisansı ile yayınlanmıştır.
 - Apache Solr'dan sonra gelen ikinci en populer arama motorudur. 
    [1](http://db-engines.com/en/ranking/search+engine)

Elasticsearch kurulumu ve çalıştırılması hakkında 
[buradan](https://www.elastic.co/downloads/elasticsearch) bilgi alabilirsiniz. 
Ben genel olarak kurulumdan bahsetmeyeceğim. Elimden geldiğince ES'i daha 
hızlı nasıl öğrenirsiniz ve çalışma ortamınızı nasıl hızlıca kurabilirsiniz
bundan bahsedeceğim.

Elasticsearch'de verilerin tutulduğu [index](https://www.elastic.co/blog/what-is-an-elasticsearch-index)'ler 
vardır. Bunları ilişkili bir veritabanındaki `database`ler gibi düşünebilirsiniz. 
Indexler içerisinde farklı tipteki verileri tutabilmeniz için `type` kavramı 
vardır bunu da tablolar gibi düşünebilirsiniz. Aralarındaki ilişkiyi daha rahat 
anlamak için aşağıdaki açıklamaya bakabilirsiniz. 

```
MySQL => Databases => Tables => Columns/Rows
Elasticsearch => Indices => Types => Documents with Properties
```

Şimdi ES'i [şu](https://www.elastic.co/downloads/elasticsearch) adresteki 
açıklamalar ışığında kuralım ve çalıştıralım. Burada ES'in default ayarlarını 
kullanabilirsiniz. Normalde production sunucusuna kurulum yapmıyorsanız default 
ayarlarda kullanmanız bir sakınca yaratmaz. İlerleyen yazılarımızda ES'i daha verimli
nasıl kullanırsınız ve yüksek erişilebilir hale nasıl getirirsiniz ya da ileri seviye 
ayarlamaları konfigürasyonları nelerdir bunlardan bahsedeceğiz. Şimdi, ilk olarak 
çalışma ortamını ayarlamak için bir iki eklenti ve bir de Chrome Plugin'i 
kuracağız. 

Eklentilerden başlayacak olursak, [head](https://github.com/mobz/elasticsearch-head) 
eklentisi Elasticsearch'ün API arayüzüne rahatça kullanabilmenizi sağlar. Bu 
eklenti ile index'leri type'ları rahatça görebilirsiniz. MySQL'in PhpMyAdmin'i
ile aynı görevi görüyor diyebiliriz. Bu eklenti ile index'lerinizi rahatça 
yönetebilirsiniz. 

İkinci eklentimiz olan [inquisitor](https://github.com/polyfractal/elasticsearch-inquisitor)
eklentisi ile sorgularınızı debug edebilir ve anlamaya çalışabilirsiniz. Bu 
eklenti ile oluşturmuş olduğunuz analyzer'larınızı kolayca debug edebilirsiniz
ve nasıl çalıştığını görebilirsiniz. Burada doğal olarak analyzer ne sorusu 
geliyordur aklınıza. [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/analysis-analyzers.html)'lar 
[Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/analysis-tokenizers.html) 
ve [TokenFilter](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/analysis-tokenfilters.html)'lardan 
oluşurlar. Tokenizer kavramını kısaca anlatacak olursak, verilen bir string'i daha 
küçük parçalara ayırma işlemini yaparlar. TokenFilter'lar ise bu parçaları bir 
filtreden geçirirler. Böylelikle Analyzer'lar ile ES'de tuttuğunuz verileriniz 
bir takım işlemlerden geçirerek arama yapılabilir bir halde hazırda bekletiyor 
olacaksınız. Bu konuya daha ilerde ayrı bir yazıda bahsetmeyi düşünüyorum. Şimdilik 
böyle bir şeyin olduğunu ve ileride kullanacağınızı bilmeniz yeterli.

Bunlar dışında bir çok eklenti mevcut. Hepsinden bahsedemeyeceğim ama zamanı 
geldikçe eklentiler hakkında kısaca bilgi vermeye çalışacağım. 

Şimdi development ortamımızı şenlendirecek Chrome Eklentimize. 
[Sense](https://www.elastic.co/blog/found-sense-a-cool-json-aware-interface-to-elasticsearch)
eklentisi ile ES sorgularını hızlı ve rahat bir şekilde çalıştırabilirsiniz. 
Aşağıda bir dizi komut yazacağım ve Sense eklentisi ile hızlıca deneyebileceksiniz.
Sense ile sorgu çalıştırmak için aşağıdaki formatta bir sorgu yapmamız gerekiyor.

```
METHOD /es/paths
{
    "JSON" : "DATA"
}
```

Bu sorgu formatında METHOD isteğin metodunu belirlemektedir. GET, POST, DELETE, ... 
gibi methodlar ile ES'e sorgu atabileceğiz. `/es/paths` kısmı ise ES'de sorgu 
yapacağınız index, type'ı belirlemek için kullanılan bir URL. Geri kalan kısmı ise 
veri kısmı. Bundan sonraki yazılarımızda sorguları bu formatta yazacağımız için
çabuk ısınacağınıza inanıyorum.

Aşağıda Sense ile çalıştırabileceğiniz bir kaç komut dizisi oluşturdum. Bunlar 
üzerinden giderek kısaca başlangıç yapabilirsiniz. 

Yeni bir index oluşturmak.

```
POST /test-index
```

Yeni bir döküman oluşturmak.

```
POST /test-index/test/1
{
    "id": 1,
    "name": "haydar külekci"
}
```

Yeni bir döküman oluşturmak

```
POST /test-index/test/2
{
    "id": 1,
    "name": "Test Name"
}
```

Type Mapping Bilgilerini Getirmek

```
GET /test-index/test/_mapping
```

Oluşturulmuş bir dökümanı getirmek

```
GET /test-index/test/1
```

Arama yapmak:

```
GET /test-index/test/_search
{
    "query": {
        "match_all": {}
    }
}
```



Bağlantılar 

 - [https://www.elastic.co/blog/what-is-an-elasticsearch-index](https://www.elastic.co/blog/what-is-an-elasticsearch-index)
 - [https://lucene.apache.org/core/](https://lucene.apache.org/core/)
 - [https://en.wikipedia.org/wiki/Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch)

Diğer Bazı Eklentiler 

 - [ICU Analysis plugin for Elasticsearch](https://github.com/elastic/elasticsearch-analysis-icu)
 - [ElasticSearch analysis plugin providing Turkish stemming functionality](https://github.com/skroutz/elasticsearch-analysis-turkishstemmer/)
 - [Web admin interface for elasticsearch](https://github.com/lmenezes/elasticsearch-kopf)
