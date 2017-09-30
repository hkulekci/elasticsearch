---
layout: post
title: Kibana'yı Zaman Damgası (Timestamp) Olmayan Kayıtlar İle Kullanma
categories:
- blog
summary: Kibana arayüzü genellikle zaman damgası olan log kayıtları üzeribde grafikler oluşturarak kullanılır. Bu sadece bununla sınırlı değildir. Index'i tanımlarken zaman damgasını göz ardı ederek verilerinizi görselleştirebilirsiniz. 
---

Daha önceden sunumlarımda ve sohbetlerde bir çok kez şunu söylemiştim. Kibana 
arayüzünde zaman damgası olan kayıtlar üzerinde görselleştirme yapıyoruz. Bu 
konuda öncelikle beni dinleyenlerden özür dileyerek bu yazıma başlamak istiyorum.
Bir arkadaşım ile kibana arayüzü hakkında sohbet ederken farkettik ki 
olabiliyormuş. Kendisine de beni çok teşekkür ederim. 

Zaman damgası olmayan kayıtları da rahatlıkla bu arayüzden görselleştirebilirsiniz. 
Nasıl mı? Çok basit. Kibana arayüzünde index'inizi tanımlarken bir işaret 
kutusundaki işareti kaldırarak. 

![http://elasticsearch.kulekci.net/assets/img/kibana-index-contains-time-based-events-checkbox.png](http://elasticsearch.kulekci.net/assets/img/kibana-index-contains-time-based-events-checkbox.png)

Arayüzde diğer herşey aynı şekilde ilerleyecektir. Ancak bu durumda şuna dikkat 
etmenizi öneririm. Sıralama değişecektir. Kayıtlarınızı sıralaması zaman bazlı 
olmadığından en son gelen kayıt en üst sırada olmayacaktır. Ayrıca üst başlık 
satırındaki zamana göre verilerinizi kısıtlama ekranı da gelmeyecektir.

![http://elasticsearch.kulekci.net/assets/img/kibana-time-range.png](http://elasticsearch.kulekci.net/assets/img/kibana-time-range.png)

Eğer tarih bazlı olmayan şekilde index'i eklerseniz sıralamanız `_score`'a göre 
eğer tarih bazlı eklerseniz bu durumda varsayılan sıralamanız seçtiğiniz tarih 
alanına göre azalan olarak gelecektir. 

Tarih bazlı kullanımlarda `Discover` ekranında varsayılan olarak tarih alanı 
gelirken, tarih bazlı kullanmazsanız gelmeyecektir. Bunlar dışında gördüğüm 
kadarıyla çok fazla da farklılık yok. Aramalar ve grafikler iki yöntem için de 
geçerli ve çalışıyor.