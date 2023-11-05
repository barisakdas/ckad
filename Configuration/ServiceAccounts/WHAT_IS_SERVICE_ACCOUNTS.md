# WHAT IS SERVICE ACCOUNTS
Kubernetes içindeki "ServiceAccount" bileşeni, Kubernetes küme içinde çalışan uygulamaların (pod'lar) diğer Kubernetes bileşenleri ve hizmetleriyle güvenli bir şekilde iletişim kurabilmesini sağlayan bir kimlik doğrulama mekanizmasıdır. ServiceAccount'lar, her pod veya çalıştırılabilir nesne için özelleştirilmiş yetkilendirmeler ve izinler sağlar. Aşağıda ServiceAccount'lerin önemli işlevleri ve CKA sınavında çıkabilecek detaylar verilmiştir:

* Kimlik Doğrulama: ServiceAccount, bir pod'un kimliğini belirler ve bu kimliği kullanarak diğer Kubernetes bileşenleriyle iletişim kurmasını sağlar. Bu, bir pod'un diğer pod'lar, servisler veya diğer Kubernetes bileşenleriyle güvenli bir şekilde etkileşime girmesine yardımcı olur.

* Yetkilendirme: ServiceAccount'lar, bir pod'un Kubernetes kaynaklarına (örneğin, bir depo, bir servis veya bir PVC) erişim yetkilerini denetler. Bu sayede her pod'un sadece ihtiyaç duyduğu kaynaklara erişmesi sağlanır.

* RBAC (Rol Tabanlı Erişim Kontrolü): Kubernetes içinde ServiceAccount'lar, RBAC politikalarını kullanarak pod'lara erişim yetkilerini belirler. Bu, CKA sınavında sıkça karşılaşabileceğiniz bir konudur.

* Pod ve Konteynerler İçin İzolasyon: ServiceAccount'lar, pod'ların birbirlerinden izole edilmesine yardımcı olur ve her podun kendi kimlik bilgilerini korumasını sağlar.

ServiceAccount'ler, Kubernetes kümesindeki uygulamaların güvenliğini artırmak için önemli bir rol oynar ve CKA sınavında bu konuyla ilgili sorularla karşılaşmanız olasıdır. ServiceAccount'lar hakkında detaylı bilgi sahibi olmak ve bunların nasıl oluşturulacağını, yönetileceğini ve RBAC ile nasıl bütünleştirileceğini öğrenmek CKA sınavına hazırlanırken size yardımcı olabilir.

----------------------------------------------------------------

# SERVICE ACCOUNTS DETAILS
Kubernetes içerisinde 2 tip hesap vardır; User & Service
* User hesapları isminden anlaşılacağı üzere kullanıcılar için açılır. Bu hesabı kullanan kişiler uygulama geliştiricileri veya sistemleri kontrol eden admin kullanıcılar olabilirler.
* Service hesapları ise makineler, yani uygulamalar için açılır ve kullanılır. Bu hesaplar ise uygulamalar tarafından Kubernetes ile iletişime geçmek için kullanılabilirler. Örneğin: Prometheus gibi bir monitoring uygulaması performans ve metrikler için Kubernetes api servislerine bağlanmak için service hesaplarını kullanır. Ya da AzureDevops ya da Jenkins gibi uygulama dağıtımından sorumlu araçlar uygulamaları deploy edebilmek için service hesabı kullanırlar.

Bu hesaplar için daha açıklayıcı ve basit bir örnekleme yapalım.
Herhangi bir dil ile yazılmış ve Kubernetes üzerindeki uygulamalarımızı izlememizi sağlayan bir uygulamamız olsun.
Bu uygulama Kubernetes'e bağlanarak podları ve diğer componentleri çekerek ekranda bize gösteriyor olsun.
Kubernetes servisine bağlanabilmesi için bu uygulamanın yetki kontrollerinden başarı ile geçmesini bekleriz. Bunun için servis hesaplarını kullanıyoruz ve yetkilendirme ve doğrulama(authorization & authentication) işlemlerini bu hesap üzerinden yapıyoruz.
Bir service hesabı oluşturmak için iki yol vardır:

## Uygula Kubernetes kümeleri(node) üzerinde barındırılmayan üçüncü taraf bir uygulama olarak kullanılsın.
Bu durumda uygulamamızın dışarıdan önce Kubernetes'e erişimi gerekecektir.

```bash
kubectl create serviceaccount <your_service_account_name>
kubectl get serviceaccount 
kubectl describe serviceaccount <your_service_account_name>
```
Bir servis hesabı oluşturulduğunda otomatik olarak bir TOKEN oluşur. Bu token uygulamanın kullanması gereken tokendir. Bu token bir secret nesnesi olarak saklanır.
Service hesaplarını describe komutu ile çağırdığınızda göreceksiniz ki bu token `<your_service_account_name>-token-kbbdm` olarak adlandırılır.
Yani servis hesabı oluşturulduğunda bir toke oluşur. Hemen ardından bir secret komponenti oluşur ve bu token oluşan secret komponentine eklenir. Ve bu secret daha sonrasında service hesabına bağlanır. Buradaki secreti görmek için aşağıdaki komutu yazdığınızda içindeki veriyi görebileceksiniz.

```bash
kubectl describe secret <your_service_account_name>-token-kbbdm
```
Bu token Kubernetes api isteklerinde doğrulama işlemi için kullanılır. Örneğin bir curl isteğinde bu token bearer olarak eklendiğinde istek başarılı olacaktır.

```bash
curl https://<your_cluster_ip>:<port> -insecure --header "Authorization: Bearer <your_service_account_token_starts_with_ey...>"
```
Bu oluşan token yukarıda bahsettiğimiz izleme uygulamasının istek alanına eklenebilir. Bu sayede her istekte bu token kullanılacaktır.

## Peki bizim yazdığımız bu izleme uygulaması Kubernetes içerisinde barıdırılıyorsa?
Bu durumda service hesabı belirtecini(token) dışa aktarma ve üçüncü taraf uygulamasının bunu kullanacak şekilde yapılandırılmasının tamamını podun içerisinde secret dosyasını monte etme işlemi(mount) ile sağlayabilirsiniz. Bu sayede uygulama kendi yaml dosyasının içerisine yerleştirilen berliteci(token) kolayca okuyabilir.
Bu işlemi elle sağlamak zorunda değilsiniz.
Kubernetes içerindeki tüm alan adları(namespace) için otomatik olarak alan adları ile aynı olacak şekilde bir servis hesabı oluşur.

```bash
kubectl get serviceaccount
```
Bu alan adı içerisinde bir uygulama ayağa kaldırıldığında(pod oluştuğunda) otomatik olarak bu varsayılan olarak oluşturulan servis hesabı ve belirteci(token) otomatik olarak -tanım dosyasında(yaml) bunu belirtmenize gerek olmaksızın- bu uygulamaya(pod) monte edilir. Bunu uygulamanın detaylarını almaya çalıştığınızda görebilirsiniz.

```bash
kubectl describe pod <your_pod_name>
```

Bu detayların içerisinde `Volume` bağlığında bu bağlantıyı görebilirsiniz. Bu varsayılan servis hesabının belirteç(token) bilgisi `/var/run/secrets/kubernetes.io/serviceaccount` konumuna bağlanır.

```bash
kubectl exec -it <your_pod_name> --ls /var/run/secrets/kubernetes.io/serviceaccount
```

Bu komutu çalıştırdığınızda göreceğiniz ekranda `token` isminde bir dosya bulunur.

```bash
kubectl exec -it <your_pod_name> --ls /var/run/secrets/kubernetes.io/serviceaccount/token
```
Yukarıdaki komut ile token dosyasının içerisinde yazan bilgiye erişebilirsiniz.
Tabi size varsayılan olarak gelen bu belirtecin çok kısıtlı yetkilerinin olduğunu söylememde fayda var.
Bu belirteç sadece temel api komutlarını çalıştırmak için tasarlanıyor.
Bu durumda kendi servis hesabımızı oluşturmak ve bunu uygulamamıza bağlamak istersek nasıl bir yol izleyebilirizi tartışalım.
Uygulamamızı dağıttığımız tanım dosyasının(pod_definition.yaml ) içerisine aşağıdaki blokları ekleyerek kendi oluşturduğumuz servis hesabını buna bağlayabiliriz. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mykubernetesapp
  labels:
    type: application
spec:
  containers:
    - name: mykubernetesapp
      image: mykubernetesapp-image
  serviceAccountName: mykubernetesapp-sa 
```

NOT: Bu monte işleminin yalnızda uygulamanın yeniden dağıtımı esnasında kullanılacağını, mevcut uygulamaların yaml dosyalarının editlenmesinde servis hesabının değişmeyeceğini hatırlatmak istiyorum. Zaten mevcuttaki bir podun servis hesap bölümünü güncellemenize Kubernetes izin vermeyecektir. Bu durumda uygulamayı silmeniz ve tekrar dağıtmanız gereklidir. 

```bash
kubectl delete pod <your_pod_name> --force
kubectl apply -f <your_new_pod_definition_yaml_file>
```

NOT: Eğer ki bir uygulama dağıtırken otomatik olarak bir servis hesabının monte edilmesini istemiyorsanız bunu `automountServiceAccountToken: false` komutu ile yaml dosyanıza eklemeniz gereklidir. Aksi takdirde varsayılan işlem olarak otomatik monte edecektir.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mykubernetesapp
  labels:
    type: application
spec:
  containers:
    - name: mykubernetesapp
      image: mykubernetesapp-image
  automountServiceAccountToken: false 
```

## Kubernestes version 1.22 ve 1.24 ile gelen değişiklikler.
Yukarıda anlattığımız gibi varsayılan olarak oluşturulan belirteç uygulamanın erişimleri için kullanılıyordu.
Peki bu belirteci parçalayarak içerisinde neler var bakmak isteseydik neler görürdük.
Şimdi dağıtılmış uygulamamızın detaylarını alalım ve o kısımda gördüğümüz `Mounts` alanındaki belirtecin detaylarına bakalım.

```bash
kubectl exec -it <your_pod_name> --ls /var/run/secrets/kubernetes.io/serviceaccount/token
```
Buradan aldığımız değeri `jwt.io` websitesinde parçalayalım. 

```json
{
  "iss": "https://kubernetes.default.svc.cluster.local",
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {
      "name": "dashboard-sa",
      "uid": "9b2593ab-05c9-49c6-9128-fff9ad856596"
    }
  },
  "nbf": 1699191954,
  "sub": "system:serviceaccount:default:dashboard-sa"
}
```

Burada gördüğümüz şeylerden daha çok göremediklerimiz bizi ilgilendiriyor.
İki parça göremediğimiz şeyi sizinle paylaşayım

```json
  "aud": [],    // Audience: belirtecin kimler tarafından kullanılabileceğini gösterir.
  "exp": 0      // ExpireData: belirtecin bitiş süresini gösterir.
```
Yukarıdaki json dan görebileceğiniz gibi üretilen belirtecin bir kısıtı ve süresi bulunmuyor. Bu da bize bir güvenlik sorunu doğurur.
Kubernetes 1.22 versiyonunun duyurusunda yaptığı bir açıklama(`KEP 1205 - Bound Service Account Token`) ile bu yapının güvenlik ve ölçeklenebilirlik konularında
sorun teşkil ettiğini açıkladı. Bu halde üretilen bir belirteç zaman ve erişim kısıtları olmadan sunuluyordu. Bu yüzden bu belirteç servis hesabı yaşadığı sürece geçerliydi.

Ayrıca her belirteç için bir tane servis hesabı gerekiyordu ve bu da ölçeklenebilirlik sorunlarına neden oluyordu.
Bu nedenle Kubernetes 1.22 sürümünde `TokenRequestAPI` servisini belirteçlerin bir Kubernetes api aracılığıyla daha güvenli ve ölçeklenebilir bir şekilde sağlanması için geliştirdi.
Dolayısıyle TokenRequestAPI tarafından üretilen belirteçler artık kısıtlamalar ile üretilmeye başlandı.
* Audience Bound
* Time Bound
* Object Bound

Versiyon 1.22 den sonra oluşan tüm uygulamalarda(pod) eskisi gibi bir belirteç yapısı yerine yeni api tarafından üretilen bir belirteci alıyor.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mykubernetesapp
  labels:
    type: application
spec:
  containers:
    - name: mykubernetesapp
      image: mykubernetesapp-image
      volumeMounts: 
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-6mtg8
        readOnly: true
  volumes:
    - name: kube-api-access-6mtg8
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken: 
            expirationSeconds: 3607
            path: token 
```

Burada gördüğünüz gibi volue ve volumeMount alanları `kube-api-access-6mtg8` ismi ile birbirlerine bağlanıyor.

Versiyon 1.24'te getirilen değişikliklerle beraber bir servis hesabı oluşturduğumuzda otomatik olarak oluşan bir belirteç ve secret artık oluşmuyor.
Eğer ki bir belirteç üretmek istiyorsanız artık aşağıdaki komutu çalıştırmanız gerekiyor.

```bash
kubectl create token <your_service_account_name>
```

Bu komut ilgili servis hesabı için bir belirteç oluşturur ve bunu ekrana yazdırır.
Ekranda yazan belirteci parçaladığımızda artık şu şekilde görebiliriz.

```json
{
  "aud": [
    "https://kubernetes.default.svc.cluster.local",
    "k3s"
  ],
  "exp": 1699195554,
  "iat": 1699191954,
  "iss": "https://kubernetes.default.svc.cluster.local",
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {
      "name": "dashboard-sa",
      "uid": "9b2593ab-05c9-49c6-9128-fff9ad856596"
    }
  },
  "nbf": 1699191954,
  "sub": "system:serviceaccount:default:dashboard-sa"
}
```

Burada da gördüğünüz gibi artık belirteç içerisinde `aud ve exp` alanlarını görebilirsiniz.

# Eğerki hala eskisi gibi süresi dolmayan bir belirteç oluşturmak istiyorsak ne yapmalıyız?
Bu durumda artık bir secret komponenti oluşturarak bunun içerisinde servis hesabını elle vermemiz gereklidir.

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: serviceaccount
  annotations:
    kubernetes.io/service-account.name: myserviceaccount
```

Bu yaml ın çalışması için öncesinde mutlaka belirtilen isimde bir servis hesabınızın var olmasına dikkat ediniz.
Böylece süresi dolmayan bir belirteç üretebileceksiniz. 

NOT: Bu tercihin getireceği güvenlik sorunlarını kabul etmeden uygulamamanızı öneririm.