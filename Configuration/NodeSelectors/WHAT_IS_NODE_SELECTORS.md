# NODE SELECTOR NEDİR?

Kubernetes üzerinde hazırlanmış bir clustera sahip olduğumuzu düşünelim. Cluster içerisinde birden fazla node olabilir ve bu node lar birbirinden daha farklı boyutlarda seçilebilir. Örneğin iki tane orta büyüklükte bir tane de daha büyük node'a sahip olduğumuzu varsayalım. Bu durumda daha fazla kaynak tüketmesini beklediğimiz uygulamanın büyük node içerinde deploy olmasını isteriz. Ama kubernetes içerisindeki kubec-scheduler yapısı -eğer ki siz bir detay belirtmezseniz- uygulama deploy olacağı zaman en uygun node içerisinde deploy edecektir. Bu node bizim elimizdeki diğer iki node'dan birisi olabilir ve bu pek de istediğimiz bir durum olmaz.

Peki ben herhangi bir uygulama için daha deploy aşamasında hangi node üzerinde kurulacağını belirtseydim nasıl olurdu?.
Bu defa kube-schedule bize diğer node'lar içerisinde bir yer ayarlamak yerine istediğimiz -işaretlediğimiz- node içerisinde uygulamamızı hazırlardı.
Bu yapıyı kubernetes dünyasında NodeSelector kullanarak yapmak mümkün. Bunu yapabilmek için podun deploy edileceği yaml dosyasında `spec:nodeSelector` alanı ekleyerek işaretlemek yeterli olacaktır. Burada selector yapısının ilgili node'u tanıması için `key: value` değer çiftlerini kullanacağız. Tabi bu değer çiftlerinin node içinde tanımlı olması gerekecektir. Böylece uygulama deploy edilirken deployment tanımında (aşağıdaki örnekte olduğu gibi kullanılırsa yaml dosyasında) `nodeSelector` alanı var ise o alanda belirtilen `key/value` çiftine sahip node var mı diye bakacaktır. Eğer ki bu değerlere sahip birden fazla node varsa kube-schedule artık bu iki node üzerine dağıtımlarını yapacaktır. Ama hiç bir node içerisinde bu değer çiftleri yoksa bu sefer uygulamamız bu dağıtımda yerleşemeyeceği için `Pending` adımında kalacak ve deploy olamayacaktır.

```yaml
spec:
  container: 
  - name: sample-app
    image: sample-app
  nodeSelector: 
    size: Large
```

Yukarıdaki yaml bloğunda göreceğiniz gibi ilgili pod için bir nodeSelector ataması yapılmıştır. Bu atama yapılırken kullanılan anahtar değer `size` ve ona karşılık atanan değer ise `Large` olarak tanımlanmıştır. Burada kesinlikle bir büyüklük algılama söz konusu değildir. Yani kubernetes dağıtıcıları buradaki `Large` değerine bakarak büyük bir node arama işlemine başlamayacaktır. Bunun tek amacı bu değer çiftine sahip node olup olmadığını bulmaktır. O yüzden burada yazılan hiç bir değerin ifade ettiği anlamları dikkate almamak gerekir.

Tabi ki de bu pod oluşmadan önce sahip olunan node'lar istenilen değerlere göre işaretlenmelidir. Örneğin kullandığımız değer çiftine sahip bir node bulundurmalıyız ki uygulama dağıtım sırasında açıkta kalmasın.

Bir node'u etiketlemek için `label` özelliğini kullanırız. Örneğin bir işaretleme yapmak için terminal ekranında şu şekilde bir komut hazırlanabilir: `kubectl label nodes {node-name} {label_key}={label_value}`

Bu komutun bizim podumuza uygun halini şu şekilde görebiliriz.

```bash
kubectl label nodes node-1 size=Large
```

Artık node işaretleme ve pod içerise seçici atandığında göre uygulama dağıtım esnasında işaretlenen node'u seçerek oraya kurulacaktır. 
Teşekkürler kube-schedule :)

## Dağıtım sırasında daha karmaşık bir etiketleme sistemine ihtiyacımız olursa ne yapmalıyız?

NodeSelector bu yukarıda istediğimiz amaç için bize destek olabilir. Uygulama dağıtılırken tek bir şart varsa buna bakarak node seçebilir.
Ama gerçek hayatta senaryolar daha karmaşık olabiliyor. Örneğin tek bir etiketin yetmediği durumlarda bizim yinede dağıtımı kısıtlamamız gerekebilir.
Bir örnek senaryoyu kafanızda canlanması için şöyle anlatayım: 
Bizim uygulamamız gerekçeli sebeplerden ötürü daha büyük bir node içerisine dağıtılmalıydı ve bizde node'larımızı Large/Medium/Small gibi etiketler ile işaretledik.
Ama yeni bir uygulama geldi ve bize bu uygulama için küçük olmayan yani büyük veya orta bir yere aktarılabileceğimiz söylendi. Bu durumda bizim nodeSelector yapımız yetersiz kalacaktır çünkü nodeSelector'un sınırları var.

Böyle bir senaryo için kubernetes üzerinde `Node-Affinity` ve `Anti-Affinity` kavramları mevcut. O kısımlarda ihtiyaçlarınıza karşılık bulabilirsiniz.