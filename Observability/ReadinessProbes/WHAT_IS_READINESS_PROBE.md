# READINESS PROBE NEDİR?

Readiness probeları, bir pod'un kullanıma hazır olup olmadığını belirlemek için kullanılan mekanizmalardır. Bir pod, tüm başlangıç ​​kontrollerini geçtiğinde ve kullanıma hazır hale geldiğinde, Kubernetes bunu readiness probelarını kullanarak algılar. Readiness probeları, pod'un aşağıdakileri sağlayarak kullanıma hazır olup olmadığını kontrol eder:

Uygulamanın yanıt verme yeteneği: Probe, pod'un bir HTTP isteğine veya TCP bağlantısına yanıt verip vermediğini kontrol eder.
Uygulamanın istenen işlevselliği sağlama yeteneği: Probe, pod'un belirli bir işlevselliği yerine getirip getirmediğini kontrol eder.
Readiness probelarını kullanmanın birkaç faydası vardır:

Yalnızca kullanıma hazır pod'lara trafik yönlendirmenizi sağlar: Kubernetes, bir pod'un readiness probelarını başarıyla geçene kadar pod'a trafik yönlendirmez. Bu, yalnızca kullanıma hazır pod'lara trafik gönderilmesini ve hatalı pod'ların trafiği almasını önler.
Uygulamanızın otomatik olarak ölçeklenmesini sağlar: Kubernetes, pod'ların kullanılabilirliğini izlemek için readiness probelarını kullanır. Bir pod kullanıma hazır değilse, Kubernetes otomatik olarak yeni bir pod oluşturarak uygulamanızı ölçeklendirir.
Hatalı pod'ları otomatik olarak yeniden başlatmanızı sağlar: Kubernetes, bir pod'un readiness probelarını başarısız hale getirmesi durumunda pod'u otomatik olarak yeniden başlatabilir. Bu, hatalı pod'ların hızlı bir şekilde kurtarılmasını ve uygulamanızın yüksek kullanılabilirliğini korumasını sağlar.

Örnek Senaryolar
1. HTTP Probe:
Bir web uygulaması çalıştıran bir pod için, pod'un HTTP isteğine yanıt verip vermediğini kontrol etmek için bir HTTP probe kullanabilirsiniz. Probe, pod'un IP adresine ve belirli bir bağlantı noktasına bir HTTP isteği gönderir. Pod, isteğe belirli bir sürede yanıt verirse, kullanıma hazır kabul edilir.

2. TCP Probe:
Bir veritabanı sunucusu çalıştıran bir pod için, pod'un TCP bağlantısına yanıt verip vermediğini kontrol etmek için bir TCP probe kullanabilirsiniz. Probe, pod'un IP adresine ve belirli bir bağlantı noktasına bir TCP bağlantısı açmaya çalışır. Pod bağlantıyı kabul ederse, kullanıma hazır kabul edilir.

3. Exec Probe:
Bir komut çalıştıran bir pod için, pod'un komutu başarıyla çalıştırıp çalıştırmadığını kontrol etmek için bir exec probe kullanabilirsiniz. Probe, pod'da belirli bir komutu çalıştırır ve komutun çıkış kodunu kontrol eder. Komut, belirli bir çıkış kodu ile tamamlanırsa, pod kullanıma hazır kabul edilir.

Detaylı Açıklama
Readiness probeları, Kubernetes PodSpec'inin bir parçası olarak tanımlanır. Probe'un aşağıdaki özelliklerini tanımlayabilirsiniz:

Tür: Probe'un türü (HTTP, TCP veya Exec).
Hedef: Probe'un nasıl çalıştırılacağını tanımlayan bir yapı.
Başarılı olma koşulları: Probe'un başarılı sayılması için gereken koşullar.
Gecikme: Probe'un ilk kez çalıştırılmadan önce gecikecek süresi.
Periyot: Probe'un tekrarlanma aralığı.
Başarısızlık toleransı: Probe'un başarısız sayılmadan önce izin verilen başarısızlık sayısı.

## Readiness Probe için YAML Örnekleri

1. HTTP Probe:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /health
            port: 80
            scheme: HTTP
          initialDelaySeconds: 15
          periodSeconds: 20
          failureThreshold: 3
          successThreshold: 1
```

Bu örnekte:

`httpGet` altında path, port ve schema belirtilerek HTTP isteği gönderilir.
`initialDelaySeconds` ilk probe çalıştırılmadan önce geçen süreyi belirler.
`periodSeconds` probe tekrarlanma aralığını belirler.
`failureThreshold` probe başarısız sayılmadan önceki başarısızlık sayısını belirler.
`successThreshold` probe başarılı sayılması için gereken başarılı yanıt sayısını belirler.

2. TCP Probe:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
      - name: my-db
        image: mysql:latest
        ports:
        - containerPort: 3306
        readinessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 15
          periodSeconds: 20
          failureThreshold: 3
```

Bu örnekte:

`tcpSocket` altında port belirtilerek TCP bağlantısı açılır.
Diğer parametreler HTTP probe örneğiyle aynı amaca hizmet eder.

3. Exec Probe:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: health-check
spec:
  replicas: 1
  selector:
    matchLabels:
      app: health-check
  template:
    metadata:
      labels:
        app: health-check
    spec:
      containers:
      - name: health-check
        image: busybox:latest
        command: ["sh", "-c"]
        args: ["nc -z localhost 80 && exit 0 || exit 1"]
        readinessProbe:
          exec:
            command: ["sh", "-c"]
            args: ["nc -z localhost 80 && exit 0 || exit 1"]
          initialDelaySeconds: 15
          periodSeconds: 20
          failureThreshold: 3
```

Bu örnekte:

`exec` altında komut belirtilerek çalıştırılır.
Komutun başarılı çıkış kodu ile probe başarılı kabul edilir.
Diğer parametreler diğer örneklerle aynı amaca hizmet eder.