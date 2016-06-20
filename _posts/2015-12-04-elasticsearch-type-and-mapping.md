---
layout: post
title: Elasticsearch Type ve Mapping
categories:
- blog
summary: Elasticsearch'de `type`ın ne olduğunu ve nasıl oluşturabilsiniz? `Type`ı ne için kullanabilirsiniz? `Type Mapping` nedir ve ne için kullanılır?
---

Elasticsearch'te `type` benzer dökümanlar sınıfıdır. Örneğin, `users` adında bir index'iniz
varsa, bu index içerisinde `user` diye bir `type` oluşturabilirsiniz. Ya da kullanıcı role'lerine 
göre `type`'lar oluşturabilirsiniz. `admin`, `user`, `staff` gibi. 

Elasticsearch'te `mapping` kavramı, daha önceden de belirttiğim üzere verinin yapısıyla ilgilidir. 
Veritabanı şeması gibi düşünebilirsiniz. Mappingler, verilerinizin hangi alanlardan oluştuğunu, bu 
alanların tiplerinin ve özelliklerinin neler olduğunu belirtirler. Hangi alanların `index`e 
alınacağını yada hangi alanların Lucene'de tutulacağını belirler. 

Elasticsearch daha önceden belirttiğimiz üzere Lucene tabanlıdır. Peki Lucene 
Elasticsearch üzerinden gönderdiğimiz dökümanları nasıl görür ve nasıl saklar? 

Lucene'de bir döküman basit `field-value` (`alan-değer`) çiftlerini içerir. Bir alanın en az 
bir değeri ya da birden fazla değeri olmalıdır. Bazen, tek bir `string` değer analiz 
işlemleri sonunda birden fazla değere dönüşebilir. Bazen de uzun bir `string`  değeri
analizler sonucunda kısa bir string değerine dönüşebilir. Bunu örneklemek gerekirse 

```json
{
    "title": "Örnek bir başlık"
}
``` 

değeri bazı analizler sonunda aşağıdak gibi  

```json
{
    "title": {
        "original": "Örnek bir başlık",
        "filtered": "Ornek bir baslik"
     }
}
```

bir hal almış olabilir. Lucene, değerin `string`, sayı yada tarih formatında olmasını 
önemsemez. Tüm değerler `opaque bytes` olarak kabul edilir. 

Lucene'de biz bir dökümanı `index`lediğimizde her bir alan (field) için değerler (values) ilgili 
alan için `inverted index`e eklenir. İsteğe bağlı olarak, orjinal değerler sonradan ulaşılabilir 
olması için değişmeden de saklanabilir. 

Lucene'de şimdiye kadar her seferinde `field-value` tutulur diye konuştuk ve `type`dan hiç 
bahsetmedik. Peki, `Lucene`de bütün veriler `field-value` tutulurken Elasticsearch'deki 
`type` kavramı nasıl uygulanır? `Type`ların Lucene tarafında direk olarak bir karşılığı 
yoktur. Elasticsearch tarafında ise bir `index`te birden fazla `type` ve bunların her birinin 
kendi yapıları (mapping) vardır. Lucene tarafında bu veriler her bir dokümanın 
`meta data`sında `_type` diye bir alanında tutulur. Biz özel bir type'a göre bir arama 
yaptığımızda Lucene tarafında `_type` alanında bir filtreleme yapar.

Lucene'de aynı zamanda `mapping` diye bir kavram da yoktur. `Mapping`ler Elasticsearch'ün 
karışık JSON dökümanlarını Lucene'in beklediği bir yapıya sokmak için kullandığı bir ara
katmandır. 

Burada dikkat edilmesi gereken konu şudur. Biz Elasticsearch'de verileri `type`lara ayırdık 
gibi düşünsek de temelde o veriler Lucene'de aynı yerde tutulmaktadır. Bu da aynı index
içerisinde aynı alan (field) adı ile farklı türde yada farklı analiz edilmiş veriler tutmak anlamına 
gelir ki bu durum ileride başımızı ağrıtabilir. 

Burana çıkacak sorunları [azaltmak](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html#_avoiding_type_gotchas) 
için bu tür verileri farklı alan isimleri ile yada farklı indexlerde tanımlayarak sorunları 
çözebilirsiniz. 

Örneğin, kullanıcı ve şirket bilgileri tuttuğunuz bir Elasticsearch node'unuz olsun. Burada 
bilgileri bir index altında `type`lar ile tutabilirsiniz. 

```json
POST /core/user/_mapping
{
    "properties": {
        "id": {
          "type": "integer"
        },
        "name": {
          "type": "string"
        }
    }
}

POST /core/company/_mapping
{
    "properties": {
        "id": {
          "type": "integer"
        },
        "name": {
          "type": "string"
        },
        "description": {
            "type": "string"
        }
    }
}
```

Gördüğünüz gibi `name` ve `id` alanları aynı. Eğer index'imiz bu şekilde olacaksa bir sorun ile 
karşılaşmayabilirsiniz. Ancak ileride şirket isimleri için bir analiz işlemi uygulayarak aramalarda
datanın doğru sonucu vermesini isteyebilirsiniz. Bu durumda bir `type`'da bir analiz işlemi çalışırken 
diğerinde başka bir analiz işlemi çalışacaktır. Aramalarda bir karmaşa oluşacaktır. Bu iki `type`ı
ayrı index'ler olarak ayırmak daha mantıklı olacaktır. Ya da name alanlarını `user_name` ve 
`company_name` olarak değiştirmek de bir çözüm olacaktır. Böylelikle filtreleme yaparken 
bu alanlarda karışıklık olmayacaktır. 

#### Kaynakça 

 - [Elasticsearch - Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)
 - [Elasticsearch - Mapping Intro](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-intro.html)
 - [Types and Mappings](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html)
