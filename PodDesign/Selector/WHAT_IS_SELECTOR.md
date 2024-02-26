Kubernetes'te Labellar (Labels) ve Selectorler (Selectors)
Kubernetes, konteynerleri yönetmek için kullanılan güçlü bir platformdur. Labellar ve Selectorler, Kubernetes'te nesneleri organize etmek ve yönetmek için kullanılan iki önemli kavramdır.

Labellar:
* Key-value (anahtar-değer) çiftlerinden oluşan bir dizi.
* Nesneleri tanımlamak ve kategorize etmek için kullanılır.
* Herhangi bir Kubernetes nesnesine (pod'lar, servisler, dağıtımlar vb.) eklenebilir.
* Sınırsız sayıda Label kullanılabilir.

Selectorler:
Labellare dayalı olarak nesneleri seçmek için kullanılan ifadelerdir.
Pod'ları belirli bir hizmete bağlamak, belirli bir ortamda çalışan pod'ları seçmek gibi çeşitli görevler için kullanılır.
İki tür Selector vardır: eşitlik tabanlı ve küme tabanlı.
Labellar ve Selectorlerin Kullanım Alanları:

* Pod'ları Hizmetlere Bağlama: Pod'lara Labellar eklenerek ve servisler için Selectorler tanımlanarak belirli pod'ların belirli servislere bağlanması sağlanabilir.
* Ortamlar Arası Dağıtım: Farklı ortamlar (geliştirme, test, üretim) için farklı Labellar ve Selectorler kullanılarak dağıtımlar kontrol edilebilir.
* Yük Dengeleme: Pod'lara Labellar eklenerek ve yük dengeleyiciler için Selectorler tanımlanarak trafik belirli pod'lara yönlendirilebilir.
* Otomatik Ölçeklendirme: Pod'lara Labellar eklenerek ve otomatik ölçeklendirme grupları için Selectorler tanımlanarak belirli koşullara göre pod'ların sayısı otomatik olarak artırılabilir veya azaltılabilir.
* Hata Toleransı: Pod'lara Labellar eklenerek ve hata toleransı politikaları için Selectorler tanımlanarak belirli pod'ların arızalanması durumunda otomatik olarak yeniden başlatılması sağlanabilir.

Label ve Selector Örnekleri:
Label:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
  labels:
    app: web-app
    tier: frontend
spec:
  containers:
  - name: web-app
    image: nginx
    ports:
    - containerPort: 80
```

Selector:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  labels:
    app: web-app
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```


Hangi Uygulama Neye Göre Label ve Selector Kullanır?

Her uygulama farklı Labellar ve Selectorler kullanabilir. Genellikle, uygulamanın gereksinimlerine göre Labellar ve Selectorler belirlenir.

Örnek:
* Bir web uygulaması, ön uç ve arka uç pod'larını ayırt etmek için "app" ve "tier" Labellarini kullanabilir.
* Bir mikro hizmetler mimarisi, her bir hizmeti tanımlamak için "service" Labelini kullanabilir.
* Bir CI/CD sistemi, farklı ortamları (geliştirme, test, üretim) tanımlamak için "environment" Labelini kullanabilir.

Sonuç:
Labellar ve Selectorler, Kubernetes'te nesneleri organize etmek ve yönetmek için güçlü araçlardır. Doğru şekilde kullanıldığında, Labellar ve Selectorler, uygulamaların daha kolay ve daha verimli bir şekilde dağıtılmasına ve yönetilmesine yardımcı olabilir.