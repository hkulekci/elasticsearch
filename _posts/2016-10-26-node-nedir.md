---
layout: post
title: Node Nedir?
categories:
- blog
summary: Elasticsearch Node Nedir? Çeşitleri nelerdir?
---

Bir Elasticsearch programı çalıştırdığınızda her zaman bir `node` ile başlarsınız. Birbiri ile bağlantılı `node`ların oluşturduğu yapıya da `cluster` denir. Sadece bir `node` ile çalıştırırsanız, bir `node`dan oluşan bir `cluster` oluşur. Şöyle düşünün; bir sunucuda ya da bilgisayarda Elasticsearch uygulamasını ayağa kaldırdığınızda bir node'unuz olur. Aynı makinede ikinci bir Elasticsearch uygulaması ayağa kaldırdığınızda ikinci bir node'unuz olur. İki makine eğer aynı cluster ismine sahip ise birbirlerini görürler/tanırlar ve bir cluster oluştururlar. Burada ben her bir Elasticsearch node'u için düğüm kavramını kullanacağım. ()Aslında bu kelime beni çeviri konusuda tatmin etmedi ancak daha iyisinide bulamadım.) Yani bu örnek ile birlikte bir sunucuda 2 düğüm ile bir cluster(küme) oluşturmuş olduk. Bazı terimleri çeviri yapmadan direk ingilizce isimleri ile kullanacağım ki çok fazla kafa karışıklığı da olmasın. 

Bir `cluster` içerisindeki her `node`(düğüm) HTTP ve Transport trafiğini varsayılan olarak işleyebilmektedir. Transport katmanı düğümler arasındaki ve Java TransportClient arayüzü ile iletişimde kullanılmaktadır. HTTP katmanı ise sadece REST arayüzü için kullanılmaktadır.

Bir küme(cluster) içerisindeki her düğüm(node) diğer düğümlerden haberdardır ve gelen istekleri uygun diğer düğümlere yönlendirmektedir. Bunun yanı sıra, her bir düğüm(node) bir ya da bir kaç tür amaca hizmet eder:

 - **Master-eligible node (Master olmak için uygun node)** : Konfigürasyonununda `node.master` kısmı `true` olarak işaretlenmiş (varsayılanda böyledir) düğümlerdir. Bu düğümler aslında kümeyi kontrol eden düğümlerdir. Birden fazla ana düğüm işaretlenmiş düğümünüz olabilir. Burada Elasticsearch `Zen Discovery` özelliği ile bir tanesini ana düğüm olarak seçecektir.
 - **Data Node** : Konfigürasyonunda `node.data` kısmı `true` olarak işaretlenmiş düğümlerdir. Data node (veri düğümü) veriyi tutar ve veriyle ilgili CRUD, arama ve aggregation gibi işlemleri yürütür.
 - **Client Node** : Bir Client Node hem `node.master` hem de `node.data` kısımları `false` olarak işaretlenmiş düğümlerdir. Bunlar "smart router" (akıllı yönlendirici) olarak davranır ve kümeye(cluster) gelen istekleri uygun ana düğümlere(master node) ve veri ile ilgili istekleri uygun veri düğümlerine(data node) yönlendirir.

Varsayılan olarak bir düğüm hem veri düğümü(data node) hem de ana düğümdür(master-eligible node). Küçük kümeler için bu gerçekten iyi olabilir ancak kümeler büyüdükçe bu koordinasyonu ele almak ve sabit veri düğümleri ve ana düğümleri belirlemek daha iyi olabilir.

Kaynak : [https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html]()