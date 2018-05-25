---
layout: post
title: Elasticsearch'de Türkçe Karaterli Sıralama
categories:
- blog
summary: Elasticsearch'de Türkçe karakter içeren verilerde sıralama nasıl yaparız ve neden böyle bir yöntem kullandık açıklamaya çalıştık.
---

Uzun zamandır yazamadığım Elasticsearch yazılarına tekrar sık aralıklarıla devam edebilmek ümidiyle bir yazı daha yazayım dedim. Herkesin projelerinde çoğunlukla kullanacağı bir özelliği ES'de nasıl yaparız bir iredeleyelim. Örnek için ES 6.2.4 sürümünü kullandım. Daha öndeki sürümlerde çalışmayabilir. İlk olarak Türkçe sıralama için neden bir ekstra uğraşa ihtiyaç duyduğumuzu anlayalım. 

İlk olarak örnek için bir index ve type oluşturalım.

```
PUT users
{
  "settings": {
    "number_of_shards": 1
  }
}
POST users/user/_mapping
{
  "properties": {
    "name": {
      "type": "keyword"
    }
  }
}
```

Şimdi index'e biraz veri ekleyelim. 

```
POST users/user/1
{
  "name": "Haydar"
}
POST users/user/2
{
  "name": "Ümit"
}
POST users/user/3
{
  "name": "Rasim"
}
POST users/user/4
{
  "name": "zeynep"
}
POST users/user/5
{
  "name": "İsmail"
}
```

Verileri ekledikten sonra artık bir arama yapıp varsayılan değerler ile bir sıralama yaptığımızda nasıl bir sonuç verecek görelim.

```
GET users/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "name": {
        "order": "asc"
      }
    }
  ]
}
```

Sıralama sonucu dönene değerler şu şekilde olacaktır: 

```
Haydar
Rasim
zeynep
Ümit
İsmail
```

şeklinde olacaktır. Peki bunun sebebi ne? Bunun için biraz kodları eşeleyelim. İlk olarak sorgumuzun nerelere gittiğini inceleyelim. ES'e gönderdiğimiz sorgu aşağıdaki kod kısımlarından geçiyor. Bu sırasıyla ya da arada başka katmanlar olmadan geçiyor demek değil. Bunlara uğruyor diyelim.  

 - `org.apache.lucene.search.IndexSearcher:searchAfter(....)`
 - `org.apache.lucene.search.TopDocs:merge(....)`
 - `org.apache.lucene.search.TopDocs:mergeAux(....)`
 - `org.apache.lucene.search.MergeSortQueue/ScoreMergeSortQueue` (Biz `field sort` yaptığımız için `MergeSortQueue`ye yöneldik. Sort gelirse Score'a göre sıralama yapılıyor.)
 - `org.apache.lucene.search.FieldComparator`
 - `org.apache.lucene.search.TermValComparator:compareValues(T first, T second)` bu sınıfta FieldComparator'u kapsıyor. Bu sınıf değerleri byte array olarak karşılaştırıyor.
 - `java.lang.Comparable<T>` interface'i uygulayan `java.lang.String:compareTo(...)` veya `java.lang.Byte:compateTo(...)` metodları kullanılıyor. 

Burada anlatmak istediğim sıralama için Java'nın temel metodları kullanıyor. Şimdi türkçe karakterli bir compate yapacağımız küçük bir Java konsol uygulaması yazalım ve sonuçları görelim: 

```
public class Main {

    public static int compareValues(String first, String second) {
        if (first == null) {
            if (second == null) {
                return 0;
            } else {
                return -1;
            }
        } else if (second == null) {
            return 1;
        } else {
            return first.compareTo(second);
        }
    }

    public static void main(String[] args) {
        System.out.println(compareValues("İsmail", "Zeynep"));
        System.out.println(compareValues("Haydar", "Zeynep"));
    }
}

// Result:
// 214
// -18
```

Gördüğünüz gibi iki sonuçta aslında `-` eksi çıkması gerekirken `İ` sanki `Z`den sonra geliyormuş gibi oldu. Bu da bize neden sıralamanın bizim için hatalı çıktığını açıklıyor. Şimdi sorunu anladık. Gelelim çözüme.

Burada bizim ilk olarak sıralamaya gönderdiğimiz verilerde Türkçe karakter varsa onları bir süzgeçten geçirmemiz gerekecek. İlk olarak bizim [ICU* Analysis Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-icu.html) eklentisini yükleyelim.

```
sudo bin/elasticsearch-plugin install analysis-icu
```

Daha sonra verilerimizi barındıracağımız `type` için mapping'i biraz değiştirelim.

```
PUT users/user/_mapping
{
  "properties": {
    "name": {
      "type": "keyword",
      "fields": {
        "sort": {
          "type": "icu_collation_keyword",
          "index": false,
          "language": "tr",
          "country": "TR"
        }
      }
    }
  }
}
```

Son olarak verilerimizi tekrar index'leyip sorgumuzuda yeni veri yapımıza uygun hale getirdikten sonra : 

```
GET users/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "name.sort": {
        "order": "asc"
      }
    }
  ]
}
```

Sonuç seti artık aşağıdaki gibi olacaktır. 

```
Haydar
İsmail
Rasim
Ümit
zeynep
```


 - * ICU => [International Components for Unicode](http://site.icu-project.org/)