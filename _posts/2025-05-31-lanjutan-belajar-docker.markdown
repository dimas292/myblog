---
layout: post
title: "Setup Mysql Database image + Spring Boot"
date: 2025-05-31 12:00:00 +0700
categories: virtualization
---

Oleh Dimas | 31 Mei 2025

Hallo, masih dengan gue Dimas. Kali ini gue akan melanjutkan *series* belajar tentang *containerization* dan untuk *project*-nya menggunakan **Spring Boot** sebagai *backend*-nya + **MySQL** untuk *database*-nya.

### Persiapan dan Miskonsepsi Awal

Yang harus dipersiapkan apa saja? *Backend* Java CRUD dengan *package* menggunakan Maven sederhana saja karena di sini belajarnya kan itu ya tentang *container*. Dari *project* Java ini, dependensi yang gue gunakan hanya Spring Boot, Lombok, Spring JPA, dan MySQL Connector.

Proses pembuatan *entity*, *controller* enggak gue tampilin ya, langsung aja gue tes di lokal apa aplikasinya bisa jalan apa enggak.

Perintah buat *run* Java Spring Boot:

*Compile* terlebih dahulu:

```bash
mvn package
```

Lalu jalankan:

```bash
mvn spring-boot:run
```

**Miskonsepsi 1: "Kalau aplikasi sudah jalan di lokal, berarti pasti aman di mana pun."**
**Yang perlu digarisbawahi:** Enggak selalu! Berjalan di lokal dengan *setup* langsung di OS itu beda jauh sama kalau sudah masuk ke *container*. Di *container*, ada *environment* yang terisolasi dan itu bisa jadi sumber masalah baru kalau konfigurasi *network* atau *volume*-nya salah. Makanya, penting banget buat belajar *containerization* kayak gini.

Setelah semua aman berjalan di lokal, gue lanjut *setting database* yak.

### Menjalankan MySQL di Docker

Pertama-tama kita *pull images* MySQL dari Docker Hub. Di sini versi yang gue gunakan `mysql:8.0.33`.

```bash
docker pull mysql:8.0.33
```

Berikut adalah cara menjalankan MySQL di Docker:

```bash
docker run --name mysql-container \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=mydb \
-e MYSQL_USER=dimas \
-e MYSQL_PASSWORD=rahasia \
-v mysql_data:/var/lib/mysql \
-d \
-p 3306:3306 \
mysql:8.0.33
```

Terlihat agak rumit yak? Mari kita bahas dengan pengetahuan gue yang seadanya. Jadi intinya adalah kita membuat *container* dengan nama `mysql-container` dengan langsung *setting password*, *user*, dan *database*-nya. Untuk *flag* `-d` agar *container*-nya berjalan di latar belakang. Nah, kalau `-p` itu buat *publish port* atau kayak *binding port* gitulah, dan terakhir itu nama *image*-nya.

*Container* MySQL sudah berjalan di latar belakang. Untuk mengakses *database*-nya bisa pakai `exec`, contohnya:

```bash
docker exec -it <id-container> bash
```

```bash
~bash# mysql -u dimas -p
~mysql>
```

Oiya, lupa! Kalau *error* saat menjalankan *container* MySQL, mungkin itu juga ada *port* dari lokal Anda yang sedang aktif.

Lanjut, gue nyoba tes lagi di *project*-nya, namun *database* sekarang menggunakan *container*, dan *yapp* sukses!

**Miskonsepsi 2: "Kalau sudah jalan pakai *container database*, berarti semuanya sudah terintegrasi otomatis."**
**Yang perlu digarisbawahi:** Nanti dulu! Meskipun *database container* sudah jalan, belum tentu aplikasi Spring Boot-mu bisa langsung nyambung. Ini dia momen krusial tentang **Docker Network** yang bakal kita bahas. Tanpa *network* yang benar, aplikasi dan *database* bisa jadi kayak orang yang beda alam, enggak bisa ngobrol!

Di tengah-tengah perjalanan gue kepikiran buat bikin *migration* dengan Flyway, tapi nanti malah enggak kelar-kelar *task* gue, jadi lanjut aja lah!

### Membuat Dockerfile untuk Aplikasi Java

Lanjut! Gue buat `Dockerfile` di *project* Java gue buat dijadikan *docker image* dengan konfigurasi seperti ini:

```Dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/spring-boot-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

CMD [ "java", "-jar", "app.jar" ]
```

Sengaja gue ambil yang `openjdk-slim` supaya *size*-nya kecil, gue pakai paket data coy, takutnya boncos. Untuk `WORKDIR` itu ketika kita masuk ke *container* diarahkan langsung ke `/app`. `COPY` arahkan ke `target` hasil *compile*-nya, argumen kedua diisi nama hasil *compile*. `EXPOSE` *port* yang bakalan kita pakai. `CMD` untuk menjalankan *script* `-jar` untuk menjalankan *file jar*.

Setelah jadi *image*, gue coba *run* dengan... eh, taunya *error*! Karena koneksi belum dibuat atau biasa disebut **Docker Network**.

**Miskonsepsi 3: "Kalau sudah bikin *image* dan *container* jalan, aplikasi langsung bisa komunikasi sama *container* lain."**
**Yang perlu digarisbawahi:** Enggak semudah itu, Ferguso! Setiap *container* itu secara *default* punya *network* isolasi sendiri. Kalau mau mereka bisa "ngobrol" satu sama lain (misalnya, aplikasi Spring Boot-mu mau nyambung ke *database* MySQL), kamu perlu **membuat Docker Network khusus** dan memasukkan kedua *container* itu ke *network* yang sama. Ini *step* yang sering banget dilupakan *developer* pemula.

### Solusi: Docker Network

Oke, kita buat *network*-nya terlebih dahulu seperti ini:

```bash
docker network create app-network
```

Setelah itu, jalankan ulang MySQL *service* dengan tambahan *flag* `--network <nama-network>` seperti ini:

```bash
docker run --name mysql-container \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=mydb \
-e MYSQL_USER=dimas \
-e MYSQL_PASSWORD=rahasia \
-v mysql_data:/var/lib/mysql \
-d \
-p 3306:3306 \
--network app-network \
mysql:8.0.33
```

Sekarang sudah satu *network* yang terisolasi. Jadi *env* Spring Boot-nya sudah bisa akses *container* MySQL dengan *config* seperti ini:

```java
# Database connection
spring.datasource.url=jdbc:mysql://mysql:3306/mydb?allowPublicKeyRetrieval=true&useSSL=false
spring.datasource.username=dimas
spring.datasource.password=rahasia
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate properties
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```
**Yang perlu digarisbawahi:** Perhatikan *host* di `spring.datasource.url`! Tadinya mungkin `localhost` atau `127.0.0.1`, tapi kalau sudah pakai Docker Network, kita panggil *container* MySQL dengan **nama *service* atau nama *container*-nya** (dalam kasus ini, `mysql` atau `mysql-container` jika tidak menggunakan *docker-compose*). Ini krusial banget agar aplikasi bisa menemukan *database*-nya di *network* Docker.

### Finalisasi dengan Docker Compose

Setelah semuanya aman, gue *push image*-nya ke Docker Hub dengan *best practice*-nya seperti berikut:

```bash
docker push dimasstudent/backend-java:v1
```
**Yang perlu digarisbawahi:** Format *push* ke Docker Hub itu penting. Biasanya pakai `docker push <username_docker_hub>/<nama_image>:<tag>`. Jadi pastikan `backend-java` itu nama *image* lo, dan `dimasstudent` itu *username* Docker Hub lo. Kalau `tag backend-java` itu buat bikin *tag* lokal, tapi *push*-nya harus spesifik ke *repo* di Docker Hub.

Setelah terdaftar di Docker Hub, tinggal kita panggil di Docker Compose.

Ini *config* Docker Compose saya:

```yaml
version: '3.8' # Lebih baik pakai versi terbaru, misalnya '3.8' atau '3.9'
services:
  mysql:
    image: mysql:8.0.33
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: mydb
      MYSQL_USER: dimas
      MYSQL_PASSWORD: rahasia
    volumes:
      - mysql_data:/var/lib/mysql # Menggunakan named volume, lebih baik untuk persistensi data
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql # Opsional, untuk inisialisasi database
    networks:
      - app-network # Memasukkan ke network yang sama

  backend-crud:
    image: dimasstudent/backend-java:v1 # Pastikan tag yang dipush sesuai
    container_name: backend-crud # Beri nama container agar lebih mudah diidentifikasi
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    environment: # Penting: Pastikan environment variables untuk koneksi DB ada di sini juga
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/mydb?allowPublicKeyRetrieval=true&useSSL=false
      SPRING_DATASOURCE_USERNAME: dimas
      SPRING_DATASOURCE_PASSWORD: rahasia
    networks:
      - app-network # Memasukkan ke network yang sama

networks: # Mendefinisikan network
  app-network:
    driver: bridge

volumes: # Mendefinisikan named volume
  mysql_data:
```

**Miskonsepsi 4: "Kalau sudah pakai Docker Compose, `docker network create` tidak perlu lagi."**
**Yang perlu digarisbawahi:** Betul! Kalau sudah pakai Docker Compose dan kamu mendefinisikan *network* di dalam `docker-compose.yaml` (seperti contoh di atas pada bagian `networks:` di bawah `services:`), Docker Compose akan **otomatis membuat *network*** itu untukmu saat kamu menjalankan `docker-compose up`. Jadi, kamu enggak perlu lagi *run* `docker network create` secara manual. Ini salah satu keunggulan Docker Compose, menyederhanakan orkestrasi *container*.

Semua *task* hari ini selesai, kelihatan cukup mudah dan lancar jaya padahal mah enggak! Banyak banget *trial and error* yang enggak gue tulis di sini. Tapi, dari situlah kita belajar, kan?

Gimana, ada *error* lain yang lo temuin pas *setup* kayak gini? Share di kolom komentar ya!
