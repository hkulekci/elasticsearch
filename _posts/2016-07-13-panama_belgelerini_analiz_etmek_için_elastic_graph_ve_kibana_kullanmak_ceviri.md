---
layout: post
title: Panama Belgelerini Analiz Etmek için Elastic Graph ve Kibana Kullanmak (Çeviri)
categories:
- blog
summary: Elasticsearch'ün eklentisi Graph ile Panama Belgelerini inceleyeceğiz ve Graph Kibana eklentisi ile de bu verileri görselleştireceğiz.
---

Yeni Elastic Graph'ın yetenekleri veri içerisindeki bağlantıları daha kolay 
analiz etmemizi sağlar. Eğer işiniz Panama Belgelerindeki yabancı ülkelerin 
içinden çıkılmayacak finansal düzenlemelerini veya revaçtaki bir e-ticaret 
sitesinin yüksek seviye tıklama davranışlarını takip etmek ise, Graph 
teknolojisi bu ilişkileri size sağlamakta yardımcı olabilir.

Graph'ın yetenekleri Elastic Stack için ticari X-Pack eklentisinin bir parçası 
olarak gelmektedir. Paket bir Kibana uygulaması ve yeni bir Elasticsearch API 
uçnoktası içermektedir. Bu ilk Graph yazımızca biz API'ın ve Kibana arayüzünün 
bize ne sundukları ile ilgili kısa bir göz atacağız.

### Forensic Analiz: Panama Papers

Panama Belgeleri, Off-shore hukuk firması Mossack Fonseca tarafından 
[Mali ve Yasal kayıtların yayınlanması](https://panamapapers.icij.org/) 2016'nın
en çok tepki toplayan haberlerinden birisidir. Kayıtlar bir çok politikacı, 
kraliyet üyesi, zengin ve onların ailelerinin deniz aşırı vergi rejimlerini 
kullanmak için kurulan petrol şirketlerini açığa çıkardı. Gazeteciler ve finans 
kuruluşları dikkatli bir şekilde bu veriler üzerinde odaklandılar(çalıştılar) 
ama bağlantıları ortaya çıkarmak gerçekten çok zor oldu ve de zaman aldı ama 
Kibana Graph uygulaması bunu herkes için kolay hale getirdi:

![Panama Paper Graph Dashboard Relationships](https://www.elastic.co/assets/blt0f07da9e0ac0dc6b/panama-papers-graph-dashboard-relationships.jpg)

Yukarıda Viladimir Putin'in yakın arkadaşı 
[Sergei Roldugin](http://www.theguardian.com/news/2016/apr/03/panama-papers-money-hidden-offshore)'in, 
şirketler ve bireysel kişiler ile ilişkilerini görüyorsunuz. Bu resim Graph ile 
bir kaç basit adım ile oluşturulabilir:

### Veri kaynağını seçmek

Başlangıçta indis listemizden "panama"yı seçiyoruz ve sonra içerisinde diagramda 
göstermek istediğimiz veriler bulunan bir kaç alan(field) seçiyoruz (Varsayılan 
olarak tek bir alan(field) seçimine izin veriliyor ancak sağ üssteki tuşlarda 
"Advanced Mode"a tıkladığınızda daha fazla alan seçebiliyor olacaksınız). 
Diagramda görünen "düğüm"ler için her bir alanı temsil eden bir ikon ve renk 
verilir.

![Panama Papers Selecting Datasource](https://www.elastic.co/assets/blta22818e6b496f4d1/panama-papers-selecting-datasource.jpg)

### Arama Yapmak

Şimdi Putin'in arkadaşının adını "Roldugin" içeren dökümanlara ulaşmak için bir 
normal bir metin araması çalıştıralım.

![Panama Papers Running A Search](https://www.elastic.co/assets/bltf24921afe4b5ad83/panama-papers-running-a-search.jpg)

Döküman içerisinde eşleşen terimler bir ağ olarak gösterildi - her bir çizgi 
terimler ile eşleşen bir veya daha fazla dökümanı göstermektedir. 

ICIJ'deki veriyi çıkaran gazeteciler belgelere eklenmiş her bir dökümanın 
referans verdiği gerçek hayattaki her bir birim/varlık için (person/company/
address) tekil bir ID vermeyi denediler. Ne yazık ki, insan isimleri ve adresler 
eşleşmeler için uygun olmayabilir - gazeteciler 3 dökümanda ilişkili halde 
bulunan 12180773 kimlik numaralı Person varlığı belirlediler ama bu kişi ile 
benzer isimde iki kişi daha olduğunu rahatça görebiliriz ve bu kişiler farklı 
bir kimliğe sahipler. Gelecekteki bir yazımızda da, otomatik varlık çözümleme 
(Automated Entity Resolution) için Graph API'ın kullanımı hakkında konuşacağız. 
Şimdilik bunu elle (gruplama aracı ile) düzenleyelim.


### Düğümleri Gruplama

Gelişmiş mod aracını kullanarak seçebiliriz(Sağdaki "Selections" kutusunu), 
sonra düğümleri birleştirmek için `group` düğmesine basın. Bu bize daha temiz 
bir resim verecektir. (Bir düğüme tıklayın ve daha sonra "linked" tuşuna 
tıklayarak bağlantılı diğer düğümleri seçin. Sonrasında bazılarının seçimini 
kaldırmak istiyorsanız listelenen düğümlerin ikonlarına tıklayarak 
kaldırabilirsiniz.)

![](https://www.elastic.co/assets/blt2e6f3a71812dab8f/panama-papers-grouping-vertices.jpg)

Eğer istiyorsak, gruplanmışları da ayrıca gruplayabiliriz. Örn, farklı 
kimliklere sahip kişilerin bir düğümde toplanması ve sonra bunları şirket 
düğümünde birleştirme.

### "Spidering out"

Graph en ilişkili bağlantıları gösterecektir. Ancak siz farzedelim varlıklarla 
bağlantılı diğer varlıklarıda (entities) görmek istiyoruz? Araç çubuğundaki "+" 
düğmesini kullanarak varlıklar ile ilişkili diğer bağlantıları keşfetmeye devam 
edebiliriz. 

![](https://www.elastic.co/assets/blta23d518f9fd9a049/panama-papers-spidering-out.jpg)

"+" tuşuna tekrar tekrar basarak resmi genişletebiliriz ve grafiğin belirli bir 
alanına odaklananabiliriz. "undo" ve "redo" tuşları ilgisiz sonuçlardan geri 
dönmek için önemli tuşlardır. Ek olarak, "delete" ve "blacklist" tuşları ile 
düğümlerin tamamen sonuçlardan çıkarılmasını ya da sadece karalisteye alınmasını 
sağlayabilirsiniz. Seçilen düğümlerin örnek dökümanlarıda "Show example docs" 
tuşu ile ayrıca görüntüleyebilirsiniz.

Eğer Panama Belgelerini kendiniz görüntülemek istiyorsanız. 
[Elasticsearch](https://www.elastic.co/downloads/elasticsearch), 
[Kibana](https://www.elastic.co/downloads/kibana), 
ve [Graph plugin](https://www.elastic.co/downloads/graph) eklentisinin birer 
kopyasını edinin ve [bit.ly/espanama](bit.ly/espanama) şuradaki kurulum 
adımlarını izleyin.

### Wisdom of crowds

Panama Belgeleri her tekil dökümanın önemli bağlantılarının olduğu "forensic" tipinde 
bir soruşturma örneğidir. 

Ama, Elastic Graph kullanıcı tıklama günlüklerinin bulunduğu verilerin 
davranışlarının özetlenmesi için de rahatça kullanılabilir. Analizin bu formunu 
tanımlamak için kullanılan genel ifadeler "collective intellegence" veya "wisdom 
of crowds"dur. Bu senaryoda, her bir dökümanın kendisi ilgili değildir ama bir 
çok kullanıcının davranışları bir desen oluşturabilir ve onlar öneri sisteminde 
kullanılabilir. Örn; "X ürününü satın alan kullanıcılar Y ürününü almak 
isteyebilir.". Bu senaryoda bizim sahte bağlantılar oluşturan tek seferlik 
dökümanların bağlantılarını ve zaten bilinen bağlantıları önlememiz gerekir 
(Örneğin X ürünü alan insanlar zaten süt almak zorunda ise). Bu düşüncede 
varsayılan ayarlar sadece en önemli bağlantıları tanımlamak için ayarlanmıştır.

Öneri kullanımı için [LastFM dataset](http://mtg.upf.edu/node/1671) veri setine 
bir bakalım. Eğer, her bir dökümanında her bir dinleyici için sevdiği 
şarkıcıları bir dizi içerisinde tutan kullanıcı merkezli bir index oluşturursak, 
bu index içerisinde insanların "Chopin"i aradığımızda bu şarkıcıyı sevenlerin 
başka kimleri sevdiğine ulaşabiliriz.

![](https://www.elastic.co/assets/blt3edb78a25587143f/lastFM-wisdom-of-crowds.jpg)

Birbiri ile ilişkili klasik müzik sanatçıları dönecektir. Bu sanatçılar 
arasıdaki çizgilere tıkladığımızda ne kadar dinleyicinin ilgili olduğunu 
görebilirsiniz. Neredeyse tüm Mendelssohn dinleyicilerinin yarısı ayrıca Chopin 
dinlemektedirler.

Graph API sadece _önemli ilişkileri_ tanımladı. Bu diğer Graph arama 
teknolojilerinden önemli bir ayrımdır.

### Popüler != Önemli

Ayalar sekmesine geçip "sadece önemli bağlantılar" özelliğini kasten kapatıp 
neler olduğunu bir görelim.

![](https://www.elastic.co/assets/bltba33a0df4f361b86/graph-popular-equals-significant.jpg)

"significant links" kutusu seçili değilken Chopin dinleyenler aramasını tekrar 
çalıştıralım ve farkı görelim.

![](https://www.elastic.co/assets/blt2c3bba7188423781/lastFM-wisdom-of-crowds-new.jpg)

Radiohead ve Coldplay gibi (dünyada) popüler sanatçılar şimdi sonuçlara 
sızmıştır kesin. 5,721 Cohip severin 1,843 tanesi Beatles'da sevmektedir. Biz 
bunu süt satın alan kişiler gibi 
["commonly common"](http://www.infoq.com/presentations/elasticsearch-revealing-uncommonly-common) 
olarak tanımlıyoruz. "Significant links" düğmesini açtığımızda "uncommonly 
common" olarak cağırdığımız gürültülere dikkat etmeden direk konumuza 
odaklanıyor olacağız. Aslında yıllardır arama motorlarına güç veren TF-IDF 
algoritması da aynı ilkelere dayanmaktadır.

_By reusing these relevance ranking techniques we can stay "on topic" when exploring connections in data. This is an important distinction from graph databases which have no concept of relevance ranking and are typically forced to employ a strategy of just deleting popular items (see the problem of "supernodes")._

_Note: When performing detailed forensic work such as exploring the Panama Papers, it can help to keep "significant links" turned on to try avoid the super-popular companies but the "certainty" setting should be lowered from its default value of three to one. For wisdom-of-crowds type scenarios we want at least three documents to assert a link before we trust it whereas in forensics every document is potentially interesting._

### Özet

Umarız bu yazı Graph'ın iki ana modun kullanımı için hızlı bir girişi olmuştur:

 - Forensics: Her döküman potansiyel olarak ilgi çekicidir. Arama bireysel
 kayıt ve aktörlere odaklanır. 
 - Wisdom of crowds: Büyük ölçekli davranışların "büyük resmine" odaklanır. 
 Bir çok gürültü yaratacak bağlantı mevcuttur ancak verideki sadece gerçekten
 önemli olan bağlantılara odaklanır.

Gelecekteki yazılarımızda diğer özel kullanım alanlarına ve Kibana 
uygulamasının arka planda kullandığı Elasticsearch API'ını nasıl kullandığına 
daha derin bakıyor olacağız.

Bizi takip edin...


Kaynak : [https://www.elastic.co/blog/using-elastic-graph-and-kibana-to-analyze-panama-papers](https://www.elastic.co/blog/using-elastic-graph-and-kibana-to-analyze-panama-papers)