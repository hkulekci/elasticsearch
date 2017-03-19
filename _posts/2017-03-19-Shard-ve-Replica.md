---
layout: post
title: Shard ve Replica
categories:
- blog
summary: Elasticsearch'ün temel konularından olan Shard ve Replica nedir, neden önemlidir ve hayatımıza ne katmaktadır kısa bir göz atacağız. 
---


 > Bazı kavramları bilerek ve isteyerek Türkçe'ye çevirmedim. Çünkü araştırma yaparken ingilizcesi üzerinden araştırma yapmak daha çok sonuç getirecektir ve bilgiye daha rahat ulaşılacaktır.

Bir "index", bir "node"a sığmayacak kadar büyük veri setini içerebilir. Örneğin bir "index" içerisinde milyarlarca döküman olabilir ve bunların büyüklükleri 1TB'a ulaşabilir ve bir "node" üzerindeki disklerinizin büyüklüğü bu kadar olmayabilir. Diğer taraftan sadece bir node üzerinden bu büyük veriyi sunmanız aramalarınızı yavaşlatabilir. 

Bu problemi çözmek için, Elasticsearch "shard" adı verilen, index'inizi alt parçalara bölen, bir yapı sunmaktadır. Elasticsearch sayesinde bir "index" oluşturueken rahatça kaç "shard" olması gerektiğini belirtebilirsiniz. Her bir "shard" aslında kendi başına tam donanımlı birer "index"dir.

"Sharding" 2 sebeple önemlidir:

 - Yatayda ölçeklendirme imkanı sunar
 - Arama Performansınızı/İşlem hacminizi artırmak için işlemlerinizi birden fazla "shard"a(ya da "node"a) dağıtır ve paralel bir şekilde çalışmasını sağlar.

"Shard"ların nasıl dağıtık olduğunu ve "shard"lara ayrılmış dökümanların arama sırasında nasıl geri birleştirildiği tamamen Elasticsearch tarafından yönetilmektedir ve kullanıcı olarak ekstra bir çaba harcamanız gerekmez. 

Ağ ve bulut mimarilerinde herhangi bir zamanda hatalar olabilmektedir. Bir "node" ya da "shard" herhangi bir sebeple ulaşılamaz olduğunda ya da bozulduğunda, bu hata durumunu düzeltecek bir sistemin olması gerçekten çok yararlıdır ve aynı zamanda tavsiye edilmektedir. Bu amaçla, Elasticsearch "index shard"larının bir veya daha fazla kopyalarını oluşturmaktadır ve bunlara "replica shard" denilmektedir. İngilizce olarak kısaca "replicas" denilmektedir. 

"Replication" 2 sebeple çok önemlidir: 

 - Shard veya node hata verdiğinde yüksek erişilebilirlik imkanı sunar. Bu sebeple, bir replica shard'ın, kendini kopyaladığı ana shard ile aynı node içerisinde olmamaması gerekmektedir. 
 - Aramalarınız paralel olarak tüm replica'larda çalıştığı için arama hazminizi veya işlem hacminizi ölçeklendirmenize olanak sağlar.

Özetlemek gerekirse, her bir "index" bir veya daha fazla "shard"a ayrılır. Bir "index"te hiç "replica" olmayabilir veya birden fazla da olabilir. Bir kez "replica" edilirse her bir "index"in bir "primary shard"ları ve aynı zamanda "primary shard"ların kopyaları olan "replica shard"ları olacaktır. "Shard" sayıları ve "replica" sayıları "index" oluşturulurken her bir "index" için ayrı ayrı verilebilir. "index" oluşturulduktan sonra "replica shard" sayısı dinamik olarak değişitirilebilirken, "shard" sayısı değişitirilemez. 

Varsayılan olarak, Elasticsearch'de her bir "index" 5 "primary shard" 1 "replica" ile oluşturulur. Bunun anlamı, eğer "cluster" içerisinde 2 "node" var ise 5 "primary shard" ve 5 "replica shard" ile birlikte toplamda her bir "index"iniz için 10 "shard" olacaktır.

 > **Note:**
 > Her Elascticsearch shard'ı bir **Lucene Index**'dir. Bu "index" için bir maximum döküman sayısı mevcuttur. [LUCENE-5843](https://issues.apache.org/jira/browse/LUCENE-5843)'e göre limit 2,147,483,519 (= Integer.MAX_VALUE - 128) dökümandır. Shard boyutunuzu `_cat/shards` isteğinde bulunarak görebilirsiniz.