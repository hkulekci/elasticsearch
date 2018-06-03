---
layout: post
title: JVM Ayarları (ES 6.2.x)
categories:
- blog
summary: JVM ayarları için ne yapmak gerekir ve nasıl değiştirilir değinmeye çalıştık ve bu ayarlardan bazılarını daha detaylıca açıklamaya çalıştık.
---

## 

JVM ayarları sıklıkla değiştirilmese de geliştirme ortamından canlı ortama geçiş sırasında daha performanslı bir cluster için bazı değişiklikler yapılması gerekir. ES'in eski sürümlerinde bu ayarları verirken ES'i çalıştırma komutuna parametre olarak veriyorduk:

```
bin/elasticsearch -Xms1g -Xmx1g
```

5.x ve Daha sonraki sürümlerde JVM ayarları için `jvm.options` dosyası ve `ES_JAVA_OPTS` çevresel değişkeni üzerinden yapabiliyor olduk. Bu yazıda ben kısaca `jvm.options` dosyasından bahsedeceğim. 

`jvm.options` dosyasını elasticsearch kurulumunun yapıldığı dizindeki `config` klasörü altında ya da Debian ya da RPM üzerinden kurulumlarda `/etc/elasticsearch` klasörü altında bulabilirsiniz. Bu dosya içerisinde bazı satırlar `#` karakteri ile başlar bir çok dilde de olduğu gibi bu satırlar yorum satırıdır. 

Bu dosya üzerinden JVM ayarlamaları yaparken Java'nın farklı sürümleri için farklı ayarlamalar yapabilirsiniz. Örneğin tüm java sürümleri için bir ayarlama yapmak istiyorsanız satırı `-` ile başlatmanız yeterliyken sadece JDK 8 sürümü için bir ayarlama yapmak istiyorsanız `8:` başlatmanız gerekir. Örneğin 9+ JDK sürümleri için bir ayarlama yapmak isterseniz bu durumda da satır başınız `9-:` ile başlamalıdır. Bunlar için örnekler vermek gerekirse, ES'in varsayılan `jvm.options` dosyasına kısa bir göz atmak yeterli olacaktır. 

```
....
# log4j 2
-Dlog4j.shutdownHookEnabled=false
-Dlog4j2.disable.jmx=true

....
## JDK 8 GC logging

8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
....

# JDK 9+ GC logging
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
....
```

Tabiki varsayılan dosya bu kadar ayarlama içermiyor ben sadece kısa kesitler halinde örnekleri buraya ekledim. Daha detaylı bakmak için şu adresi kullanabilirsiniz. 

[https://github.com/hkulekci/elastic-docker/blob/6.2.4/.docker/elasticsearch/config/jvm.options]()

#### JVM Heap Size

Java uygulamasının yanı ES'ini başlangıçta ayıracağın bellek miktarını ve maksimum kullanabileceği bellek miktarını belirler. Bunun için aşağıdaki parametreleri `jvm.options` dosyasına yazabiliriz ya da var ise değiştirebiliriz.

```
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms1g
-Xmx1g
```

Buradaki parametrelerin ne olduğunu daha iyi anlamak için Heap Bellek (heap memory) ne demek onu anlamak gerekir. Java uygulamalar bir takım verileri belleğe kaydetmek için iki tür yöntem kullanır. Birisi `stack` diğeri ise `heap`dir. `stack` türünde verilerin bellekte tutulup silinmesini uygulamanın kendisi değil işletim sistemi/alt katmanlar yönetir. `heap` türünde ise bellek kullanımını ve temizlenmesini uygulama yönetir. Java'da objeler `heap` bellekte tutulur ve `Garbage Collector` denen yapılar sayesinde `heap` bellek yönetimi sağlanır.

Burada sorulacak soru aslında çok fazla. Java'da farklı türlerde Garbage Collector'lar mevcut. Bunların seçimleri bazı durumlarda performans artışı sağlarken bazı durumlarda sağlamayabiliyor. Ya da GC nasıl çalışıyor? gibi gibi. Bu soruları cevaplarken yazının asıl amacından sapacağımız için şimdilik bu onuyu kaynaklar bölümündeki linkler ile devam ettirmeyi öneriyorum. Daha fazla güzel kaynak için yorumlardan bana yazabilirsiniz. 

#### Garbage Collector Ayarlamaları

Yukarıda da bahsettiğimiz gibi `heap` bellek alanının dolduğunda ya da belirli aralıklarla temizlenmesi için gerekli bir araç olan Garbage Collector için ayarlamalar yapabiliriz. Örneğin, bazı kaynaklarda çok önerilmese de, `+UseG1GC` ile GC türünüzü daha yeni bir GC olan G1GC ile değiştirebilirsiniz. Ancak ES bazı hatalardan dolayı daha G1GC'i desteklemiyor. [*](https://docs.datastax.com/en/cassandra/3.0/cassandra/operations/opsTuneJVM.html#opsTuneJVM__choose-gc) [*](https://discuss.elastic.co/t/g1gc-in-production-with-regard-to-consistency/47561/3) [*](https://discuss.elastic.co/t/elasticsearch-appears-to-ignore-xx-useg1gc-in-jvm-options/96862)

Diğer `CMSInitiatingOccupancyFraction` ayarı ile `heap` bellek doluluk oranına göre GC'nin çalışmasını sağlayan bir oran belirleyebilirsiniz. Java dünyasında ortalama %70-75 civarları ideal görülüyor. ES için varsayılan değer %75.

GC için varsayılan değerler aşağıdaki gibi:

```
## GC configuration
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
```

#### Heap Bellek Hata Ayarları

Heap bellek yönetimi sırasında bazı hatalar oluşabilir. Bu hataları yakalamak ve bir yerlere kaydetmen için aşağıdaki jvm ayarlarını kullanabilirsiniz. `HeapDumpPath` ile bir dosya belirleyerek bu dosyaya hataları yazdırabilirsiniz.

```
# generate a heap dump when an allocation from the Java heap fails
# heap dumps are created in the working directory of the JVM
-XX:+HeapDumpOnOutOfMemoryError

# specify an alternative path for heap dumps
# ensure the directory exists and has sufficient space
#-XX:HeapDumpPath=/heap/dump/path
```

Bunlar dışında daha bir sürü ayarlamalar mevcut. Yazının başında da paylaştığım linkten bu ayarlara ulaşabilirsiniz. Yazıyı hazırlarken okuduğum yazıları kaynaklar olarak paylaşıyorum. Umarım yararlı olur. 


**Genel Kaynaklar :**

 - https://www.elastic.co/guide/en/elasticsearch/reference/current/jvm-options.html
 - https://docs.oracle.com/cd/E22289_01/html/821-1274/configuring-the-default-jvm-and-java-arguments.html
 - https://www.gribblelab.org/CBootCamp/7_Memory_Stack_vs_Heap.html
 - https://www.elastic.co/blog/a-heap-of-trouble
 - http://blog.sokolenko.me/2014/11/javavm-options-production.html

**Garbage Collector Kaynakları :**

 - https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html
 - https://blog.takipi.com/garbage-collectors-serial-vs-parallel-vs-cms-vs-the-g1-and-whats-new-in-java-8/
 - https://www.cakesolutions.net/teamblogs/low-pause-gc-on-the-jvm
 - https://dzone.com/articles/java-garbage-collectors-when-will-g1gc-force-cms-out
 - https://dzone.com/articles/garbage-collectors-serial-vs-0
 - https://blog.novatec-gmbh.de/g1-action-better-cms/
 - https://stackoverflow.com/questions/16695874/why-does-the-jvm-full-gc-need-to-stop-the-world
 - https://www.journaldev.com/4098/java-heap-space-vs-stack-memory
 - http://blog.ragozin.info/2011/12/garbage-collection-in-hotspot-jvm.html
 - http://www.javainuse.com/java/permgen
 - http://blog.sokolenko.me/2014/11/javavm-options-production.html