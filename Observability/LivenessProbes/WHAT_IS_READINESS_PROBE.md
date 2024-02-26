# LIVENESS PROBE NEDİR?

Liveness probe, bir Kubernetes pod'unun canlı olup olmadığını belirlemek için kullanılan bir mekanizmadır. Bir pod'un canlılığı, birden fazla faktöre bağlı olarak belirlenebilir. Liveness probe'u, pod'un aşağıdakileri sağlayarak canlı olup olmadığını kontrol eder:
* Uygulamanın yanıt verme yeteneği: Probe, pod'un bir HTTP isteğine veya TCP bağlantısına yanıt verip vermediğini kontrol eder.
* Uygulamanın istenen işlevselliği sağlama yeteneği: Probe, pod'un belirli bir işlevselliği yerine getirip getirmediğini kontrol eder.

Liveness probelarını kullanmanın birkaç avantajı vardır:
* Yalnızca canlı pod'lara trafik yönlendirmenizi sağlar: Kubernetes, bir pod'un liveness probelarını başarıyla geçene kadar pod'a trafik yönlendirmez. Bu, yalnızca canlı pod'lara trafik gönderilmesini ve hatalı pod'ların trafiği almasını önler.
* Uygulamanızın otomatik olarak ölçeklenmesini sağlar: Kubernetes, pod'ların canlılığını izlemek için liveness probelarını kullanır. Bir pod canlı değilse, Kubernetes otomatik olarak yeni bir pod oluşturarak uygulamanızı ölçeklendirir.
* Hatalı pod'ları otomatik olarak yeniden başlatmanızı sağlar: Kubernetes, bir pod'un liveness probelarını başarısız hale getirmesi durumunda pod'u otomatik olarak yeniden başlatabilir. Bu, hatalı pod'ların hızlı bir şekilde kurtarılmasını ve uygulamanızın yüksek kullanılabilirliğini korumasını sağlar.

Liveness Probe Ne Zaman Kullanılmalıdır?
Liveness probelarını aşağıdaki durumlarda kullanabilirsiniz:
* Uygulamanızın yüksek kullanılabilirliğini korumak istiyorsanız: Liveness probeları, hatalı pod'ların otomatik olarak yeniden başlatılmasını sağlayarak uygulamanızın yüksek kullanılabilirliğini korumanıza yardımcı olabilir.
* Uygulamanızın otomatik olarak ölçeklenmesini istiyorsanız: Liveness probeları, Kubernetes'in pod'ların canlılığını izlemesine ve gerektiğinde yeni pod'lar oluşturarak uygulamanızı otomatik olarak ölçeklendirmesine yardımcı olabilir.
* Yalnızca canlı pod'lara trafik yönlendirmek istiyorsanız: Liveness probeları, Kubernetes'in yalnızca canlı pod'lara trafik yönlendirmesini sağlayarak hatalı pod'ların trafiği almasını önleyebilir.

Liveness Probe Nasıl Uygulanır?
* Liveness probeları, Kubernetes PodSpec'inin bir parçası olarak tanımlanır. Probe'un aşağıdaki özelliklerini tanımlayabilirsiniz:
* Tür: Probe'un türü (HTTP, TCP veya Exec).
* Hedef: Probe'un nasıl çalıştırılacağını tanımlayan bir yapı.
* Başarılı olma koşulları: Probe'un başarılı sayılması için gereken koşullar.
* Gecikme: Probe'un ilk kez çalıştırılmadan önce gecikecek süresi.
* Periyot: Probe'un tekrarlanma aralığı.
* Başarısızlık toleransı: Probe'un başarısız sayılmadan önce izin verilen başarısızlık sayısı.

## Liveness Probe Örnekleri

1. HTTP Probe:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: my-app
    image: my-app:latest
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /health
        port: 80
        scheme: HTTP
      initialDelaySeconds: 15
      periodSeconds: 20
      failureThreshold: 3
      successThreshold: 1
  restartPolicy: Always
```

Bu örnekte;

* `httpGet` altında path, port ve scheme belirtilerek HTTP isteği gönderilir.
* `initialDelaySeconds` ilk probe çalıştırılmadan önce geçen süreyi belirler.
* `periodSeconds` probe tekrarlanma aralığını belirler.
* `failureThreshold` probe başarısız sayılmadan önceki başarısızlık sayısını belirler.
* `successThreshold` probe başarılı sayılması için gereken başarılı yanıt sayısını belirler.

2. TCP Probe:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-db-pod
spec:
  containers:
  - name: my-db
    image: mysql:latest
    ports:
    - containerPort: 3306
    livenessProbe:
      tcpSocket:
        port: 3306
      initialDelaySeconds: 15
      periodSeconds: 20
      failureThreshold: 3
  restartPolicy: Always
```

Bu örnekte;

* `tcpSocket` altında port belirtilerek TCP bağlantısı açılır.
Diğer parametreler HTTP probe örneğiyle aynı amaca hizmet eder.

3. Exec Probe:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-check-pod
spec:
  containers:
  - name: health-check
    image: busybox:latest
    command: ["sh", "-c"]
    args: ["nc -z localhost 80 && exit 0 || exit 1"]
    livenessProbe:
      exec:
        command: ["sh", "-c"]
        args: ["nc -z localhost 80 && exit 0 || exit 1"]
      initialDelaySeconds: 15
      periodSeconds: 20
      failureThreshold: 3
  restartPolicy: Always
```

Bu örnekte;

* `exec` altında komut belirtilerek çalıştırılır.
Komutun başarılı çıkış kodu ile probe başarılı kabul edilir.
Diğer parametreler diğer örneklerle aynı amaca hizmet eder.