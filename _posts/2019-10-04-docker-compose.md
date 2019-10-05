---
title: "Docker Compose İle Multi-Container Yapısı"
date: 2019-10-04 18:21:00 +0800
categories: [DevOps, Docker]
tags: [vagrant, docker, devops]
---

![Docker]({{ site.baseurl }}/assets/img/posts/docker-compose.png)

[Docker Nedir?]({{ site.baseurl }}/posts/docker-nedir) adlı yazımda Docker Compose'un ne işe yaradığından bahsetmiştim. Şimdi biraz nasıl kullanıldığına bakalım.

Kabaca üç adet web service'imiz olduğunu ve üçünün de ayrı konteynerde olduğunu düşünelim. Bu üç service'i nginx ile farklı portlardan koşturmak için docker-compose.yml dosyası içerisine aşağıdaki gibi bir kod bloğu yazmamız gerekecek. Bu komutlarla elimizde üç ayrı portta çalışan üç nginx imajlı konteyner oluşacak:
```
version: '2'

services:
   company_a_web_server:
      image: nginx:latest
      ports:
         - "8001:80"

   company_b_web_server:
      image: nginx:latest
      ports:
         - "8002:80"

   company_c_web_server:
      image: nginx:latest
      ports:
         - "8003:80"
```

Biraz daha karmaşık bir yapıyı çözelim. Örneğin Node.js ile geliştirdiğiniz bir proje var, MySQL ile Redis Cache'i aynı anda kullanıyor. Docker'ın olmadığı bir hayatta bu üç yapıyı ayağa kaldırıp çalıştırmak için:

- Node.js kurulumu yapılacak,
- NPM kurulumu yapılacak,
- Redis kurulumu yapılacak,
- Mysql kurulumu yapılacak,
- Mysql içinde yeni bir veritabanı açılacak ve giriş bilgileri tanımlanacak,
- Nginx kurulumu yapılacak,
- Nginx içinde yönlendirme için düzenlemeleri yapacak,
- Projenin dosyaları ilgili dizinlere taşınacak,
- Dizinlere ilişkin yetkilendirmeler yapılacak,
- Proje ayağa kalkması için npm paketlerinin yüklenmesi sağlanacak,
- Proje ayağa kaldırmak için komut çalıştırılacak,
- Proje durduğunda kendiliğinden ayağa kalkması için ek programların kurulması gerekecek,

Gelin hepsini tek bir `docker-compose.yml` dosyasında yapalım:

````
mysql:
  container_name: mysql
  build: /home/codes/container/mysql
  ports:
    - "3306:3306"
  environment:
    - MYSQL_USER=username
    - MYSQL_PASSWORD=password
    - MYSQL_ROOT_PASSWORD=password
  volumes:
    - /home/codes/data/db/mysql:/var/lib/mysql
    - /home/codes/db/mysql-config.cnf:/etc/mysql/conf.d/mysql-config.cnf
    - ./install.sql:/docker-entrypoint-initdb.d/install.sql
  restart: always

project:
  container_name: NodeExample
  build: /home/codes/container/NodeExample
  ports:
    - 2181:2181
    - 9092:9092
  environment:
    - ADVERTISED_HOST=192.168.50.4
  restart: always

redis:
  container_name: redis
  build: /home/codes/container/redis
  ports:
    - "6379:6379"
  restart: always
  ````
Yukarıdaki örnekte imaj olarak kendi build ettiğimiz imajlar kullanılmış, bunun yerine her defasında hazır imajlar da kullanabiliriz.
