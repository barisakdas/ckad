# NODE AFFINITY NEDİR?

Kubernetes üzerinde bulunan NodeSelector yapısının yetersiz kaldığı senaryolar yaşayabiliriz. 

Örneğin tek bir etiketin yetmediği durumlarda bizim yinede dağıtımı kısıtlamamız gerekebilir.
Bir örnek senaryoyu kafanızda canlanması için şöyle anlatayım: 
Bizim uygulamamız gerekçeli sebeplerden ötürü daha büyük bir node içerisine dağıtılmalıydı ve bizde node'larımızı Large/Medium/Small gibi etiketler ile işaretledik.
Ama yeni bir uygulama geldi ve bize bu uygulama için küçük olmayan yani büyük veya orta bir yere aktarılabileceğimiz söylendi. Bu durumda bizim nodeSelector yapımız yetersiz kalacaktır çünkü nodeSelector'un sınırları var.

Böyle bir senaryo için kubernetes üzerinde `Node-Affinity` ve `Anti-Affinity` kavramları mevcut. 

Node Affinity özelliği, asıl amacı uygulama parçalarının belirli node'lar üzerinde dağılımını sağlamak olan bir kubernetes yapısıdır.
Bu yapı uygulamaların dağıtımı sırasında seçebileceğimiz node'lar için bize daha karmaşık senaryoları yönetme şansı verir.

Bu yapıyı kurmadan önce bilmemiz gereken bazı kavramları açıklayalım.

## Node Affinity tipleri nelerdir?
Node affinity, bir Pod'un hangi düğümlerde çalıştırılabileceğini kontrol etmenizi sağlar. Bu, Pod'ların belirli gereksinimleri karşılayan düğümlerde çalışmasını ve kaynakların daha verimli kullanılmasını sağlar.

Node affinity'nin üç tipi vardır:

1. Required: Bu tip, Pod'un çalıştırılabilmesi için en az bir düğümün bu koşulu karşılaması gerektiğini belirtir. Birden fazla koşul belirtilebilir ve hepsinin karşılanması gerekir.

```yaml
spec:
  container: 
  - name: sample-app
    image: sample-app
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: "size"
          operator: In
          values:
          - "Large"
          - "Medium"
```
Bu örnekte, Pod yalnızca Large veya Medium olarak işaretlenmiş node'lar üzerinde çalıştırılabilir.

2. Preferred: Bu tip, Pod'un bu koşulu karşılayan bir düğümde çalıştırılmasını tercih eder, ancak bu zorunlu değildir. Birden fazla tercih belirtilebilir ve Pod, en çok tercihi karşılayan düğümde çalıştırılır. Örneğin:

```yaml
spec:
  container: 
  - name: sample-app
    image: sample-app
  nodeAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
          - key: "size"
            operator: In
            values:
            - "Large"
    - weight: 2
      preference:
        matchExpressions:
          - key: "disktype"
            operator: In
            values:
            - "ssd"
```
Bu örnekte, Pod öncelikle Large ile işaretlenmiş node üzerinde çalıştırılmaya çalışılır. Bu uygun değilse, Pod, SSD diske sahip bir node üzerinde çalıştırılmaya çalışılır.

3. Anti-Affinity: Bu tip, Pod'un bu koşulu karşılayan bir düğümde çalıştırılmamasını sağlar. Birden fazla anti-affinity belirtilebilir ve Pod, bu koşulların hiçbirini karşılamayan bir düğümde çalıştırılır. Örneğin:

```yaml
spec:
  container: 
  - name: sample-app
    image: sample-app
  nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: "app"
        operator: In
        values:
        - "database"
```
Bu örnekte, "database" uygulamasını çalıştıran bir Pod, aynı düğümde çalışan başka bir "database" Pod'u ile çalıştırılamaz.

## Uygulamalar çalışırken birisi node etiketlerini değiştirirse ne olur?"
Bir düğüm üzerindeki etiketler değiştiğinde uygulamaların nasıl tepki vereceği, etiketlerin nasıl kullanıldığına bağlıdır.

İki olasılık vardır:

1. Etiketler Pod'ların yerleştirilmesini etkilemek için kullanılıyorsa:

Eğer etiketler, Pod'ların hangi düğümlerde çalıştırılabileceğini belirlemek için kullanılıyorsa (node affinity), o zaman etiketler değiştiğinde Pod'lar yeniden programlanabilir. Bu, şu anlama gelir:

Etiketler, Pod'un çalıştığı düğümle eşleşmez hale gelirse: Pod, uygun bir düğüme taşınır.
Etiketler, Pod'un çalıştırılabileceği düğümleri içerecek şekilde değişirse: Pod, yeni düğümlerde çalıştırılabilir hale gelir.

2. Etiketler uygulamalar tarafından doğrudan kullanılıyorsa:

Eğer etiketler uygulamalar tarafından doğrudan kullanılıyorsa (örneğin, konfigürasyon bilgileri sağlamak için), o zaman uygulamalar etiketlerdeki değişikliklere göre farklı şekilde davranabilir. Bu, şu anlama gelir:

Uygulamalar etiketleri dinliyorsa: Etiketler değiştiğinde, uygulamalar otomatik olarak güncellenir ve yeni etiket değerlerine göre davranır.
Uygulamalar etiketleri dinlemiyorsa: Etiketler değiştiğinde, uygulamaların manuel olarak yeniden başlatılması gerekir.
Etiketlerdeki değişikliklere nasıl tepki vereceklerini belirlemek, uygulamaların geliştiricilerine bağlıdır.

Bazı yaygın senaryolar şunlardır:

* Uygulamalar etiketleri bir konfigürasyon kaynağı olarak kullanıyorsa: Etiketlerdeki değişiklikler, uygulamaların yeniden başlatılmasını tetikleyebilir veya uygulamaların çalışma zamanında güncellenmesine neden olabilir.
* Uygulamalar etiketleri bir kimlik doğrulama mekanizması olarak kullanıyorsa: Etiketlerdeki değişiklikler, uygulamaların erişim kontrol listelerini (ACL'ler) güncellemesine neden olabilir.
* Uygulamalar etiketleri izleme veya telemetri için kullanıyorsa: Etiketlerdeki değişiklikler, izleme sistemlerinin güncellenmesine veya telemetri verilerinin yeniden işlenmesine neden olabilir.

Etiketleri değiştirirken, uygulamaların nasıl tepki vereceğini ve herhangi bir kesinti veya veri kaybından kaçınmak için hangi adımların atılması gerektiğini anlamak önemlidir.

## Node-Affinity üzerinde bu durumları nasıl yönetiriz?
Bu senaryoları yönetebilmemiz için node affinity tiplerini iyi anlamamız gereklidir. Örneğin `requiredDuringSchedulingIgnoredDuringExecution` tipini parçalayalım.
Bu tip Pod'un çalıştırılabilmesi için en az bir düğümün bu koşulu karşılaması gerektiğini belirtir. Birden fazla koşul belirtilebilir ve hepsinin karşılanması gerekir.
Parçaladığımızda karşımıza çıkan iki ana yapı bize bu durumu daha iyi açıklar:

* `requiredDuringScheduling`: Pod'un planlama sırasında bu koşulu karşılayan bir düğümde çalıştırılması gerekir. Bu koşulu karşılayan bir düğüm yoksa, Pod planlanamaz ve hata oluşur.
* `IgnoredDuringExecution`: Pod çalışmaya başladıktan sonra bu koşulun karşılanmaması önemli değildir. Pod, koşulu karşılamayan bir düğüme taşınabilir veya koşulu karşılayan bir düğümden kaldırılabilir.

Diğer tipler içinde aynı işlemi yaptığımızda şu detayları görebiliriz:

* `preferredDuringScheduling`: Pod'un planlama sırasında bu koşulu karşılayan bir düğümde çalıştırılması tercih edilir. Bu koşulu karşılayan birden fazla düğüm varsa, Pod, en çok tercihi karşılayan düğümde çalıştırılır.
* `requiredDuringExecution`: Pod çalışmaya başladıktan sonra da bu koşulun karşılanması gerekir. Koşulu karşılamayan bir düğümden Pod kaldırılabilir.

## Node affinity tiplerini seçerken göz önünde bulundurulması gerekenler nelerdir?

* Uygulamanızın gereksinimleri: Uygulamanızın belirli donanıma veya yazılıma ihtiyacı varsa, bu gereksinimleri karşılayan düğümleri seçmek için node affinity'yi kullanabilirsiniz.
* Yüksek kullanılabilirlik: Uygulamanızın her zaman kullanılabilir olması gerekiyorsa, birden fazla düğümde çalıştırmak için node affinity'yi kullanabilirsiniz.
* Güvenlik: Uygulamanız hassas verilerle çalışıyorsa, bu verileri korumak için node affinity'yi kullanabilirsiniz.

## Tip matrisini tanımlayalım

Type 1: `requiredDuringSchedulingIgnoredDuringExecution`
Type 2: `preferredDuringSchedulingIgnoredDuringExecution`
Type 3: `requiredDuringSchedulingRequiredDuringExecution`

| ------ | DuringScheduling | DuringExecution |
| Type 1 |     Required     |     Ignored     |
| Type 2 |     Preferred    |     Ignored     |
| Type 3 |     Required     |     Required    |     