---
layout: post
title: Elasticsearch 5.0 ile gelen Değişiklikler - Eklentiler
categories:
- blog
summary: Elastic 5.0 ile birlikte birçok değişikliğe gitti. Örneğin özellikle daha önceden önerdiğim Head ve Kopf eklentilerinin 5.x sürümlerinde eklenti olarak yüklenemiyor. Bu yazıda bu gibi eklentiler üzerinde ne gibi değişiklikler yaşandı bahsetmeye çalışacağım. 
---


Elastic 5 sürümü ile birlikte bazı servislerini farklı ürünler olarak elasticsearch 
bünyesinden çıkardı. Bununla birlikte eklentilerde de biraz değişiklikler oldu. Bu 
yazımda buna biraz değineceğim. 

İlk olarak bir eklenti yüklerken ya da silerken kullandığımız `bin/plugin` komutu
isim değiştirdi. yeni ismi şu şekilde `bin/elasticsearch-plugin`. `analysis-icu`
eklentisini kurarken aşağıdaki gibi bir komut çalıştırmamız gerekiyor.

```
bin/elasticsearch-plugin install analysis-icu
```

Site arayüzü olan eklentiler elasticsearch eklentisi olamayacaklar artık. Bu da 
belkide en çok kullandığımız [Head](https://github.com/mobz/elasticsearch-head) 
ve [Kopf](https://github.com/lmenezes/elasticsearch-kopf) eklentilerinin olmayacağı 
anlamına geliyor. Tabiki de tamamen hayatımızdan silinmiyorlar. Bunları kullanabileceğiniz 
çeşitli yöntemler mevcut. Head eklentisi kendi başına bir 
[node server](https://github.com/mobz/elasticsearch-head#running-as-a-plugin-of-elasticsearch) 
üzerinde çalışabiliyor mesela. 

Bunlar genel yapı ile ilgili iken bazı eklentiler direk Elasticsearch içerisine 
alındı. Örneğin `Delete-By-Query` eklentisi [Delete By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/5.1/docs-delete-by-query.html) olarak içeri alındı. Bunun dışında Mapper Attachements 
eklentisi [`ingest-attachment`](https://www.elastic.co/guide/en/elasticsearch/plugins/5.1/ingest-attachment.html)
olarak değişti. Tabi buradan şunuda anlamamız gerekiyor. Daha öncede bahsetmiştik 
IngestNode diye de bir kavram oluştu. Bu kavram ile birlikte ES, dışardan gelen verileri 
bir süzgeçten geçirmemize de olanak sağlıyor. Bunu ben Logstash'in input bacağındaki 
filtrelere benzetiyorum.

Eklentiler dışında bir çok konuda değişikler mevcut. Hatta bu değişikleri daha 
kolay üstesinden gelmek için ES ekibi 
[elasticsearch-migration eklentisi](https://github.com/elastic/elasticsearch-migration/blob/2.x/README.asciidoc) 
çıkardılar. Bu eklenti, 5.x sürümlerine geçiş için neleri değiştirmeniz gerektiğiyle 
ilgili bir liste sunuyor. 
