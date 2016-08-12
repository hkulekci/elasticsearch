---
layout: post
title: Kendi Beat Uygulamanızı Oluşturun (Çeviri)
categories:
- blog
summary: Elasticsearch Beat ile çeşitli yerlerden (network, file, ...) verileri kolayca okuyup elasticsearch ya da logstash arayüzlerine gönderebilirsiniz. Biz de kendimiz için bir beat'i nasıl oluştururuz onu göreceğiz.
---

Beat verilerinizi Elasticsearch platformuna taşımanızı sağlayan açık kaynak ve 
hafif bir platformdur. [Packetbeat](https://www.elastic.co/guide/en/beats/packetbeat/master/index.html) 
sunucularınız arasındaki ağ trafiğini izleyebilmenize, 
[Filebeat](https://www.elastic.co/guide/en/beats/filebeat/master/index.html)
sunucularınızdaki log dosyalarınızı takip edebilmenize ve yeni yayınlanan 
[Metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/master/index.html)
'de periyodik olarak dış bir kaynaktan veri almanıza yardımcı olur. Eğer sizin 
daha farklı veriler toplama ihtiyacınız var ise bu durumda kendinize `libbeat` 
üzerine bir beat yazmanız gerekebilir. Hali hazırda 
[25+ Beat](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html) 
topluluklar tarafından yazılmış.

Biz kendiniz için Beat oluşturabilmeniz için bir 
[Beat oluşturucu](https://github.com/elastic/beats/tree/master/generate) 
paket imkanı sunuyoruz. Bu blog yazısında, Beat oluşturucuyu kullanarak nasıl 
bir Beat oluşturabileceğinizi göreceksiniz. Bu oluşturacağımız Beat'in adı 
`lsbeat`. `lsbeat` Unix komutu olan `ls` ile benzer şekilde dosyaların ve 
klasörlerin bilgilerini ES'e gönderecek. Bu yazı Unix tabanlı ilerlemektedir. 
Eğer Windows ya da diğer işletim sistemlerini kullanıyorsanız kendinize göre 
uyarlayın.

### Adım 1 - Golang Ortamını Kurun

Beat'ler Golang'de yazılmıştır. Haliyle yeni bir Beat oluşturmak istiyorsak, 
Golang'in bilgisayarınızda kurulu olması gerekmekte. 
[Kurulum](https://golang.org/doc/install) adımlarını izleyebilirsiniz. Şu anda 
Beat en az Golang 1.6 gerektirmektedir. Ayrıca `$GOPATH` kurulumunuzu düzgün 
yaptığınıza emin olun.

Şimdi lsbeat için kullanacağımız kodu görelim. Bu bir klasördeki tüm dosyaları,
klasörleri ve onların alt klasörlerini listeleyen basit bir Golang programı.

```
package main
import (
    "fmt"
    "io/ioutil"
    "os"
)
func main() {
    //apply run path "." without argument.
    if len(os.Args) == 1 {
        listDir(".")
    } else {
        listDir(os.Args[1])
    }
}
func listDir(dirFile string) {
    files, _ := ioutil.ReadDir(dirFile)
    for _, f := range files {
        t := f.ModTime()
        fmt.Println(f.Name(), dirFile+"/"+f.Name(), f.IsDir(), t, f.Size())
        if f.IsDir() {
            listDir(dirFile + "/" + f.Name())
        }
    }
}
```

Biz ileride `listDir` fonksiyonunu tekrar kullanacağız.

### Adım 2 - Oluşturma

Kendi beat uygulamanızı oluşturmak için 
[Beat Generator](https://github.com/elastic/beats/tree/master/generate/beat)'u 
kullanıyoruz. İlk olarak [cookiecutter](https://github.com/audreyr/cookiecutter)'ı 
kuralım. Kurulum adımlarını 
[buradan](http://cookiecutter.readthedocs.org/en/latest/installation.html) takip 
edebilirsiniz.  Cookiecutter'ı kurduktan sonra beat uygulamanızın adına karar
vermelisiniz. İsim tamamen küçük harflerden oluşmalıdır. Bu örnekte biz `lsbeat`
kullandık.

[Beat](https://github.com/elastic/beats) reposunda kullanılabilir durumda olan 
Beat Generator paketini, Beat iskeletini kurmak için çekmaliyiz. Daha önce 
kurduğumuz Golang ile Beats Generator paketini `go get` komutunu kullanarak 
indirebilirsiniz. Bu komutu çalıştırdığınızda, tüm kaynak dosyaları `$GOPATH/src`
dizini altına indirilecektir.

```
$ go get github.com/elastic/beats
```

Şimdi, GOPATH altında kendi dosyalarımızı oluşturup cookiecutter'ı Beat 
Generator yolunu ile çalıştırıyoruz.

```
$ cd $GOPATH/src/github.com/{user}
$ cookiecutter $GOPATH/src/github.com/elastic/beats/generate/beat
```

Cookiecutter size bir kaç soru soracaktır. `project_name` için lsbeat, 
`github_user` için sizin github kullanıcı adınızı girebilirsiniz. Sonraki iki 
soru zaten hali hazırda otomatik olarak belirlenmiş durumda. Son alan için 
adınızı ve soyadınızı girebilirsiniz. 

```
project_name [Examplebeat]: lsbeat
github_name [your-github-name]: {username}
beat [lsbeat]:
beat_path [github.com/{github id}]:
full_name [Firstname Lastname]: {Full Name}
```

Bu komut size lsbeat klasörü altında bir kaç dosya oluşturacaktır. Bu klasöre 
gidelim ve içerisinde neler oluştuğuna bir göz atalım.

```
$ cd lsbeat
$ tree
.
├── CONTRIBUTING.md
├── LICENSE
├── Makefile
├── README.md
├── beater
│   └── lsbeat.go
├── config
│   ├── config.go
│   └── config_test.go
├── dev-tools
│   └── packer
│       ├── Makefile
│       ├── beats
│       │   └── lsbeat.yml
│       └── version.yml
├── docs
│   └── index.asciidoc
├── etc
│   ├── beat.yml
│   └── fields.yml
├── glide.yaml
├── lsbeat.template.json
├── main.go
├── main_test.go
└── tests
    └── system
        ├── config
        │   └── lsbeat.yml.j2
        ├── lsbeat.py
        ├── requirements.txt
        └── test_base.py
```

Şimdi bizim basit bir Beat şablonumuz oluştu ancak halen projeye çekmemiz 
gereken bağımlılıklarımız var ve bir git reposu oluşturmamız gerekiyor.

İlk olarak bağımlılıklarımızı, ki bunlar şimdilik `libbeat` kütüphanesi ve basit 
bir kaç konfigurasyon ve şablon dosyası, çekmemiz gerekiyor. Bu şablonlara ve 
konfigürasyonlara daha detaylı bakacağız. 

```
$ make setup
```

Kendi Beat uygulamanızı oluşturduktan sonra artık topluluklar ile 
paylaşabilirsiniz.

![image](https://img.readitlater.com/i/www.elastic.co/assets/bltc6313851494a95b5/Screen%20Shot%202016-07-13%20at%2010.53.58%20AM/RS/w1408.png?&ssl=1)

Kodlarınızı Github'a göndermek için aşağıdaki komutları kullanıyoruz:

```
$ git remote add origin git@github.com:{username}/lsbeat.git
$ git push -u origin master
```

Şimdi oluşturduğumuz uygulamamızın ilk sürümünü Github üzerinde yayınlamış olduk. 
Şimdi kurulum ve çalıştırmak için kodu biraz daha derinlemesine inceleyelim.

### Adım 3 - Ayarlar

Önceki komutları çalıştırdığınızda `lsbeat.yml` ve `lsbeat.template.json` 
dosyaları oluşacaktır. Tüm basit ayarlar zaten bu dosyalarda bulunuyor. 

lsbeat.yml:

```
lsbeat:
  # Defines how often an event is sent to the output
  period: 1s
```

`period` tüm Beat uygulamarlı için var olan bir koşullu sabittir. Bu sabit 
sayesinde uygulamamızın hangi sıklıkta işleyeceğini belirtir. `period` alanını
1s'den 10s'ye değiştirelim ve programının hangi dizinleri tarayacağını 
belirleyen yeni bir `path` parametresi ekleyelim. Biz bu parametreleri `etc` 
dizini altındaki `beat.yml` dosyasına da ekleyebiliriz.

```
lsbeat:
  # Defines how often an event is sent to the output
  period: 10s
  path: "."
```

Yeni parametreler ekledikten sonra değişikliklerin `lsbeat.yml` konfigürasyon 
dosyasına gönderilmesi için `make update` komutunu çalıştırmalıyız. `etc/beat.yml`
üzerinde yaptığımız değişiklikleri şimdi `lsbeat.yml`da görebiliriz. 

```
$ make update
$ cat lsbeat.yml
################### Lsbeat Configuration Example #########################
############################# Lsbeat ######################################
lsbeat:
  # Defines how often an event is sent to the output
  period: 10s
  path: "."
###############################################################################
```


Konfigürasyon dosyalarını düzenledikten sonra, `config/config.go` dosyasını da 
düzenlemelisiniz. Buraya da `path` parametresini ekleyelim.

```
package config
import "time"
type Config struct {
    Period time.Duration `config:"period"`
    Path   string        `config:"path"`
}
var DefaultConfig = Config{
    Period: 10 * time.Second,
    Path:   ".",
}
```

Varsayılan konfigürasyon olarak 10s periyotunda ve aktif tarama klasörüs 
olarak da (.) klasörünü yazıyoruz.

### Adım 4 - Kodu Ekleme

Her bir Beat uygulaması `Beater` arayüzündeki `Run()` ve `Stop()` fonksiyolarını
uygulamalıdır. Daha detaylı bir kaynak için 
[şurayı](https://www.elastic.co/guide/en/beats/libbeat/master/beater-interface.html) 
inceleyebilirsiniz.

Bunun için sadece `Lsbeat` adında `Beater` arayüzünü uygulayan bir struct 
tanımlamalısınız. Son veri kayıt zamanını tutması için `lastIndexTime` 
ekleyelim. 

```
type Lsbeat struct {
    done   chan struct{}
    config config.Config
    client publisher.Client
    lastIndexTime time.Time
}
```

Ek olarak, her bir Beat konfigürasyon bilgilerinin erişebilmesi ve Lsbeat 
nesnesinin dönülebilmesi için `New()` fonksiyonunu uygulamalıdır.

```
func New(b *beat.Beat, cfg *common.Config) (beat.Beater, error) {
    config := config.DefaultConfig
    if err := cfg.Unpack(&config); err != nil {
        return nil, fmt.Errorf("Error reading config file: %v", err)
    }
    ls := &Lsbeat{
        done:   make(chan struct{}),
        config: config,
    }
    return ls, nil
}
```

Lsbeat için `Run()` fonksiyonunun varsayılanın üzerine dosya ve klasör 
bilgilerini de alacak şekilde değiltirelim. 

`Run()` fonksiyonunu değiştirmeden önce, Dosya ve klasör bilgilerini toplayan 
`listDir()` fonksiyonunu lsbeat.go dosyasının en sonuna ekleyelim. Bu fonksiyon 
aşağıdakileri içeren bir event oluşturacaktır:

 - "@timestamp": common.Time(time.Now())
 - "type": beatname
 - "modtime": common.Time(t)
 - "filename": f.Name()
 - "path": dirFile + "/" + f.Name()
 - "directory": f.IsDir()
 - "filesize": f.Size()

Uygulama ilk seferde tüm dosya ve klasörleri index'e atacaktır, ama sonrasında
eğer ilk seferde baktığına göre değişikliğe uğramış ya da oluşturulmuş dosya ve 
klasörleri kontrol edecektir ve sadece yeni dosyaları index'e atacaktır. 
Son seferdeki zaman damgası `lastIndexTime` alanına kaydedilecektir. 

```
func (bt *Lsbeat) listDir(dirFile string, beatname string) {
    files, _ := ioutil.ReadDir(dirFile)
    for _, f := range files {
        t := f.ModTime()
        path := filepath.Join(dirFile, f.Name())

        if t.After(bt.lastIndexTime) {
            event := common.MapStr{
                "@timestamp": common.Time(time.Now()),
                "type":       beatname,
                "modtime":    common.Time(t),
                "filename":   f.Name(),
                "path":       path,
                "directory":  f.IsDir(),
                "filesize":   f.Size(),
            }

            bt.client.PublishEvent(event)
        }

        if f.IsDir() {
            bt.listDir(path, beatname)
        }
    }
}
```

`io/ioutil` paketini eklemeyi unutmayın.

```
import (
    "fmt"
    "io/ioutil"
    "time"
)
```

Şimdi, `listDir()` fonksiyonunu çağıracak olan `Run()` fonksiyonunu görelim ve 
zaman damgasını `lasIndexTime` alanına yazalım.

```
func (bt *Lsbeat) Run(b *beat.Beat) error {
    logp.Info("lsbeat is running! Hit CTRL-C to stop it.")

    bt.client = b.Publisher.Connect()
    ticker := time.NewTicker(bt.config.Period)

    for {
        now := time.Now()
        bt.listDir(bt.config.Path, b.Name) // call listDir
        bt.lastIndexTime = now             // mark Timestamp

        logp.Info("Event sent")

        select {
        case <-bt.done:
            return nil
        case <-ticker.C:
        }
    }
}
```

`Stop()` fonksiyonu döngüyü sonlandırır ve otomatik oluşturulan ile aynı 
bıraktık.

```
func (bt *Lsbeat) Stop() {
    bt.client.Close()
    close(bt.done)
}
```

Neredeyse bitirdik. Şimdi elasticsearch için kullanılacak olan yapımıza yeni 
alanımızı ekleyelim. Bunun için `etc/fields.yml` dosyasını değiştiriyoruz.

```
- key: lsbeat
  title: LS Beat
  description: 
  fields:
    - name: counter
      type: integer
      required: true
      description: >
        PLEASE UPDATE DOCUMENTATION
    #new fiels added lsbeat
    - name: modtime
      type: date
    - name: filename
      type: text
    - name: path
    - name: directory
      type: boolean
    - name: filesize
      type: long
```

Ve güncellemeli onaylayalım. 

```
$ make update
```

`filename` alanı nGram tokenizer ile analiz ediliyor olacak. Bunun için 
`lsbeat.template.json` dosyasında "settings" altına bir analyzer ekleyelim. 

```
{
  "mappings": {
        ...
  },
  "order": 0,
  "settings": {
    "index.refresh_interval": "5s",
    "analysis": {
      "analyzer": {
        "ls_ngram_analyzer": {
          "tokenizer": "ls_ngram_tokenizer"
        }
      },
      "tokenizer": {
        "ls_ngram_tokenizer": {
          "type": "ngram",
          "min_gram": "2",
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  },
  "template": "lsbeat-*"
}
```

### Adım 5 - Kurulum ve Çalıştırma

Şimdi kurup çalıştırabiliriz. Sadece make komutunu çalıştırın ve kodlar 
derlenecektir, `lsbeat` (windows için `lsbeat.exe`) binary dosyası 
çalıştırılabilir olarak oluşacaktır.

```
$ make
```

Ana bir klasör belirtmek için `lsbeat.yml` dosyasını düzenleyin. Bizim 
senaryomuzda `$GOPATH` dizinini kullanıyoruz. Tam adresi verdiğinizden emin olun.

```
lsbeat:
  # Defines how often an event is sent to the output
  period: 10s
  path: "/Users/ec2-user/go"
```

Ve ayrıca Elasticsearch ve Kibana arayüzlerinizin çalıştığından emin olun. Şimdi
Lsbeat'i çalıştıralım ve ne olacağını görelim.

```
$ ./lsbeat
```

`_cat` arayüzünü kullanarak oluşan index'lere göz atabilirsiniz. 

![](https://img.readitlater.com/i/www.elastic.co/assets/blt47d7e8421c87b1b7/cat/RS/w1408.png?&ssl=1)

Yeni oluşan lsbeat-2016.06.03 index'ini ve döküman sayısını görebilirsiniz. 
nGram ile analiz edilmiş "filename" alanını kullanarak bir arama çalıştıralım.

![](https://www.elastic.co/assets/blte53a76e132c239b6/query.png)

Çalıştı! Tebrikler.


_Kaynak :_ [https://www.elastic.co/blog/build-your-own-beat](https://www.elastic.co/blog/build-your-own-beat)

Not : Örnek oluşturabilecek diğer bir beat olarak 
[filecountbeat](https://github.com/hkulekci/filecountbeat)'i de örnek 
alabilirsiniz. Belirtilen dizindeki dosyaların ve klasörlerin sayılarını log 
olarak tutan bir beat örneğidir. 