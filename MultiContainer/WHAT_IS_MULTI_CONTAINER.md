# MULTI CONTAINER NEDİR?

Kubernetes dünyasındaki en küçük yapı olarak POD'lar bilinir. Pod, uygulamanın bir örneğinin çalışması için hazırlanan ve tüm bileşenlere sahip en küçük komponenttir.

Mikroservise mimarisi gibi mimarilerde birden fazla büyük veya küçük uygulamalar inşa etmeniz gerekebilir. Çoğu zaman her bir uygulama bir podda tek başına çalışıyor olsa bile bazı durumlarda birden fazla uygulamanın aynı pod içerisinde olması gerekebilir.

Mikroservise mimarisinde uygulamaların herbiri diğerlerinden bağımsız olarak ölçeklendirilebilir veya farklı ayarlamalara sahip olabilir. Ancak bazı durumlarda iki ya da daha fazla uygulamanın aynı anda yani birlikte çalışmasına ihtiyaç duyabiliriz. Birlikte çalışmalarına rağmen bu iki uygulamanın kodlarını birleştirmek ve biri patladığında diğerininde problem yaşamasını istemeyebiliriz. Bu durumda iki uygulama bir birinden bağımsız olarak geliştirilmelidir.

Peki birbirinden bağımsız olarak geliştirilen ancak beraber çalışmak zorunda olan hatta aynı şekilde ölçeklenmek zorunda olan bu iki uygulamayı nasıl beraber olmadan beraber çalıştırabiliriz.

Kubernetes dünyasında podlar çoklu konteynır yapısı ile kurulabilir. Yani bir podun içerisinde birden fazla konteynır yaratmanız mümkündür. Ölçekleme ve konfigurasyon işlemleri pod seviyesinde olacağından iki uygulama içinde aynı olacaktır. Aynı poddaki bu iki uygulama birlikte yaratılıp birlikte ölürler. Aynı network ağını paylaşır hatta aynı depolama birimine erişirler. Aralarında iletişim sağlamak için podlar arasında iletişim kurmanız veya servis hizmeti oluşturmanız gerekmez. Çoklu konteynıra sahip bir pod oluşturmak için podun hazırlandığı yaml dosyasında yeni bir konteynır eklemek yeterlidir.

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: multi-container-pod
    labels:
      name: multi-container-pod
spec:
  containers:
  - name: sample-web-app
    image: sample-web-app
    ports:
      - containerPort: 8080
  - name: log-app
    image: log-app
```

Pod tanım dosyasında `spec:containers` alanı bir liste yapısındadır. Bu da birden fazla konteynır ekleyebileceğiniz anlamına gelir. Yukarıdaki örnekte `sample-web-app` isminde bir uygulama için bir pod tasarladık. Ama aynı pod içerisinde çalışmasını istediğimiz `log-app` uygulamasını da konteynır olarak ilgili alana ekledik. Bu sayede pod ayağa kalktığında içerisindeki iki konteynır da ayağa kalkacaktır.

Çoğu zaman bu örnek gerçek hayatta bu şekilde kullanılmıyor. Çoklu konteynır yapısı kullanılacağı zaman genellikle 3 tane yaygın kullanılan model mevcuttur.
* Sidecar Pattern
* Adapter Pattern
* Ambassador Pattern

## Sidecar Pattern nedir?
Çoklu konteynır yapısından bahsederken, Sidecar Pattern'den bahsetmek de önemlidir. Bu, çoklu konteynır yapısını kullanan bir mimari desendir ve bir ana uygulamaya ek özellikler sağlamak için yan tarafta çalışan başka bir konteynır kullanır.

Sidecar Desen Örneğinin Temel Özellikleri:

* Ana Uygulama ve Yardımcı Uygulama: Bu desen, bir ana uygulama ve ona ek özellikler sağlayan bir yardımcı uygulama olmak üzere iki konteynırdan oluşur. Yardımcı uygulama, "sidecar" olarak adlandırılır, çünkü ana uygulamaya benzer şekilde yan tarafta çalışır ve onu destekler.

* Yardımcı Uygulama Bağımsızdır: Yardımcı uygulama, ana uygulamadan tamamen bağımsızdır. Bu, farklı dillerde yazılabileceği, farklı bağımlılıklara sahip olabileceği ve hatta farklı ekipler tarafından geliştirilebileceği anlamına gelir.

* Destekleyici İşlevler Sağlar: Yardımcı uygulama, ana uygulamaya çeşitli destekleyici işlevler sağlayabilir. Bu işlevler şunları içerebilir:
    * Logging: Yardımcı uygulama, ana uygulamanın kütüklerini toplayabilir ve analiz edebilir.
    * İzleme: Yardımcı uygulama, ana uygulamanın performansını izleyebilir ve sorunları tespit edebilir.
    * Güvenlik: Yardımcı uygulama, ana uygulamayı güvenlik tehditlerine karşı koruyabilir.
    * Yük Dengeleme: Yardımcı uygulama, gelen trafiği farklı konteynırlara dağıtabilir.
    * Gecikme Azaltma: Yardımcı uygulama, istekleri önbelleğe alarak gecikmeyi azaltabilir.

Sidecar Desen Örneğinin Avantajları:
* Modülerlik: Yardımcı uygulama, ana uygulamadan bağımsız olduğu için, her ikisini de ayrı ayrı geliştirmek ve dağıtmak mümkündür.
* Ölçeklenebilirlik: Yardımcı uygulamalar, ana uygulamalardan bağımsız olarak ölçeklendirilebilir.
* Esneklik: Yardımcı uygulamalar, çeşitli işlevler sağlamak için kullanılabilir, bu da uygulamayı daha esnek hale getirir.
* Yalıtım: Yardımcı uygulama, ana uygulamadan yalıtılmıştır, bu da bir uygulamada bir sorun diğerini etkilemez.

Sidecar Desen Örneğinin Kullanım Alanları:
* Mikro hizmetler: Sidecar desen örneği, mikro hizmet mimarisinde sıklıkla kullanılır. Her mikro hizmet, kendi yardımcı uygulamasına sahip olabilir.
* DevOps: Yardımcı uygulamalar, izleme ve günlük kaydı gibi DevOps süreçlerini otomatikleştirmek için kullanılabilir.
* Yapay zeka: Yardımcı uygulamalar, yapay zeka modelleri eğitmek ve çalıştırmak için kullanılabilir.

Sidecar Desen Örneği, çoklu konteynır yapısını kullanan güçlü bir mimari desendir. Modülerlik, ölçeklenebilirlik, esneklik ve yalıtım gibi avantajları sayesinde çeşitli uygulamalar için ideal bir seçimdir.

## Adapter Pattern nedir?

Adapter pattern, birbiriyle uyumsuz olan iki arayüzü birbirine bağlamak için kullanılan bir tasarım desenidir. Bu desen, bir adaptör nesnesi oluşturarak iki arayüz arasındaki iletişimi sağlar.

Daha iyi açıklamak için şöyle örnekler verebiliriz.
* Bir elektrikli ev aletiniz var ve bu aletin fişi Avrupa standardında. Fakat siz priziniz Amerikan standardında olduğu bir evde yaşıyorsunuz. Bu durumda, cihazınızı prize takmak için bir adaptöre ihtiyacınız olacaktır. Adapter pattern'de de durum buna benzer. İki farklı arayüzümüz var ve bu arayüzler birbiriyle uyumlu değil. Bir adaptör nesnesi oluşturarak bu iki arayüz arasındaki iletişimi sağlayabiliriz.

Başka bir senaryoda şu şekilde olabilir:
* Diyelim ki bir e-ticaret web siteniz var ve bu web sitesinde ürünlerinizi satıyorsunuz. Ürünlerinizin bilgilerini bir XML dosyasında saklıyorsunuz. Fakat web siteniz, ürün bilgilerini JSON formatında bekliyor. Bu durumda, XML dosyanızdaki bilgileri JSON formatına dönüştürmek için bir adaptöre ihtiyacınız olacaktır.

Adapter nesnesi, XML dosyasını okur ve bilgileri JSON formatına dönüştürür. Bu sayede, web siteniz ürün bilgilerini JSON formatında alır ve kullanabilir.

Adapter Pattern'in Temel Özellikleri:

* Uyumluluk Sağlar: Adapter pattern, iki uyumsuz arayüzü birbirine bağlayarak uyumluluk sağlar. Bu, farklı kütüphaneleri, çerçeveleri veya sistemleri birlikte kullanmak için ideal bir çözüm sunar.
* Esneklik: Adapter pattern, uygulamanıza yeni işlevler eklemek için esneklik sağlar. Mevcut kodunuzu değiştirmeden yeni arayüzleri entegre etmek için bir adaptör nesnesi oluşturabilirsiniz.
* Bakımı Kolay: Adapter pattern, kodunuzu daha modüler ve bakımı kolay hale getirir. Her arayüzü kendi adaptör nesnesine ayırarak kodunuzu daha organize hale getirebilirsiniz.

Multi Konteyner Yapısında Adapter Pattern:
Multi konteyner yapısında, adapter pattern, farklı konteynerler arasında iletişim kurmak için kullanılabilir. Örneğin, bir konteyner veritabanına erişmek için bir API sağlarken, başka bir konteyner bu API'yi kullanarak veritabanıyla etkileşime girebilir.

Adapter Pattern'in Avantajları:
* Uyumluluk: Farklı arayüzleri birbirine bağlama imkanı sunar.
* Esneklik: Yeni işlevler eklemek için esneklik sağlar.
* Bakımı Kolay: Kodun daha modüler ve bakımı kolay hale gelmesini sağlar.
* Yeniden Kullanılabilirlik: Farklı senaryolarda tekrar kullanılabilir.

Adapter Pattern'in Kullanım Alanları:
* Farklı kütüphaneleri veya çerçeveleri birlikte kullanmak
* Yeni işlevler eklemek
* Mevcut kodunuzu değiştirmeden yeni arayüzleri entegre etmek
* Farklı konteynerler arasında iletişim kurmak

Adapter pattern, multi konteyner yapısında çok kullanışlı bir tasarım desenidir. Uyumluluk, esneklik ve bakımı kolaylık gibi avantajları sayesinde, farklı konteynerler arasında iletişim kurmak için ideal bir çözüm sunar.

## Ambassador Pattern nedir?

Ambassador pattern, mikro hizmetler gibi dağıtık sistemlerde sıklıkla kullanılan bir mimari deseni olarak karşımıza çıkar. Bu desen, bir uygulamanın dış dünyaya açılan kapısını bir aracı hizmet aracılığıyla yöneterek çeşitli faydalar sunar.

Ambassador Pattern'in Temelleri:

* Aracı Hizmet: Bu desenin temel unsuru, ana uygulama ile dış dünya arasındaki iletişimi yöneten bir aracı hizmettir. Bu hizmet, bir ayrı konteynerde çalışır ve ana uygulamaya "elçi" gibi davranır.
* Ayrıcalıklar: Aracı hizmet, ana uygulamadan daha fazla ayrıcalığa sahip olabilir. Örneğin, güvenlik duvarını yönetme, trafiği dengeleme, protokol çevirisi yapma gibi yetkilere sahip olabilir.
* Esneklik: Aracı hizmet sayesinde, ana uygulama dış dünyaya açılan kapısını değiştirmeden farklı özellikler ekleyebilirsiniz. Örneğin, yeni bir protokol desteği eklemek için sadece aracı hizmeti güncellemeniz yeterlidir.

Ambassador Pattern'in Faydaları:
* Güvenlik: Aracı hizmetin daha sıkı güvenlik politikalarıyla yönetilmesi, ana uygulamaya daha fazla koruma sağlar.
* Ölçeklenebilirlik: Trafiği dengeleme ve yük dağıtımı gibi yetkileri aracı hizmete vererek ana uygulamanın performansını artırabilirsiniz.
* Yönetim Kolaylığı: Aracı hizmeti, dış dünyaya açılan kapıyı tek bir noktada yönetmenizi sağlar.
* Esneklik: Yeni özellikler eklemek ve değişiklikleri uygulamak kolaylaşır.

Multi Konteyner Yapısında Ambassador Pattern:
Multi konteyner yapısında, Ambassador pattern her konteynerin ayrı bir aracı hizmete sahip olmasıyla uygulanabilir. Bu sayede her konteynerin güvenliği, ölçeklenebilirliği ve yönetimi daha kolay hale gelir.

Ambassador Pattern'in Kullanım Alanları:
* Mikro hizmetler mimarisinde dış dünyaya açılan kapıyı yönetmek
* Farklı protokollerle iletişim kurmak
* Trafiği dengelemek ve yük dağıtımı yapmaks
* Güvenliği artırmak
* Yeni özellikler eklemek ve değişiklikleri uygulamak

Daha iyi anlaşılması için bir örnek:
Bir e-ticaret uygulamasında, mikro hizmetlere ayrılmış bir yapı düşünelim. Ödeme, ürün yönetimi ve kullanıcı yönetimi gibi farklı hizmetler ayrı konteynerlerde çalışıyor olsun. Bu durumda, her hizmet için bir Ambassador konteyneri kullanabilirsiniz. Ambassador konteynerleri, dış dünyadan gelen istekleri doğru hizmete yönlendirerek güvenliği artırabilir, trafiği dengeleyebilir ve yeni özellikler eklemeyi kolaylaştırabilir.