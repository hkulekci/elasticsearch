---
layout: post
title: Kapasite Planlaması
categories:
- blog
summary: Elasticsearch ile uğraşan herkesin sorduğu bir soru olan "Bu veri için kaç Shard'lık bir Index oluşturmam lazım?" sorusuna yanıt bulmaya çalışacağız.
---

Çoğu zaman kullanıcılar kaç shard kullanmalıyım. En doğru yaklaşım nedir diye düşünmketedirler. 1 shard çok az ya da 1000 shard çok fazla ise kaç shard kullanmam gerektiğini nerden bileceğim. Bu soru genellikle cevaplanması en imkansız sorulardan birisidir. Çünkü genellikle kişilerin ya da sistemlerin kendine has sorunları ve baş etmesi gereken kısımları olmaktadır. İşte bu yüzden bu sorunun cevabını etkileyecek çok fazla değişken mevcuttur: kullanıdığınız sunucu ya da donanım, dökümanınızın büyüklüğü ve karmaşıklığı, dökümanlarınızı nasıl analiz ettiğiniz ve index'e gönderme yönteminiz, çalıştırdığınız sorguların tipleri, yaptığınız `aggregation` çeşitliliğiniz, veriyi nasıl modellediğiniz, ve böyle bir çok konu vardır.

Neyseki, bunun için bir yöntem geliştirebiliriz:

 - Production'da kullanılacağınız konfigürasyona çok yakın olacak donanıma sahip bir tek sunucudan oluşan bir cluster(küme) oluşturalım.
 - Yine production'da kullanacağınızı düşündüğünüz index ile aynı `settings` ve `analyzers`a sahip bir index oluşturalım, ama sadece 1 primary shard'dan oluşsun ve hiç yansısı olmasın.
 - Bu index'i gerçek ya da gerçeğe çok yakın olabilecek dökümanlar ile dolduralım.
 - Bu idnex üzerinde gerçek sorgular ve gerçek `aggregations` sorgularını çalıştıralım.

Aslında, gerçek dünyadaki kullanımı neredeyse aynısını yapmak istemekteyiz ancak bu her zaman mümkün olmayabilir. Ancak bu örnek ile birlikte bu bir shard'lık index'imizi biraz zorlayalım (bol bol sorgular ve veriler göndererek). Hatta kırılma noktası da size bağlı: kimi kullanıcılar her bir cevap 50ms'den az olması lazım diyebilir; diğerleri 5s benim için uygundur diyebilir. Burada kırılma noktalarını da kendimiz belirleyip kendi sistemimize saldıralım. Burada aldığımı sonuçlardan sonra tekil bir shard için bir kapasite belirlemiş olduk.

Bir tekil shard için kapasiteyi belirledikten sonra, artık tüm index'iniz için  bilinmeyenlere ulaşmak daha kolay olacaktır. Index'inizde bulunan toplam verinizin boyutunu alın, veya yakın gelecekteki büyümeyi de hesaplayıp , ve bir shard için kapasitenize bölün. Sonuç, neredeyse sizin primary shard olarak ihtiyacınızı verecektir. Buradaki sonuçlar kesinlik ifade etmez. 1 shard için denediğinizde ağ gecikmeleri vs olmayacaktır. Ancak sorgularınızda birden fazla shard'da sonuç dönerken farklo sunuculardaki verilerinizde ağ gecikmeleri yaşanabilir.

> **Uyarılar**
> 
> Kapasite planlama sizin birinci adımınız olmaması gerekir.
> 
> Ilk olarak bakmanız gereken başka yerler vardır. Mesela, efektif çalışmayan sorgularınız olabilir mi?, RAM yetersiz olabilir mi?, Swap belleğe düştünüz mü?, ağdan dolayı bir gecikme yaşıyor olabilir misiniz? ...
> 
> Bir çok kullanıcı, basit bir wildcard sorgusunu silmek yerine garbage collector'u nasıl daha hızlandırırım veya thread sayılarını nasıl daha efektif ayarlarım diye uğraşmaktadır. 