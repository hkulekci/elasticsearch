### Elasticsearch ve Kibana'nın Beta Sürümleri için Docker İmajlarına Giriş

Elasticsearch ve Kibana için beta sürümleri için docker imajları burada! 5.0 sürümlerinin son sürümleri ve 
ayrıca [x-pack eklentisinin](https://www.elastic.co/downloads/x-pack) de içinde gelecektir. İmajlar 
Elastic'in kendi docker servisinde tutulmaktadır. 

Talimatlar [elasticsearch-docker](https://github.com/elastic/elasticsearch-docker) ve 
[kibana-docker](https://github.com/elastic/kibana-docker) github sayfalarında da bulunabilir, ama şimdi
Elasticsearch + Kibana ikilisinini bunlar ile ne kadar kolay olduğunu görelim.

Öncelikle şunlardan emin olalım:

 - [Docker Engine](https://docs.docker.com/engine/installation/) kurulu olmalı
 - [Ön hazırlıklar](https://github.com/elastic/elasticsearch-docker#host-prerequisites) yapılmış olmalı
 - Eğer Linux kullanıyorsanız [docker-compose](https://github.com/docker/compose/releases/latest) kurulu olmalı
 
Elasticsearch ve Kibana kurulumunda yardımcı olarak örnek [docker-compose.yml]() dosyanızı indir:

```
---
version: '2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana
    links:
      - elasticsearch
    ports:
      - 5601:5601

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch
    cap_add:
      - IPC_LOCK
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200

volumes:
  esdata1:
    driver: local
```

Daha sonra aşağıdaki komutları çalıştıralım:

```
$ docker-compose up
```

> Note: Yukarıdaki kurulum sizin hali hazırda Elasticsearch ve Kibana için 
> [varsayılan portları](https://gist.github.com/dliappis/52de1f4bb10fd23f5b91ea5fcb5d2560#file-docker-compose-yml-L9-L18) 
> kullanmadığınızı varsayar. Kendiniz bu portları docker-compose.yml dosyasından ayarlayabilirsiniz. 

Ekranda Elasticsearch ve Kibana'dan bazı günlükleri oluştuğunu göreceksiniz. Kibana arayüzüne girmek için
[http://localhost:5601](http://localhost:5601) adresini kullanın ve sizi bir Kibana 5.0 giriş sayfası 
karşılayacaktır.

![kibana-login-page](https://www.elastic.co/assets/blt396c10498991296c/Kibana-5.0-login.png)

[X-Pack](https://www.elastic.co/downloads/x-pack) önceden yüklenmiş olduğundan giriş yapmak için 
varsayılan bilgileri kullanabilirsiniz.(Kullanıcı adı: `elastic`, şifre: `changeme`) Bu bilgileri Kibana'da bulunan
yönetim aracını kullanarak kesinlikle değiştirmelisiniz.

Şimdi biraz veri ekleyelim.  Bunun için curl kullanarak `users` index'ine iki tane kullanıcı ekleyeceğiz:

```
curl -u elastic -XPOST 'localhost:9200/users/1' -d '{"first_name": "John","last_name": "Doe","email_address": "user@example.com"}'
curl -u elastic -XPOST 'localhost:9200/users/2' -d '{"first_name":"James","last_name":"Kirk","email_address":"jkirk@unknowngalaxy.com"}'
```

Kibana arayüzüne dönelim, ve veriler için hangi index'i kullanacağımızı ayarlayalım. Bu alana index adı 
olarak `users` girelim ve `Index contains time-based events` kutusunu işaretlemeyelim. (Çünkü şu anda 
zaman damgası olan bir verimiz yok. Eğer zaman damgası olan bir verimiz olursa, günlükler gibi, o zaman bu alanı işaretlemeniz iyi olur.). Ekranımız aşağıdaki gibi olacaktır:

![](https://www.elastic.co/assets/bltadc2c18663857c8b/Kibana-5.0-configureindex.png)

`Create` düğmesine tıklayalım ve eklemiş olduğumuz verileri Kibana arayüzünde artık görebiliriz.

![](https://www.elastic.co/assets/blt4f2f0d63e73f09a8/Kibana-5.0-discover.png)

Konteynerlarınızı `docker-compose down` komutu ile kapatabilirsiniz; bu komut oluşturduğunuz 
Elasticsearch veri klasörleri dahil konteynerleri tamamen silmeyecektir. `docker-compose up`u tekrar 
çalıştırdığınızda verilerinize tekrar ulaşabilirsiniz. `docker-compose down -v` ile hem verilerini hem de 
konteyneri kaldırabilirsiniz. 

İmajlar beta sürümüdür ve production ortamında kullanmanızı önermiyoruz. Ama Github sayfamızda bir kaç 
öneri bulunmaktadır, [kontrol edebilirsiniz](https://github.com/elastic/elasticsearch-docker#notes-for-production-use-and-defaults).

Kaynak : [https://www.elastic.co/blog/releasing-beta-version-of-elastic-docker-images](https://www.elastic.co/blog/releasing-beta-version-of-elastic-docker-images)