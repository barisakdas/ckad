# RESOURCE REQUIREMENTS NEDİR?

Bir kubernetes cluster oluşturduğunuzda içerisindeki her bir node için kendi içerisinde CPU ve Memory kaynakları vardır.
Bu kaynaklar cluster kurulurken ayarlanır ve daha sonrasında değiştirilebilir.

Her bir pod çalışabilmesi için bir dizi kaynağa ihtiyaç duyar. Uygulama bu ihtiyaçlarını içinde bulunduğu node'un kaynaklarını kullanarak karşılar.
Bir node içerisindeki kaynaklar tamamen tüketildiğinde veya yeni gelecek bir pod için yeterli kaynak kalmadığında bu podun
başka bir node içerisinde ayağa kaldırılması gereklidir. 

Kubernetes içerisinde bulunan `kubes-cheduler` yapısı bir podun hangi node üzerinde ayağa kaldırılacağını belirlemekten sorumludur.
Kube-Scheduler yeni gelen podun ihtiyaçlarını ve mevcut nodeların kaynak durumlarını göz önüne alarak pod için en iyi node'u seçmeye çalışır.
Bunu yaparken seçeceği node'un yeterli kaynağının olduğundan emin olur.

Eğer ki elindeki hiç bir node'un yeterli kaynağı yoksa podu ayağa kaldıramayacak ve beklemeye alacaktır. Bu durumda podun statüsü `pending` olarak atanacak ve kaynakların müsait olmasını bekleyecektir. Podun eventlarının içerisine girdiğinizde size yeterli kaynak olmadığından ötürü beklemede olduğunu söyleyecektir.

## Podların kaynak değerleri nasıl ayarlanır?
Her bir uygulama kendi içerisinde belirli bir kaynak tüketme potansiyeline sahiptir. Örneğin 1 CPU ya da 1 gigabyte memory tüketmesi gibi.
Pod oluşurken bu değerleri belirtebilir ve uygulamanın ihtiyaçlarını gösterebiliriz. Bu container'ın talep ettiği kaynak değerlerini gösterir.
Dolayısıyla kube-scheduler burada talep edilen kaynağı esas alarak node seçimini yapar.
Pod oluşurken yaml dosyası içerisindeki `containers` alanının altına ekleyeceğimiz `resources` alanında talep ettiğimiz kaynakları belirtebiliriz.

```yaml
spec:
  container: 
  - name: sample-app
    image: sample-app
    ports:
      - containerPort: 8080
    resources:
      request:
        memory: "1Gi"
        cpu: 1
```

Yukarıdaki örnekte oluşturulacak olan sample-app podu için `1 gibibyte memory` ve `1 cpu` kaynaklarının talep edildiğini görebiliriz.
Kube-scheduler bu talep edilen kaynaklara göre bir node seçerek uygulamayı ayağa kaldıracak ve podun bu kaynaklara erişebileceğinin garantisini sağlamış olacaktır.

## CPU ve Memory değerleri neyi işaret ediyor?
Bir pod için belirlenen cpu değeri 0 dan büyük olacak şekilde istenildiği gibi ayarlanabilir. Örneğin 0.1 cpu değeri olarak ayarlanan kaynak talebi 100m olacak şekilde ayarlanır. Buradaki m `mili` anlamına gelir ve bir cpu kaynağı bu değerin altına inemez.
Cpu değeri için 1 girilmesi aslında 1 vCPU'ya eşittir. Yani eğer ki siz bir AWS üzerinde kullanıyorsanız onun ayarladığı 1 cpu değeri olacaktır.
Cpu için bir üst limit olmamakla beraber eğer finanse edebiliyorsanız bunu daha fazla arttırabilirsiniz. Ama burada asıl önemli olan uygulamanın harcadığı cpu değerlerini ölçmek ve gerçeğe uygun olarak bir değer girmektir. En yoğun anında 0.1 cpu değer kullanan bir uygulama için 5 cpu luk bir talepte bulunabilirsiniz. Ama bu size gereksiz yere maliyet doğuracaktır. Bunun gibi onlarca uygulama kurmanız gerektiğinde maliyetler katlanarak artacaktır. 
Sonuç olarak uygulama ayağa kalktığında metriklerini izlemeli ve en doğru değerleri talep etmelisiniz. 

Pod için kullanılacak olan memory değerleri de cpu gibi ayarlanabilir. Burada yine en önemli olan konu uygulamanın ne kadar memory ihtiyacı olduğu veya olabileceği konusudur.
Uygulama için `1G` veya `1Gi` ifadeleri kullanarak değerler tanımlayabiliriz. Ama buradaki iki ifade bir birinden farklıdır.
1G olarak tanımladığımızda bunun anlamı `1 Gigabyte` kullanmak istediğinizdir. Eğer ki 1Gi olarak tanımlarsanız da `1 Gibibyte` kullanacağınız anlamına gelir.
Bu iki ifadenin arasındaki farkı şu şekilde gösterebiliriz.

`1G : 1.000.000.000 bytes`
`1Gi : 1.073.741.824 bytes`

## Uygulamanın Kaynak Sınırlaması
Bir uygulama bir node üzerinde ayağa kalktığında belli bir kaynak onun için ayrılır. Ama bu ayrılma o uygulamanın daha fazla kaynak tüketemeyeceği anlamına gelmez.
Kubernetes üzerinde uygulamanın kaynak tüketimi sınırsızdır. Yani bir uygulama talep edilenin üstünde bir kaynak harcamaya başladığında üzerinde bulunduğu node'un kaynaklarını kullanmaya başlar. Ta ki node'un kaynakları tükenene kadar. Node üzerindeki kaynaklar tükendiğinde diğer uygulamalar ve yerek işlemler  artık daha fazla kaynak kullanamaz ve sorun yaşamaya başlar.

Böyle bir durumla karşılaşmamak için her bir container için kaynak kullanımlarını sınırlayabiliriz. Örneğin bir container için 1 cpu luk bir limit eklediğinizde ilgili container daha fazla cpu kaynağı tüketemez.

```yaml
spec:
  containers: 
  - name: sample-app
    image: sample-app
    ports:
      - containerPort: 8080
    resources:
      request:
        memory: "1Gi"
        cpu: "1"
      limits:
        memory: "2Gi"
        cpu: "2"
```

Yukarıdaki örnekte sample-app uygulamasının kaynak tüketimleri 2cpu ve 2Gi memory olarak ayarlanmıştır. Bu durumda uygulama bu değerleri geçemeyecektir. Buradaki limitler sadece bağlı olduğu container'a eklenir. Eğer ki pod içerisinde birden fazla container alıyorsa diğer container içinde değerler eklenmelidir.

## Pod belirlenen kaynakları aşmaya çalıştığında ne olur?
Söz konusu CPU olduğunda sistem sınırın ötesine geçemeyecek şekilde kısıtlar. Bir container verilen limitten daha fazla cpu kullanamaz. 

Ancak memory için durum biraz daha farklıdır.
Bir container belirlenen limit değerinden daha fazla memory kullanabilir. Dolayısıyla bir pod sürekli limitinden daha fazla memory tüketmeye çalışırsa podun yaşam döngüsü sonlanacaktır. Bu durumda podun eventlerinde ya da loglarında `OOM (Out Of Memory)` hatası ile `terminate` durumuna geçtiğini görebilirsiniz.

## Kubernetes üzerinde CPU yapılandırması nasıl yapılır?
Kubernetes üzerinde ayağa kaldırılacak bir container için herhangi bir limit ve talep değeri girmezseniz, yani default olarak bırakırsanız bu değerler sınırsız olarak kabul edilir. Yani bu, herhangi bir podun herhangi bir düğümde gerektiği kadar kaynak tüketebileceği ve üzerinde çalıştığı node'un diğer podlarını veya local işlemleri boğabileceği anlamına gelir.

Bu durumu göz önünde bulundurursak CPU kısıtlama ve taleperinde şu senaryolar ile ilerleyebiliriz.

1) NO REQUEST - NO LIMITS: Bu durumda bir pod istediği kadar cpu kaynağı tüketebilir ve diğer podlar için sorun olmaya başlayabilir. Bu senaryo çoğu zaman önerilmez.

2) NO REQUEST - LIMITS: Herhangi bir request değeri belirtilmeden sadece limit belirtildiğinde kubernetes buradaki limit değerlerini request değerleri olarak kullanır. Yani siz 0.1cpu limit cpu değer verdiğinizde ve request vermediğinizde kubernetes sizin yerinize limit için tanımladığınız 0.1cpu değerini request olarak tanımlar.

3) REQUEST - LIMITS: Bir container için hem request hemde limit tanımlamaları yaptığınızda ilk olarak uygulamanız için ayarlanan request değerleri baz alınarak uygun node seçilir ve limit değerlerinde izin verilen kaynaklara kadar kullanım hakkı sunulur. Bu durum ideal bir senaryo gibi görünse de bir uygulama cpu ihtiyacı duyduğunda ve onu sınırladığımızda onun ihtiyacına cevap verememiş oluruz. Ve eğer ki diğer uygulamalar daha fazla cpu tüketmiyorsa boşu boşuna ilgili uygulamayı kısıtlamış oluruz. Bunu çok fazla istemeyiz. Eğer ki sistemde yeterli miktarda cpu kullanım şansı varsa bunu değerlendirmek isteriz.

4) REQUEST - NO LIMITS: Bir uygulama ayağa kaldırılacağı zaman request tanımlaması ile birlikte bir cpu kaynak kullanım garantisi verilir. Ve bu container sınırları içermediğinden istediği kadar cpu tüketebilir. Bu durumda uygulama daha rahat ve sorunsuz olarak çalışacaktır. Bu CPU için daha çok önerilen bir durumdur.

## Kubernetes üzerinde memory yapılandırması nasıl yapılır?
CPU için bahsettiğimiz senaryolar memory kullanımları içinde geçerlidir.

1) NO REQUEST - NO LIMITS: Bu durumda bir pod istediği kadar memory kaynağı tüketebilir ve diğer podlar için sorun olmaya başlayabilir. Bu senaryo çoğu zaman önerilmez.

2) NO REQUEST - LIMITS: Herhangi bir request değeri belirtilmeden sadece limit belirtildiğinde kubernetes buradaki limit değerlerini request değerleri olarak kullanır. Yani siz 1Gi bir limit memory değer verdiğinizde ve request vermediğinizde kubernetes sizin yerinize limit için tanımladığınız 1Gi değerini request olarak tanımlar.

3) REQUEST - LIMITS: Bir container için hem request hemde limit tanımlamaları yaptığınızda ilk olarak uygulamanız için ayarlanan request değerleri baz alınarak uygun node seçilir ve limit değerlerinde izin verilen kaynaklara kadar kullanım hakkı sunulur. Bu durum ideal bir senaryo gibi görünse de bir uygulama cpu ihtiyacı duyduğunda ve onu sınırladığımızda onun ihtiyacına cevap verememiş oluruz. Ve eğer ki diğer uygulamalar daha fazla cpu tüketmiyorsa boşu boşuna ilgili uygulamayı kısıtlamış oluruz. Bunu çok fazla istemeyiz. Eğer ki sistemde yeterli miktarda cpu kullanım şansı varsa bunu değerlendirmek isteriz.

4) REQUEST - NO LIMITS: Bir uygulama ayağa kaldırılacağı zaman request tanımlaması ile birlikte bir cpu kaynak kullanım garantisi verilir. Ve bu container sınırları içermediğinden istediği kadar cpu tüketebilir. Bu durumda uygulama daha rahat ve sorunsuz olarak çalışacaktır. Bu CPU için daha çok önerilen bir durumdur.

## Oluşturulan her pod varsayılan ayarlara nasıl sahip olur?
Kubernetes üzerinde tanımlanabilen `LimitRange` componenti ile limit aralıklarının belirlenmesi mümkündür. Bu yapılandırma komponenti namespace bazında geçerlidir. Yani bir namespace içerisinde tanımlanan limitrange komponenti diğer namespace içindeki containerleri etkilemez.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 500m
    max:
      cpu: "1"
    min:
      cpu : 100m
    type: Container
```

Yukarıdaki örnekte olduğu gibi bir `LimitRange` komponenti oluşturabiliriz. Burada default değerleri, max ve min değerleri belirleyebiliriz. Hatırlatmak isterim ki burada girilen değerler hayalidir, yani siz kendi uygulamanız için gerçek değerleri belirlemelisiniz. CPU için yapılan bu örneğin benzerini memory içinde yapabiliriz.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory : 500mi
    type: Container
```

Bir limitrange komponenti oluturduğunuzda bu mevcutta çalışan podları etkilemez. Yeni gelecek podlar için tanımlamalar alınır.

## Kubernetes üzerindeki namespace için limit tanımlaması nasıl yapılır?
Bir namespace içerisindeki uygulamaların toplam tüketebileceği kaynakları kısıtlamak için kubernetes üzerinde bir `ResourceQuota` komponenti mevcuttur. Bu komponentin amacı namespace içerisindeki tüm podlar toplam olarak belirlenen CPU veya memory kullanımı olacak şekilde sınırlandırılabilir. 

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

ResourceQuota komponenti kubernetesdeki kaynak sınırlama olarak kullanılabilecek başka bir yöntem olarak burada bulunabilir.