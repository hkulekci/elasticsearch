
---
layout: post
title: Elasticsearch Index - Part 2
categories:
- blog
---

Elasticsearch arama motoru hakkında bir önceki yazımda kullanabileceğiniz 
araçlar, eklentiler ve kullanımları hakkında kısa bir önbilgi vermiştim. Bu
yazımda ise nasıl index oluşturacağız, mapping nedir ve nasıl oluşturabiliriz,
bir index'in ne gibi özellikleri vardır, gibi sorulara cevap vermeye çalışacağım.

Index daha önceki yazımda da bahsettiğim gibi verilerin tutulduğu 
veritabanlarıdır. Index'leri iki şekilde oluşturabilirsiniz. Birincisi otomatik 
oluşturma yöntemi diğeri ise manuel index oluşturma yöntemidir. Bu 
yöntemleri sırayla örneklemeye çalışayım.

Elasticsearch API arayüzünde `create` işlemleri için `POST` metodu kullanılır. 
Bir index oluşturmak için de `POST` metodunu kullanıyoruz. Aşağıdaki gibi
bir komut ile Sense arayüzünden `users` adında test bir index oluşturalım.

```
POST /users
```

Bu yöntem 1. yöntem olsun. Index oluşturmak için çeşitli yollar vardır. 
Örneğin siz veri oluşturmak için aşağıdaki sorguyu çalıştırdığınızda da bir
 `users` index'i oluşacaktır. 

```
POST /users/user/1
{
    "name":"Haydar"
}
```

Bu yönteme de 2. yöntem diyelim. Bu yöntem ile index oluşturulduğunda 
aynı zamanda bir kullanıcı da oluşacaktır ve sonuç olarak aşağıdaki gibi bir 
cevap dönecektir. 

```
{
   "_index": "users",
   "_type": "user",
   "_id": "1",
   "_version": 1,
   "created": true
}
```
Burada iki yöntemde de index oluşacaktır. Ancak ikinci yöntem diğerine göre 
biraz daha sakıncalı bir yöntemdir.  Bunun sakıncalarına daha ilerde 
değineceğiz. Şimdi oluşturduğumuz index'e bir göz atalım.  

```
GET /users
```

Bu komutu Sense arayüzünden çalıştırdığımda eğer index'i birinci yöntem ile 
oluşturursam `mappings` alanı boş gelecektir. Ancak eğer ikinci yöntem ile 
kullanılmış olsaydı aşağıdaki gibi olacaktı. Burada ikinci yöntemde aslında biz 
bir veri oluşturmaya çalışıyoruz. Eğer index yok ise ES o index'i otomatik 
oluşturuyor. Bu sırada veri oluşturma işinide yaptığımız için index'in ve 
mapping'i (yani veriyi tutma şeklinide) otomatik oluşturmuş oluyoruz. Mapping 
mapping verinin yapısını belirler. Eğer biz bunu kendimiz belirlemez ise ES 
otomatik yapacaktır. 

```
{
   "index-name": {
      "aliases": {},
      "mappings": {
          "user": {
            "properties": {
               "test": {
                  "type": "string"
               }
            }
         }
      },
      "settings": {
         "index": {
            "creation_date": "1447004861339",
            "uuid": "KzZW7zJyTPavy4n1TTeNkg",
            "number_of_replicas": "1",
            "number_of_shards": "5",
            "version": {
               "created": "1070399"
            }
         }
      },
      "warmers": {}
   }
}
```

Bu yanıttan anlaşılacağı üzere index'leri oluşturan şeyler `mappings`, 
`settings`, `warmers` ve `aliases`lardan oluşur. Bunların hepsinden bu yazımda
tabiki bahsedemeyeceğim. Ancak sadece kısaca ne işe yaradıklarından 
bahsedeceğim. 

`aliases` kısmında index'inize verdiğiniz takma isimler listelenmektedir. Bu 
isimlendirme yöntemi sizi uygulama katmanındaki index'e erişiminiz çok 
kolaylaştıracak bir özelliktir.  Bu konu hakkında ayrı bir yazıda özellikle 
değineceğim. Uygulamanın kesinti yaşamadan index değiştirebilmesi için
çok güzel bir özelliktir.

`mappings` kısmında ise index'in içerisinde saklanacak verinin yapısı 
tutulmaktadır. Ben bunu tabloların yapılarına benzetiyorum. Tabiki aklınızdan 
NoSQL veritabanında yapıya ne gerek var. Zaten JSON tutulmuyor mu diye 
düşünebilirsiniz. Ancak performans açısından bazı verilerin belirli bir yapıda 
kaydedilmesi daha hızlı sonuç dönebilmenize olanak sağlamaktadır. Özellikle
üzerinde arama yapacağınız alanların ell belirlenmesi çok daha iyi sonuçlar 
doğuracaktır. Arama yapmayacağınız alanları ise arama yapmayacağım diye 
belirtmeniz aramalarınız hızlandıracaktır. 

`settings` kısmında ise index ile ilgili olarak ayarlamalar tutulmaktadır. Örneğin 
bu index kaç shard olacaktır ve replica sayısı ne olacaktır. Bunlar dışında 
`analyzer` bilgileri de index'lerin `settings`leri içerisinde tutulmaktadır.  

Genel olarak index'lerin yapısını inceledik. Şimdi farklı şekillerde index'ler 
oluşturarak daha iyi pekiştirmeye çalışalım. Örneğin aşağıda 2 shard ve 
2 replica ile bir index oluşturalım. 

```
POST /index-name
{
    "settings": {
        "index": {
            "number_of_replicas": "2",
            "number_of_shards": "2"
        }
    }
}
```

Yukarıdaki sorguyu Sense ile çalıştırdığınızda index oluşacaktır ve bu index'deki 
veriler 2 shard ile bölünecektir. Aynı zamanda 2 makinede replica'ları oluşacaktır. 
Burada tabiki iki makineniz var ise. Eğer bu ayarları vermez iseniz Elasticsearch
varsayılan ayarları kullanacaktır ve 5 shard 1 replica ile bir index oluşturacaktır.

Varsayılan ayarlarınızı elasticsearch'ün config klasöründeki `yaml` dosyalarından
değiştirebilirsiniz. Elasticsearch dizininde `config` klasörü altında `elasticsearch.yml` 
ve `logging.yml` dosyaları bulunmaktadır. Adlarından da anlaşılacağı gibi 
loglama ayarları için `logging.yml` dosyasını Elasticsearch'in genel ayarları için 
`elasticsearch.yml` dosyasını kullanabilirsiniz. Ayarlar klasörü için yine ayrı bir 
yazımda değineceğim. Ayarların tümüne zaten bir yazıda değinmek çok yetersiz 
olur. Buradaki ayarlamalar ile çok daha performanslı bir yapı kurabilirsiniz ya da 
yanlış kullanım nedeniyle performansınızı tamamen kaybedebilirsiniz. 

Sağlıcakla.