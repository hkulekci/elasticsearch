---
layout: post
title: Takma isim (Alias) kullanarak verileri filtrelemek
categories:
- blog
summary: Takma isimler kullanarak verilerinizi kolayca filtrelenmiş halleriyle sorgulayabilirsiniz. 
---

Bir önceki yazımda `index` için takma isim oluşturmayı ve bu takma ismi kullanmayı
görmüştük. Elasticsearch'ün yeni sürümü (2.x) ile birlikte takma isimler ile birlikte 
index üzerinde filtreleme de yapabiliyorsunuz. Bir örnek ile açıklamaya çalışalım. 

`id`, `type`, `keywords` alanlarından oluşan bir index ve tipimiz var. Aşağıdakine
benzer verilerin olduğunu düşünün. 

```
{
    "id": 1,
    "type": "search",
    "keywords": ["elasticsearch", "elastic", "framework"]
},
{
    "id": 4,
    "type": "client",
    "keywords": ["golang", "elasticsearch"]
}
```

Örnek veri için bir gist hazırladım. 
Göz atabilirsiniz: [https://gist.github.com/hkulekci/2146c1da124e42cd02ed3c4635934f62](https://gist.github.com/hkulekci/2146c1da124e42cd02ed3c4635934f62)

Bu verilerimiz bir için bir takma isim oluşturacağız ve ayrıca bu takma isim 
üzerinden yaptığımız bütün aramalarıda `type` alanı `client` olanları filtrelemesini
isteyeceğiz. Böylelikle bu alias aslında `index`imizin sadece bazı kısımlarını 
temsil etmiş olacak.

Peki bunu nasıl yapacağız. Bunun için önceki yazımızda gördüğünüz takma isim 
oluşturma yönteminde olduğu gibi `_aliases` arayüzünü kullanacağız ancak bu arayüze
verimizi gönderirken ekstra olarak `filter` diye bir parametre daha göndereceğiz.

Hemen bir örnek yapalım:

```
POST /_aliases
{
   "actions": [
      {
         "add": {
            "index": "test",
            "alias": "test_client",
            "filter": {
               "term": {
                  "type": "search"
               }
            }
         }
      }
   ]
}
```

Yukarıdaki örnekte `test` index'i için bir filtreleme oluşturdum ve bu takma ismi 
kullandığım zaman sadece `type` alanı `search` olan verileri kullanmak istediğimi
belirtmiş oldum. 

Şimdi gidip `/test_client/_search` aramasını yaptığımda bana sadece `type` alanı 
`search` değerinde olan verileri geri döndürecektir. Diğer veriler sanki bu 
index'te yok gibi davranacaktır. 

Bu özelliği bir çok şey için rahatça kullanabiliriz. Örneğin kendimize büyük 
filtrelemeler sonucu hazırladığımız sorgularımızı bir alias olarak tanımlayarak 
sanki SQL'deki view'lar gibi ya da `procedure`ler gibi kullanabiliriz. 
