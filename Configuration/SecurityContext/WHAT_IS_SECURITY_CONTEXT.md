# SECURTIY CONTEXT
Kubernetes içindeki "securityContext" özelliği, bir Kubernetes pod'unun güvenlik ayarlarını ve davranışlarını belirlemek için kullanılan bir özelliktir. Bu özellik, podlar ve konteynerler düzeyinde güvenlik politikalarını özelleştirmek için kullanılır ve CKA (Certified Kubernetes Administrator) sınavında da detaylı bir şekilde anlayış gerektirebilecek bir konudur.

SecurityContext, aşağıdaki önemli işlevlere sahiptir:

* Pod ve Konteyner Güvenliği: SecurityContext, pod ve konteynerlerin çalışma zamanında belirli güvenlik ayarlarını uygulamasını sağlar. Bu ayarlar, potansiyel güvenlik tehditlerini azaltmaya yardımcı olur. Örneğin, bir securityContext özelliği, pod'un çalışma kullanıcısını veya grup kimliğini belirleyebilir.

* İzolasyon ve Sınırlama: SecurityContext, kaynak sınırları (CPU ve bellek), özel bir kullanıcı kimliği veya grup kimliği, root erişimi gibi konteynerlerin güvenliğini artırmak için izolasyon ve sınırlama politikalarını tanımlamak için kullanılır.

* SELinux veya AppArmor: Linux tabanlı sistemlerde, SecurityContext özelliği, özel bir güvenlik politikası uygulamak için SELinux veya AppArmor gibi güvenlik modellerini etkinleştirmeye yardımcı olur.

* Yönetilen İzinler: SecurityContext, bir pod'un ve konteynerlerinin dosya ve dizin izinlerini ayarlamak için kullanılabilir. Örneğin, dosya izinleri ve sahipliği, securityContext ile belirlenebilir.

CKA sınavında, securityContext özelliği hakkında sorularla karşılaşmanız muhtemeldir. Bu nedenle bu özelliği iyi anlamak ve uygulamak önemlidir. Pod ve konteyner güvenliğini artırmak için securityContext özelliğini kullanabilme yeteneği CKA sınavında olumlu bir değer katabilir. Bu konuyu çalışırken, güvenlik politikalarını nasıl tanımlayacağınızı ve hangi parametreleri kullanmanız gerektiğini öğrenmek önemlidir.