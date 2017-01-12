---
layout: post
title: Nested Veri Tipi
categories:
- blog
summary: Nested veri tipine kısaca bir göz atacağız ve Elasticsearch bu veriyi nasıl tutuyor onu inceleyeceğiz. İçerisinde nested veriler bulunan bir index üzerinde aramalar yapıp aslında belkide bilmeden yanlış aramalar yaptığımız bir index'i düzelteceğiz. 
---

Aslında `nested` veri tipi `object` veri tipinin biraz daha özelleştirilmiş halidir. Bu özellik alt nesne olarak eklediğimiz verilerimizi bağımsız olarak indexleme imkanı ve her birini bağımsız olarak arama imkanı sağlar. Bunu nasıl sağlamaktadır kısaca bir göz atalım.

Lucene'de nested objeler için bir yapı bulunmamaktadır. Örneğin siz aşağıdaki gibi nested bir yapıda olan JSON objesini Lucene'de store etmek isterseniz:

```
{
	"id": 1,
	"type": {
		"id": 1,
		"name": "Dram"	},
	"name": "Schindler's List"}
```

Lucene'de bu şöyle bir yapıda olacaktır:

```
{
	"id": 1,
	"type.id": 1,
	"type.name": "Dram",
	"name": "Schindler's List"}
```

Aslında Elasticsearch'de Lucene üzerine kurulduğu için bu gibi durumlarda aramaya Lucene gibi yaklaşmaktadır. Bu verdiğimiz küçük örnekte arama tarafında aslında bir sıkıntı çıkmamaktadır. Siz "Dram" diye arama yaptığınızda `type.name` alanında kelimeyi arayacaktır.  Şimdi biraz daha karmaşık bir JSON objesi yazalım:

```
{
	"id": 1,
	"producers": [
		{
			"firstname": "Steven",
			"lastname": "Spielberg"
		},
		{
			"firstname": "Gerald",
			"lastname": "R. Molen"
		},
		{
			"firstname": "Branko",
			"lastname": "Lustig"
		}
	],
	"name": "Schindler's List"
}
```

Şimdi bu karmaşık verinin Lucene'de nasıl olduğuna bir göz atalım: 

```
{
	"id": 1,
	"producers.firstname": [ "Steven", "Gerald", "Branko" ],
	"producers.lastname": [ "Spielberg", "R. Molen", "Lustig" ],
	"name": "Schindler's List"
}
```

Görsel olarak verimiz biraz küçüldü ve başka bir farklılık göremiyoruz gibi ancak şöyle bir durum oluştu. Nested olarak tuttuğumuz JSON objesinde `Steven Spielberg` için bir bağ vardı ve bu bir `producer`ı niteliyordu, isim ve soyisim olarak. Ancak ikinci objede bu bağı kurmak pek mümkün görünmüyor. Yani bu bağ kayboldu. Burada adı yine `Steven` olup soy ismi `Lustig` olan bir şahsı aradığımızda  yukarıdaki döküman  eşleşecektir. Ancak bu eşleşme doğru değildir. Aslında bizim verimizde `Steven Lustig` diye birisi yoktur. `Steven Spielberg` vardır. 

```
GET movies/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "producers.firstname": "Steven" }},
        { "match": { "producers.lastname":  "Lustig" }}
      ]
    }
  }
}
```

Bu durumda dizi olan alt objeleri bağımsız olarak indexlemek ve bunu yönetmeniz gerekmektedir. Bunun için iki yol var. Ya kendiniz ayrıca bir index açıp içerisine bu verileri ayrıca index'leyebilirsiniz ya da  `nested` objeler kullanabilirsiniz. `nested` tipini kullanırsanız bu alt objeleri Elasticsearch zaten bağımsız olarak index'leyecektir. Bunun anlamıda her bir obje diğerlerinden bağımsız olarak aranabilecektir. Bunu bir görelim.  

```
PUT movies

PUT movies/movie/_mapping
{
  "properties": {
    "producers": {
      "type": "nested"
    }
  }
}
```

Bu mapping ile birlikte normal obje yapısından farklı olarak `producers` alanı için tip olarak `nested` tipini kullandık. Veriyi kaydetme yönteminde herhangi bir değişikliğimiz oluşmadı. Buradaki değişikliği Elasticsearch hallediyor. Ancak arama yöntemi olarak eskisi gibi düz bir `match` arama sorgusu gönderemeyeceğiz. Arama sorgumuzu da `nested` olarak değiştirmemiz gerekmektedir. Aşağıdaki örnek sorguya bir göz atalım:

```
GET movies/_search
{
  "query": {
    "nested": {
      "path": "producers",
      "query": {
        "bool": {
          "must": [
            { "match": { "producers.firstname": "Steven" }},
            { "match": { "producers.lastname":  "Lustig" }} ￼
          ]
        }
      }
    }
  }
}
```

Gördüğünüz üzre bu sorguda `Steven Lustig` diye arama yaptığımızda sonuç vermezken `Steven Spielberg` diye arama yaptığımızda sonuç verecektir. 





