---
layout: post
title: Nginx ile Elasticsearch için Koruma Oluşturun
categories:
- blog
summary: Nginx ile elasticsearch sorgularını şifre koruması altında yapabilirsiniz ve nginx logları ile elasticsearch sunucularınıza gelen yükü görebilir ve hatalarınızı ayıklayabilirsiniz. 
---

Elasticsearch HTTP API arayüzü üzerinden çalıştığı için bir sunucu üzerinde çalıştırdığınızda 
dış dünya tarafından ulaşılabilir halde çalışıyor olacaktır. Burada Elasticsearch erişimini dış
dünyaya yani sizin uygulamanız dışındaki kişilere erişimini kapatmak için bir kaç yol 
bulunmakta. Bunlardan birisi de "Basic Authentication". Yani bir kullanıcı adı şifre ile 
koruma yöntemi. Bu yöntem sizin sunucularınızı tabiki tamamen koruyamayabilir. Önerim 
firewall seviyesinde ya da network seviyesinde erişimin kısıtlanmasıdır. Ancak bu kısıtlama 
iç ağdan birilerinin de elasticsearch içerisinde rahatça işlem yapmalarını engelleyemez. 
Bunun için kullanıcı adı ve  şifre ile koruyarak buradaki erişimi iç ağ üzerinde de kısıtlamaya 
çalışacağız.

Bunu nasıl yapacağız şimdi kısaca buna değinelim. Gerekli araç ve gereçler : 

 - Nginx 
 - Elasticsearch
 
Hepsi bu kadar. Ben şimdilik Elasticsearch'ün varsayılan port'u üzerinden örnekleyeceğim. 
İlk olarak bir Elasticsearch makinesini çalıştırıyoruz ve [http://127.0.0.1:9200/](http://127.0.0.1:9200/) 
adresinden çalıştığını görüyoruz. 

Daha sonra Nginx üzerinden aşağıdaki gibi bir server oluşturuyoruz.

```
server {
    listen 80;
    server_name elastic.local;

    location / {
        proxy_pass       http://127.0.0.1:9200;
        proxy_set_header Host      $host;
        proxy_set_header X-Real-IP $remote_addr;

        auth_basic "Restricted Content";
        auth_basic_user_file /usr/local/etc/nginx/.htpasswd;
    }
}
```

Burada dikkat ettiyseniz `127.0.0.1:9200` adresi üzerinden çalışan Elasticsearch için 
`elastic.local` domain'inde ve 80. porttan çalışacak şekilde bir server oluşturduk. Daha sonra 
bu adrese gelecek bütün istekleri `proxy_pass       http://127.0.0.1:9200;` ile Elasticsearch
adresimize yönlendirdik. Böylelikle uygulamamız içerisinden Elasticsearch'e erişmek için 
`elastic.local` yazmamız yeterli olacaktır.  Bunun yanında firewall üzerinden 9200 portuna 
ulaşımı engelliyoruz ki 9200 portu üzerinden API arayüzü kullanılmaya devam etmesin 
sadece nginx'in olduğu sunucu üzerinden erişilebilir olsun. Bu örnekte ben kendi 
bilgisayarımda bu işlemi yaptığım için 9200 portu üzerinden erişim konusunu gözardı ettim. 
Domain yönlendirmesi için bilgisayarımdaki `/etc/hosts` dosyasına aşağıdaki gibi bir 
tanımlama yaptım. Eğer siz bunu sunucu düzeyinde yapmak istiyorsanız DNS tanımlamalarınızı 
yapmanız gerekmektedir. Ya da burada server_name olarak bir IP adresi kullanabilirsiniz.

```
# Host Dosyası tanımlaması
127.0.0.1       elastic.local
```

Şimdi bunları yaptık ancak koruma nerede. Koruma kısmıda yukarıdaki script'in devamında.

```
        auth_basic "Restricted Content";
        auth_basic_user_file /usr/local/etc/nginx/.htpasswd;
```

Bu kısımda da sayfamıza bir koruma ekledik ve Nginx'e `/usr/local/etc/nginx/.htpasswd` bu 
dosyada yazan bilgilere göre bu alana gelen istekleri koru dedik. Bu dosyada da aşağıdaki 
gibi Nginx'in anlayacağı dilde şifremizi oluşturduk. Kaynaklar arasında benim online da test 
için kullandığım bir şifre oluşturma sitesi vardı. Ancak tabiki dışarıdaki bir aracı şifre oluşturmak
için önermiyorum. Ben bu örnekte de şifrem olan `123456` için bu siteden kodu oluşturdum.
Dosyanın içeriğide aşağıdaki gibi : 

```
hkulekci:$apr1$0DffSiqd$IX9t3AE92RfD.zZc0E97t/
```

Şimdi son olarak Nginx'i yeniden başlatalım. 

```
sudo nginx -s reload
```

Artık Elasticsearch'e `elastic.local` adresi üzerinden erişebileceksiniz. İlk denememizi yapalım.

```
➜  ~ curl -XGET "http://elastic.local"
<html>
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.8.0</center>
</body>
</html>
```

Gördüğünüz gibi elasticsearch erişimini kısıtladık, ve erişmeye çalıştığımızda bize kullanıcı adı ve 
şifre soruyor. Şimdi elasticsearch için oluşturduğumuz şifre ile erişmeye çalışalım. 

```
➜  ~ curl --user hkulekci:123456  -XGET "http://elastic.local"
{
  "name" : "Black Jack Tarr",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.3.1",
    "build_hash" : "bd980929010aef404e7cb0843e61d0665269fc39",
    "build_timestamp" : "2016-04-04T12:25:05Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}
```

Bu işlem sonrasında elasticsearh'ü nginx arkasına aldık ve kullanıcı adı ve şifre ile koruduk. Bu 
korumanın yanı sıra nginx `access.log` ve `error.log` ile elasticsearch sorgularınız hakkında 
analizler de yapabileceksiniz. Benim örnek loglarım aşağıdaki gibi : 

```
127.0.0.1 - hkulekci [23/Apr/2016:07:14:59 +0300] "GET / HTTP/1.1" 200 340 "http://elastic.local/_plugin/head/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36"
127.0.0.1 - hkulekci [23/Apr/2016:07:15:00 +0300] "GET /_cluster/state HTTP/1.1" 200 97713 "http://elastic.local/_plugin/head/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36"
127.0.0.1 - hkulekci [23/Apr/2016:07:15:00 +0300] "GET /_status HTTP/1.1" 200 12427 "http://elastic.local/_plugin/head/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36"
127.0.0.1 - hkulekci [23/Apr/2016:07:15:00 +0300] "GET /_nodes HTTP/1.1" 200 3650 "http://elastic.local/_plugin/head/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36"
```

Teşekkürler.


### Kaynaklar : 
 - https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching
  - https://www.nginx.com/resources/admin-guide/reverse-proxy/
  - https://gist.github.com/soheilhy/8b94347ff8336d971ad0
  - http://www.htaccesstools.com/htpasswd-generator/