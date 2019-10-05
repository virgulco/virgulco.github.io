---
title: "Docker Nedir?"
date: 2019-10-01 18:21:00 +0800
categories: [DevOps, Docker]
tags: [docker-compose, docker, devops]
---


![Docker]({{ site.baseurl }}/assets/img/posts/docker.png)

Merhaba! Şu küçük balinaya en az bir kaç defa rastlamışsınızdır. Ne işe yaradığına kısaca bakalım.
Docker yazılım geliştiriciler için kolay sanallaştırma, diğer deyişle konteynerleştirme çözümüdür. Konteynerleştirme kodlarınızın bir sanal kutu içerisinde çalışmasını ifade eder. Böylece birden fazla uygulamayı aynı sunucuda fakat birbirinden yalıtılmış farklı Linux sistemlerinde çalıştırabilmenize olanak sağlar. İstenmedikçe bu sistemler birbiriyle konuşmaz.

### Ortaya Çıkışı ve Popüler Oluşu
Docker, 2008 yılında Linux Kernel'e dahil edilen LXC (Linux Containers) üzerine kurulmuş bir teknolojidir. Açıkçası Docker LXC'de manuel olarak yapılan tüm işlemleri standardize etmiş, paketleştirmiş, yönetilebilir hale getirmiş bir aracı yazılımdır. Doğuşunun ardından LXC'nin karışık mimarisini kendine göre yazıp libcontainer mimarisiyle Linux Kernel'inden bağımsız hale gelmişlerdir. Docker'ın bu kadar popüler olmasının sebebi text bazlı imaj formatı kullanmasıdır. Bu imaj formatı ile konteynerlar kolaylıkla oluşturulabilir, konfigüre edilebilir, birebir kopyaları alınıp sonsuz defa kullanılabilir ve kolayca paylaşılabilir olması aslında. 

### Basit Kullanım Mantığı
Docker konteynerları Dockerfile adındaki komut dosyaları ile çalışır. Bu dosyalarda hangi yapının hangi versiyonunu nerede, nasıl kullanacağımızı tanımlarız, Docker bizim için yoksa ilgili imajı download eder, server'ı başlatır ve istediğimiz porttan yayına başlar. Temel mantığımız kurduğumuz yapının her komponenti için bir konteyner oluşturmak olmalı ki buna microservices trendi deniyor. Microservices’ler temellerini 1970’lerde Unix’ten “Do one thing and do it well” yani “Sadece bir şey yap ve iyi yap” felsefesinden almaktadır. Bizim amacımız da kompleks yapıları Docker ile parçalara bölüp en az efor ile en kaliteli verimi almak.

Uygulamanızın nginx üzerinde bir web server, PHP üzerinde bir app server ve PostgreSQL üzerinde bir database'den oluşan yapı olduğunu ve her biri için ayrı bir konteyner oluşturduğunuzu düşünelim. Bağımlılıklarımız için hangi imajlara ihtiyacımız olacağını gösterdik (nginx için git [şu imajı](https://hub.docker.com/_/nginx) al), her bir bağımlılık için versiyonlarımızı belirttik ve çalışmak için bir port numarası verdik. Docker öncelike bu imajları kontrol edecek, eğer kurulu değilse bu imajları download edecek, kuracak, çalıştıracak ve belirttiğimiz portta koşturmaya başlayacak. 

Hemen basit bir örnekle Docker'ı gösterelim:

Kurulum oldukça basit, işletim sisteminize göre ilgili adımları takip etmelisiniz. Ben macOS üzerinden anlatacağım:
- [Windows](https://docs.docker.com/docker-for-windows/)
- [MacOS](https://docs.docker.com/docker-for-mac/)
- [Linux](https://docs.docker.com/install/)

Kurulumu tamamladıysanız bir nginx web server kurup basit bir HTML dosyasını Docker üzerinde çalıştıralım. 
Öncelikle nginx konteynerini Docker'ın resmi reposu [Dockerhub](https://hub.docker.com/)'dan çekelim:

```
docker pull nginx:alpine
```

Proje dizinini oluşturalım:
```
MacBook-Pro:Codes burak$ mkdir docker-example
MacBook-Pro:Codes burak$ cd docker-example
```
Proje dizinimizde bir `Dockerfile` oluşturalım:
```
MacBook-Pro:Codes burak$ mkdir docker-example
MacBook-Pro:Codes burak$ cd docker-example
MacBook-Pro:docker-example burak$ touch Dockerfile
```
`Dockerfile`'ımızı açıp şu komutları ekleyelim:
```
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```
Nerdeyse bitti. Şimdi Docker'ın ayağa kaldırıp bize göstereceği `ìndex.html`dosyamızı hazırlayalım:
```
MacBook-Pro:Codes burak$ mkdir docker-example
MacBook-Pro:Codes burak$ cd docker-example
MacBook-Pro:docker-example burak$ touch Dockerfile
MacBook-Pro:docker-example burak$ touch index.html
```
ìndex.html`dosyamızı aşağıdaki şekilde düzenleyelim:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Nginx Docker</title>
    <style>
        h1{
            font-weight:lighter;
            font-family: Arial, Helvetica, sans-serif;
        }
    </style>
</head>
<body>
    
    <h1>
        Merhaba Docker!
    </h1>

</body>
</html>
```

Bitti! Şimdi konteynerimizi build edelim. Docker bizim için index.html dosyamızı Dockerfile'da gösterdiğimiz şekilde generate edecek. `docker build -t simple-nginx .`komutunu çalıştıralım. Şöyle bir çıktı alacağız:
```
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx:alpine
 ---> 4d3c246dfef2
Step 2/2 : COPY index.html /usr/share/nginx/html/index.html
 ---> da8744f00534
Successfully built da8744f00534
Successfully tagged simple-nginx:latest
```
Artık Docker'a run komutu verebiliriz.
````
docker run --rm -it -p 8080:80 simple-nginx
````
Ta da!
![Hello-Docker]({{ site.baseurl }}/assets/img/posts/hello-docker.png)

Buradaki komutları biraz inceleyelim:
- `--rm`konteynerimiz silindiğinde Docker imajının durmasını sağlar.
- `-it` Docker'ın shell üzerinden interactive çıktılar vermesini sağlar.
- `-p 8080:80` konteynerimizden sunucumuza erişeceğimiz portu belirtir. 80 konteynerden açılan, 8080 ise sunucumuzda konteynerdeki 80 portuna erişeceğimiz porttur. Bunu [Docker Compose İle Multi-Container Yapısı]({{ site.baseurl }}/posts/docker-compose) adlı yazımda Docker Compose kullanarak tek bir dosyada yapabileceğimizi göreceksiniz.
- `simple-nginx` ayağa kaldıracağımız imajın ismi.

İşte en basit şekilde Docker'ın çalışma mantığı böyle.

### Neden Kullanmalıyız?
#### Standardizasyon, Kompleks Yapıları Kolay Yönetim: Docker Compose
Docker Compose birden çok komponentten oluşan kompleks yapıları tek bir dosyada tanımlayarak tek bir komutla tamamını yönetebilmenizi sağlayan bir Docker aracıdır ve güncel Docker sürümü ile tümleşik gelir. Böylece tüm konteynerları kapsayan tek bir konteyner oluşturup her bir konteyner için ayrı işlem yapmamıza gerek kalmıyor. Yani konteynerleri de bir konteyner ile yönetebiliyorsunuz. Böylece kolayca eksiksiz ve standart bir geliştirme ortamı hazırlayabilir, CI pipeline için kullanabilir, tek bir host üzerinde her şeyin çalışmasını sağlayabilirsiniz.

![Docker-Compose]({{ site.baseurl }}/assets/img/posts/docker-compose.png)


#### Kolay Deploy, Load Balancer (Yük Dengeleme), Zero Downtime İle Kesintisiz Servisler: Docker Swarm
Tek bir komut dosyasıyla tüm yapıyı kurup ayağa kaldırabiliyoruz. Peki bunu sunucumuza nasıl deploy edeceğiz? Kodumuzda değişiklik yaptığımızda nasıl güncelleyeceğiz? Sunucuyu durdurmak zorunda mı kalacağız? Erişim kesilecek mi? Docker Swarm deploy, updare/upgrade, sunucu bakımı, yük dengeleme konularında yardımcı bir Docker aracıdır. Test ortamında geliştirme yaptınız ve canlıya aktaracaksınız, ama sitenizin anlık trafiği 1.000.000 ve ufak bir kesinti büyük kayıplara malolabilir. Docker Swarm ile sunucunuzdaki konteynerinizi duplicate edip biri çalışırken öbürüne bakım/güncelleme yapabilirsiniz. Burada bir load balancer'a ihtiyacımız olacak, Docker Swarm bize manger-worker konfigürasyonu sağlıyor. 

Tek sunucu içinde Docker Swarm kullanımı:
![Docker-Swarm-Single]({{ site.baseurl }}/assets/img/posts/docker-swarm-single.jpg)

Birden fazla sunucuda çalışan birden fazla uygulama için Docker Swarm kullanımı:
![Docker-Swarm]({{ site.baseurl }}/assets/img/posts/docker-swarm.png)


#### Maliyet Avantajı
Günümüzün sanal makineleri içinde birden fazla işletim sistemi ve her bir işletim sistemi içinde disk alanı barındırır. Bir sunucu kiraladığınızda muhtemelen aynı makineyi komşu olarak kullanıyor olursunuz. Aynı makinede hem sizin hem de komşunuzun OS dosyaları yer kaplar. Hem siz hem komşunuz için RAM ve işlemci ayrılmıştır.

Konteynerleştirme çözümlerinde ise tek bir işletim sistemi üzerine kurulmuş, aynı disk, RAM ve işlemciyi kullanan fakat her bir uygulamanın ayrı makinadaymışçasına koşturduğu bir yapı oluşur. Bu büyük bir donanım avantajı sağlar. Basit bir hesap yapalım:
64 çekirdekli basit bir Hypervisor içine her biri 5 GB disk alanı, 1 GB RAM ve 1 çekirdek işlemci kullanan 20 adet sanal sunucu kurmak istediğimizde sadece OS dosyaları için 5 GB x 20 = 100 GB disk, 1 GB x 20 = 20 GB memory, 1 çekirdek x 20 = 20 çekirdek kullanmış olduk. Geriye kalan kaynaklarla da uygulamalarımızı çalıştıracağız demektir. Fakat konteynerleştirme çözümü kullandığımızda 20 (veya daha fazla) sanal sunucu için 5 GB disk alanı, 1 GB RAM ve 1 çekirdek yeterli olacaktır.
