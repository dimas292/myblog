

---

# Setup MySQL Database + Spring Boot dalam Docker

**Oleh Dimas | 31 Mei 2025**

Halo! Masih bersama gue, Dimas. Kali ini gue akan melanjutkan series belajar tentang containerisasi menggunakan Docker, dengan project sederhana: backend Spring Boot + database MySQL.

---

## 🧱 Persiapan Awal

Sebelum kita mulai, pastikan kamu sudah menyiapkan:

- Sebuah project Spring Boot (Java) sederhana
- Maven sebagai build tool
- Docker terinstall di komputer kamu

Project yang gue buat adalah aplikasi CRUD sederhana dengan dependensi berikut:
- Spring Boot
- Lombok
- Spring Data JPA
- MySQL Connector

> ⚠️ Catatan: Pastikan kamu tidak menggantungkan diri pada IDE untuk test koneksi ke database — selalu uji di lingkungan Docker agar hasilnya akurat.

---

## 🚀 Jalankan Aplikasi di Local (Development)

Sebelum masuk ke Docker, pastikan aplikasi Spring Boot kamu bisa berjalan di lokal.

### 1. Build aplikasi menggunakan Maven:

```bash
mvn package
```

### 2. Jalankan aplikasi:

```bash
mvn spring-boot:run
```

Jika aplikasi berjalan lancar, lanjut ke langkah berikutnya.

---

## 🐳 Pull Image MySQL

Pertama, kita tarik image MySQL dari Docker Hub:

```bash
docker pull mysql:8.0.33
```

Gue pilih versi `8.0.33` karena stabil dan banyak dipakai di lingkungan produksi.

---

## 🔧 Jalankan Container MySQL

Sekarang kita jalankan container MySQL dengan konfigurasi yang cukup lengkap:

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

### Penjelasan:
| Bagian | Fungsi |
|--------|--------|
| `-e` | Set environment variable seperti password, nama database, dll |
| `-v` | Volume untuk menyimpan data secara persisten |
| `-d` | Menjalankan container di background |
| `-p` | Mapping port host ke container (misalnya `3306:3306`) |

> ✅ **Koreksi Konsep:**  
Flag `-p` bukan hanya "binding port", tapi **meneruskan port dari container ke host**, sehingga bisa diakses dari luar.

---

## 🔍 Akses MySQL dari Dalam Container

Kalau kamu ingin akses CLI MySQL dari dalam container:

```bash
docker exec -it <id-container> bash
mysql -u dimas -p
```

---

## 📦 Buat Dockerfile untuk Spring Boot App

Setelah aplikasi jalan di lokal, saatnya membuat Docker image untuk aplikasi Spring Boot kamu.

### Contoh `Dockerfile`:

```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/spring-boot-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

CMD [ "java", "-jar", "app.jar" ]
```

### Penjelasan:
| Baris | Fungsi |
|-------|--------|
| `FROM` | Base image Java yang digunakan |
| `WORKDIR` | Working directory di dalam container |
| `COPY` | Salin file `.jar` hasil build |
| `EXPOSE` | Beri tahu bahwa aplikasi akan listen di port 8080 |
| `CMD` | Perintah untuk menjalankan aplikasi |

> ✅ **Tips:** Kalau kamu pakai `openjdk:17-jdk-slim`, itu bagus untuk ukuran image yang kecil. Tapi kalau kamu ingin lebih ringkas lagi, gunakan `jre` bukan `jdk`.

---

## 🔌 Docker Network: Kenapa Penting?

Setelah semua siap, saatnya build dan jalankan aplikasi Spring Boot sebagai container.

Tapiii… ❗❗ Saat pertama kali gue coba, muncul error koneksi ke database. Ini karena **container backend dan container MySQL belum satu network**.

### Solusi: Buat network baru:

```bash
docker network create app-network
```

Lalu, jalankan ulang container MySQL dengan flag `--network`:

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

> ✅ **Konsep Penting:**  
Container di Docker harus berada di **network yang sama** untuk saling kenal lewat hostname. Tanpa ini, `localhost` di dalam container = dirinya sendiri, bukan host atau container lain.

---

## 📥 Jalankan Backend Spring Boot di Docker

Setelah build image aplikasi kamu:

```bash
docker build -t dimasstudent/backend-java:v1 .
```

Jalankan container backend dengan command:

```bash
docker run -d \
  --name backend-crud \
  -p 8080:8080 \
  --network app-network \
  dimasstudent/backend-java:v1
```

> ⚠️ **Konsep Salah:**  
Awalnya gue pikir setelah bikin network, langsung bisa konek. Padahal container backend harus punya konfigurasi yang benar agar bisa konek ke container MySQL.

---

## 🛠️ Konfigurasi Spring Boot untuk Docker

Agar aplikasi Spring Boot bisa konek ke MySQL yang berjalan di container, ubah konfigurasi berikut di `application.properties` atau `application.yml`:

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/mydb?allowPublicKeyRetrieval=true&useSSL=false
spring.datasource.username=dimas
spring.datasource.password=rahasia
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

> ⚠️ **Catatan Penting:**  
Hostname `mysql` hanya dikenali jika container backend dan MySQL berada di **network yang sama**. Jika kamu jalankan aplikasi dari lokal, ganti `mysql` menjadi `localhost`.

---

## 📁 Gunakan Docker Compose (Opsional tapi Direkomendasikan)

Agar lebih mudah, gue akhirnya beralih ke `docker-compose`. Ini file `docker-compose.yml` yang gue gunakan:

```yaml
version: '3'
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
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql

  backend-crud:
    image: dimasstudent/backend-java:v1
    container_name: backend-crud
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    restart: on-failure:5
```

> ✅ **Penjelasan Tambahan:**  
Meskipun `depends_on` membuat service backend menunggu MySQL di-start, **tidak menjamin MySQL sudah siap menerima koneksi**. Untuk memastikannya, kamu bisa tambahkan script `entrypoint` untuk delay atau retry logic.

---

## 🚀 Push ke Docker Hub (Opsional)

Kalau kamu ingin push image ke Docker Hub:

```bash
docker tag backend-java dimasstudent/backend-java:v1
docker push dimasstudent/backend-java:v1
```

---

## ✅ Kesimpulan

Langkah-langkah yang gue lakukan hari ini:

1. Jalankan aplikasi Spring Boot di lokal
2. Tarik dan jalankan MySQL container
3. Buat Dockerfile untuk Spring Boot
4. Jalankan container Spring Boot dengan Docker
5. Atasi masalah koneksi dengan Docker network
6. Sederhanakan setup dengan `docker-compose.yml`
7. Push image ke Docker Hub (opsional)

---

## 💡 Tips Tambahan

- **Gunakan profil (`dev`, `docker`) untuk bedakan konfigurasi lokal vs Docker**
- **Jangan gunakan `localhost` sebagai hostname di Docker**, gunakan nama service sesuai `docker-compose.yml`
- **Tambahkan retry logic** di entrypoint container agar backend menunggu MySQL siap

---

## 📝 Koreksi & Saran Penulisan

Beberapa kalimat di blog asli bisa diperbaiki agar lebih profesional dan mudah dipahami. Contoh:

> ❌ *"guee nyoba test lagi di projectnya namun database sekarang menggunakn container dan yapp success"*  
> ✅ *"Setelah mengganti database menjadi container, gue coba jalankan aplikasi dan ternyata berhasil."*

> ❌ *"guee kepikiran buat bikin migration dengan flyway tpi nnti malah gak kelar-kelar task guee jadi lanjut aja lahhh"*  
> ✅ *"Di tengah proses, gue sempat terpikir untuk menambahkan migrasi menggunakan Flyway, tapi gue skip dulu supaya fokus pada containerisasi."*

---

## 🎉 Penutup

Setup Spring Boot + MySQL dalam Docker bisa terasa rumit di awal, apalagi kalau baru belajar Docker. Tapi dengan pemahaman dasar tentang Docker network dan konfigurasi koneksi, semuanya bisa berjalan lancar.

> Jika kamu ingin, gue bisa bantu bikinkan template repository GitHub beserta Dockerfile, docker-compose, dan contoh migrasi dengan Flyway 😊

---

## 🔖 Tags

`Docker`, `Spring Boot`, `MySQL`, `Containerization`, `Belajar Docker`, `Backend Development`

--- 

Kalau kamu mau saya bantu buatkan halaman web atau PDF dari blog ini, tinggal kasih tahu aja ya 😄
