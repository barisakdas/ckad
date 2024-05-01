# Kubernetes'te Deploymen Stratejileri Nelerdir?

## Kubernetes içerisinde varsayılan strateji: RollingUpdates

Kubernetes, uygulamaların dağıtımı ve yönetimi için kullanılan popüler bir platformdur. Bir uygulamayı Kubernetes'te dağıttığınızda, birden fazla pod'da çalıştırılabilir. Bir pod, bir konteynerin tek bir örneğidir.

Uygulamanızda değişiklik yapmak istediğinizde, tüm pod'ları aynı anda yeniden başlatmak istemezsiniz. Bu, uygulamanızda bir kesintiye neden olur. Rolling updates, bu sorunu çözmek için kullanılan bir stratejidir.

Rolling updates ile, pod'larınızı tek tek yeniden başlatabilirsiniz. Bu, uygulamanızda kesinti olmadan değişiklikleri dağıtmanıza olanak tanır.

Rolling Updates Nasıl Çalışır?

Rolling updates, aşağıdaki adımları izleyerek çalışır:

Yeni bir pod'u yeni uygulama sürümüyle başlatırsınız.
Yeni pod çalışmaya başladıktan sonra, eski pod'lardan birini sonlandırırsınız.
Bu işlemi, tüm eski pod'lar sonlanana kadar tekrarlarsınız.
Rolling Updates'in Avantajları:

Uygulamanızda kesinti olmadan değişiklikleri dağıtmanıza olanak tanır.
Yeni uygulama sürümünün hatalı olması durumunda, eski sürümü geri almanıza olanak tanır.
Uygulamanızın yükünü dengelemenize olanak tanır.
Rolling Updates'in Dezavantajları:

Karmaşık olabilir.
Yeni uygulama sürümü eski sürümle uyumlu değilse, sorunlara neden olabilir.
Rolling Updates Örneği:

Bir web uygulamanız olduğunu varsayalım. Uygulamanızın iki pod'da çalıştığını varsayalım. Uygulamanızda bir değişiklik yapmak istiyorsunuz.

Rolling updates'i kullanarak değişikliği dağıtmak için aşağıdakileri yaparsınız:

Yeni bir pod'u yeni uygulama sürümüyle başlatırsınız.
Yeni pod çalışmaya başladıktan sonra, eski pod'lardan birini sonlandırırsınız.
Bu işlemi, tüm eski pod'lar sonlanana kadar tekrarlarsınız.
Bu işlem tamamlandıktan sonra, uygulamanızın tüm pod'ları yeni uygulama sürümünü çalıştıracaktır.