---
layout: post
title: Bazı Kavramlar (index, inverted index, shard, segment)
categories:
- blog
summary: Elasticsearch'de bazı kavramlar anlaması zor olabiliyor. `index` tam olarak ne demek, `inverted index` nedir? `shard`, `segment` nedir? gibi soruları kısaca açıklamalarını bulabileceğiniz bir döküman. 
---

"index" kelimesi Elasticsearch'de biraz istismar edilmiş. Anlamak için kısaca bakalım:

#### index

Elasticsearch'de bir `index` ilişkili bir veritabanındaki veritabanları gibidir. 
Verileri sakladığınız yerlerdir. Ama gerçekte, bu sadece senin uygulamanın gördüğüdür. 
Temelde, bir `index` bir veya daha fazla `shard`ı temsil eden mantıksal bir alan adıdır.

Ayrıca, "to index" (index'lemek) kavramıda veriyi Elasticsearch'e koymak/yüklemek 
anlamına gelmektedir. Verinizin geri istenebilecek şekilde saklandığını ve arama
yapılabileceğini ifade eder. 


#### inverted index

`inverted index` Lucene'in verileri aranabilir yapmak için kullandığı bir veri yapısıdır.
Bu işlemde dökümanın içerdiği verilerden `token`lar ve tekil terimler ortaya çıkarılır. 
Bu konu hakkında daha fazla bilgi almak için [buraya](https://en.wikipedia.org/wiki/Inverted_index) 
bakabilirsiniz.

#### shard

`shard` bir Lucene instance'ıdır. Kendi başına tamamen işlevsel bir arama motorudur. Bir 
`index` tek başına bir `shard`dan oluşabilir, ama genellikle `index` in büyüyebilmesi 
ve bir kaç `node` üzerinde dağıtılabilmesi için bir kaç `shard`dan oluşmaktadır.

`primary shard` bir döküman için ev sahipliği yapmaktadır. Bir `replica shard` ise 
`primary shard` bir kopyasıdır ve (1) `primary` bir şekilde hata verdiğinde ya da 
düştüğünde, (2) okuma çok arttığında, kullanılmaktadır.


#### segment

Her `shard` birden fazla `segment` içermektedir ve bu `segment`ler bir `inverted index`tir. 
Bir `shard` üzerindeki bir arama sırayla her bir `segment`'de aranacaktır ve sonra 
sonuçlar bu `shard` için son bir sonuçlar kümesinde toplanacaktır.

Siz bir dökümanı `index`lerken(indexing), Elasticsearch onları bellekte(memory) 
toplayacaktır (ve güvenlik için `transaction log`'da), sonra her saniye veya daha fazla 
zamanda bir bunu yapar, disk'e yeni bir `segment` oluşturur, ve aramaları `refresh` eder.

Bu veriyi yeni bir `segment` içerisinde aranabilir yapar, ama bu `segment` diske 
`fsync` edilmemiş haldedir. (Bende aynı soruyu kendime sordum. `fsync` nedir?) Veri halen 
kaybolma riskine sahiptir. 

Elasticsearch, sık sık `fsync` anlamına gelen `flush` işlemini yapar, (şimdi veri 
işlenmiştir.) ve artık gereksiz hale gelen `transaction log`u temizler. Çünkü biz artık
biliyoruz ki veri diske yazılmıştır. 

Ne kadar çok `segment` varsa aramalar o kadar uzun sürer. Yani Elasticsearch arkaplanda
çalışan birleştirici işlemler (merge process) ile benzer büyüklükteki bir kaç `segment`i
daha büyük bir `segment`e birleştirecektir. Yeni oluşan daha büyük `segment` diske 
yazıldıktan sonra eskileri silinecektir. Bu işlem bir çok aynı büyüklükte `segment`
olduğu sürece tekrar eder.

`Segment`ler değişmezdir. Bir döküman update edildiğinde, eski döküman silindi olarak 
işaretlenir, ve yeni bir döküman eklenir. Birleştirme işlemleri aynı zamanda silinen 
dökümanlarıda çıkarır.

#### Kaynakça 

 - [Basics about segments in elasticsearch](http://stackoverflow.com/a/15429578/721600)
 - [Basic Concepts](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html)
