---
layout: post
title: Eşzamanlılık Kontrolü
categories:
- blog
summary: Elasticsearch dağıtık bir yapıya sahiptir ve bu dağıtık yapıda eşzamanlı işlemlerimizde verilerimizi kaybetmeden nasıl sürümleriz ve sürümleri yönetiriz. 
---

Eğer Dağıtık çalışan bir uygulamanız varsa ve uygulamanızda dökümanları 
güncelleyen silen ya da oluşturan bir mekanizmanızda varsa ki vardır; Genellikle 
dökümanlar üzerindeki işlemlerde çakışmalar olur ve bu çakışmalar yüzünden veri 
kayıpları meydana gelir. Örnek olarak aşağıdaki grafige bir göz atabilirsiniz. 

![Stock Count Example](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_0301.png)

Örnekte görüldüğü üzere `web-1` ve `web-2` paralel işleyen iki web sunucusu 
ve aynı döküman üzerinden işlem yapıyorlar ve bu dökümandaki `stock_count`
alanının ilk değeri 100 iken ikiside aynı anda dökümanı çekiyor ve sonra
dökümanın değerini güncelleyip tekrar yazıyorlar. Sonuç itibariyle 
dökümanın son değeri 98 olması gerekirken 99 olarak kalıyor. Veritabanı
dünyasında buna sıklıkla rastlanır ve 2 genel yaklaşım vardır. 

 - Pessimistic concurrency control (Kötümser eşzamanlılık kontrolü) 
 İlişkili veritabanı sistemlerinde kullanılan satır ya da tablo bazında 
 erişimi engelleyerek veri kaybını önleme yönetmidir. 
 - Optimistic concurrency control (İyimser eşzamanlılık kontrolü)
 Elasticsearch tarafından kullanılan yöntemdir ve işlemi engellemez ancak
 güncelleme sırasında hata fırlatır ve çakışmayı çözme kararını uygulama 
 bırakır. 

Elasticsearch dağıtıktır. Döküman oluşturulduğunda, güncellendiğinde ya da 
silindiğinde dökümanın yeni sürümü diğer `node`lara ya da yansılar ile 
paylaşılır. Elasticsearch aynı zamanda asenkron ve eşzamanlı çalışır, 
bunun anlamı replikasyon işlemi isteklerini paralelde göndermesidir. 
Elasticsearch eski sürüm bir dökümanın yeni sürüm bir döküman üzerine 
yazılmaması için bir yola ihtiyaç duyarmaktadır. 

Daha öncede bahsettiğimiz üzere döküman için çekildiğinde ya da silindiğinde 
her bir dökümana ait bir `_version` alanında bir numarası bulunmaktadır. Bu 
numara döküman her değiştiğinde artmaktadır. Elasticsearch bu numarayı 
değişiklikleri düzgün sırayla onaylamak için kullanmaktadır. Eğer eski 
sürüm yeni bir sürümden sonra ulaşırsa sessizce göz ardı eder.

Biz `_version` numarasını kullanarak veri kaybı yaşamadan değişiklikleri 
uygulama içerisinde uygulayabiliriz. Değiştirmeyi düşündüğümüz döküman için
özel bir sürüm numarası vererek bunu yapabiliriz. Eğer bu sürüm şimdi sürüm
değil ise isteğimiz hata alır.

Şimdi bir blog yazısı oluşturalım:

```
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```

Cevap gövdesi bize yeni oluşturulan döküman için bir sürüm dönecektir. Şimdi
bu dökümanı düzenleyeceğimizi hayal edin: Bu veriyi bir form arayüzüne yükledik
ve değişikliklerimizi yaptık ve yeni bir sürüm kaydettik.

Öncelikle dökümanı çekelim.

```
GET /website/blog/1
```
Cevap gözdesi aynı sürüm numarasına sahip 1:

```
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```

Şimdi dökümanın değişikliklerini kaydetmeyi denerken özel bir sürüm numarası
gönderirsek:

```
PUT /website/blog/1?version=1 
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```

Şu durumda biz sadece bizim `_version` numaramız 1 olduğu durumda güncellemenin
doğru bir şekilde sonuçlanması gerektiğini belirtmiş oluruz. Bu istek başarıya 
ulaşacaktır ve cevap gövdesi bize yeni bir sürüm numarası olarak 2 dönecektir:

````
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```

Ancak, biz aynı isteği tekrar gönderecek olursak bu (yani sürüm numarası 1 
olarak) Elasticsearch bize "409 Conflict" başlıklı cevap ile aşağıdaki gibi bir
cevap dönecektir:

```
{
   "error": {
      "root_cause": [
         {
            "type": "version_conflict_engine_exception",
            "reason": "[blog][1]: version conflict, current [2], provided [1]",
            "index": "website",
            "shard": "3"
         }
      ],
      "type": "version_conflict_engine_exception",
      "reason": "[blog][1]: version conflict, current [2], provided [1]",
      "index": "website",
      "shard": "3"
   },
   "status": 409
}
```

Bu bize `_version` numarasının 2 olduğunu gösterir ve bizim 1. sürümü 
değiştirmeye çalıştığımızı gösterir.

Şimdi uygulamamızın gerekliliklerine göre, kullanıcılarımıza bu dökümanı 
başkalarının zaten değiştirdiğini söyleyebiliriz, kaydetmeye çalıştığı dökümanı
gösterebiliriz. Alternatif olarak, biz en son dökümanı tekrar çekip değişikliği 
tekrar kaydetmeyi deneyebilirsiniz. 

Dökümanı güncelleyen ya da silen her API uygulamanızın eşzamanlılık kontrolü 
yapabilmesine izin vermek için `version` parametresi alır.

### Dışardan (Kendimiz) Sürüm Kullanmak

Ana veritabanı olarak Elasticsearch dışında bir veritabanı sistemi kullanmak 
genel bir yöntemdir ve Elasticsearch veriyi aranabilir kılar. Bunun anlamı
bütün verinizin bir yansısı ana veritabanından bulunmasıdır ve Elasticsearch'e 
ayrı bir işlemler ile taşınır. Eğer birden fazla uygulama ya da işlemci bu 
eşzamanlamadan sorumlu ise daha önce de bahsettiğimiz eşzamanlılık sorunları
ile karşılaşabilirsiniz. 

Eğer sizin ana olarak kullandığınız veritabanı sisteminizin bir sürüm numarası 
veya değeri varsa bu numara Elasticsearch arayüzünde de sürüm numaralandırması
için kullanılabilir. Bunun için ekstra olarak `version_type=external` 
parametresini ve sürüm numarasını göndermeniz yeterlidir. Sürüm numarası 
`integer` değerinde olması ve sıfırdan büyük olmadıdır. Ekstra olarak 
`9.2e+18--a` pozitif sayısından küçük olmalıdır. 

Dış sürüm numarası içeride kullanılan sürümlemeye göre biraz farklı çalışır. 
Elasticsearch şu anki sürüm numarasının gönderilen ile aynı mı değil mi 
olduğunu kontrol etmek yerine, şu anki sürüm gönderilenden daha küçük mü
diye kontrol eder. Eğer küçük ise yeni gönderilen sürüm numarası dökümanın
yeni sürüm numarası olarak kaydedilir. 

Dış sürüm numaraları sadece güncelleme ve silme sırasında değil ayrıca yeni
döküman oluşturulurken de gönderilebilir.

Örneğin, yeni bir blog yazısı oluşturuken dışardan bir sürüm numarası olarak 
5 değerini göndermek için aşağıdaki gibi bir istekte bulunuruz:

```
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```
Cevapta yeni sürüm numarasını 5 olarak görebilirsiniz:

```
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 5,
  "created":  true
}
```

Şimdi dökümanı tekrar güncelleyip yeni sürüm numarasını 10 olarak gönderelim:

```
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```

Sonuç başarılı ve yeni sürüm numarası 10:

```
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```

Eğer bu sorguyu tekrar çalıştırırsanız çakışma hatası ile karşılaşacaksınız 
çünkü yeni göndereceğiniz sürüm numarası eskisinden daha büyük olmalı.

Kaynaklar : 
 - [https://www.elastic.co/guide/en/elasticsearch/guide/current/optimistic-concurrency-control.html](https://www.elastic.co/guide/en/elasticsearch/guide/current/optimistic-concurrency-control.html)
 - [https://www.elastic.co/guide/en/elasticsearch/guide/current/version-control.html](https://www.elastic.co/guide/en/elasticsearch/guide/current/version-control.html)