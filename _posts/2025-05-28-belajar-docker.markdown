---
layout: post
title:  "Belajar Docker Dengan Study Case Todo-List"
date:   2025-05-28 20:00:00 +0700
categories: docker belajar
---

---

# 🐳 Catatan Belajar Docker Hari Ini  
*by Dimas | 28 Mei 2025*

Hari ini saya fokus mempelajari Docker dari dasar hingga implementasi langsung ke project nyata. Berikut adalah rangkuman perjalanan belajar saya hari ini:

---

## 🧪 1. **8 Perintah Dasar Docker**

Sebagai permulaan, saya pelajari beberapa perintah dasar Docker yang sering digunakan sehari-hari:

| Perintah | Fungsi |
|--------|--------|
| `docker ps` | Melihat container yang sedang aktif |
| `docker ps -a` | Melihat semua container (aktif & berhenti) |
| `docker run [OPTIONS] IMAGE` | Menjalankan container baru |
| `docker stop <container_id>` | Menghentikan container |
| `docker start <container_id>` | Memulai kembali container |
| `docker rm <container_id>` | Menghapus container |
| `docker images` | Melihat daftar image lokal |
| `docker rmi <image_name>` | Menghapus image |

### Contoh:
```bash
docker run -d -p 5432:5432 postgres
```

Penjelasan opsi:
- `-d` → menjalankan container di background (detached mode)
- `-p 5432:5432` → mapping port host ke container
- `--name` → memberi nama pada container

---

## 🔍 2. **Memahami Opsi Penting Docker**

Saya juga belajar bagaimana opsi seperti `-p`, `-d`, `-f`, dan lainnya bekerja:

- `-p` → Port binding antara host dan container
- `-d` → Detach container dari terminal
- `-f` → Menentukan file spesifik seperti `Dockerfile` atau `docker-compose.yml`
- `--network` → Menyambungkan container ke jaringan tertentu

Contoh:
```bash
docker run -d --network my-network --name db postgres
```

---

## ⚙️ 3. **Troubleshooting Container dengan `docker logs`**

Ketika ada error atau ingin melihat output aplikasi di dalam container, saya gunakan perintah:

```bash
docker logs <container_id>
```

Ini sangat membantu untuk debugging runtime aplikasi.

---

## 🖥️ 4. **Virtualisasi vs Docker**

Saya juga membandingkan Docker dengan virtual machine (VM):

| Aspek | Docker | Virtual Machine |
|------|--------|------------------|
| Abstraksi | Pada level sistem operasi | Pada level perangkat keras |
| Resource | Lebih ringan | Lebih berat |
| Startup Time | Cepat | Lambat |
| Isolasi | Menggunakan namespace & cgroups | Menggunakan hypervisor |

Dengan Docker, layer abstraksi lebih tinggi dan overhead lebih rendah.

---

## 🛠️ 5. **Project Study Case: Todo List dengan Vue.js + Golang**

Untuk menerapkan teori, saya buat aplikasi sederhana **Todo List** dengan:
- **Frontend**: Vue.js
- **Backend**: Golang
- **Database**: PostgreSQL

### Langkah-langkah:
1. Jalankan database Postgres menggunakan Docker:
   ```bash
   docker run -d --name pgdb -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres
   ```

2. Hubungkan backend dengan database menggunakan network:
   ```bash
   docker network create todo-network
   docker run -d --network todo-network --name backend dimas/todo-backend
   ```

3. Jalankan frontend dan hubungkan ke backend:
   ```bash
   docker run -d --network todo-network --name frontend -p 8080:8080 dimas/todo-frontend
   ```

---

## 📦 6. **Membuat Dockerfile untuk Frontend & Backend**

Saya membuat `Dockerfile` untuk kedua bagian aplikasi:

### Backend (`Dockerfile`):
```Dockerfile
FROM golang:1.22-alpine
WORKDIR /app
COPY . .
EXPOSE 8080
CMD ["go", "run", "main.go"]
```

### Frontend (`Dockerfile`):
```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install && npm run build
EXPOSE 8080
CMD ["npm", "run", "serve"]
```

Setelah itu, saya build dan push image ke Docker Hub:

```bash
docker build -t dimas/todo-backend .
docker push dimas/todo-backend
```

---

## 🧱 7. **Membuat `docker-compose.yml`**

Agar tidak ribet dengan banyak perintah, saya beralih ke Docker Compose. Di sini saya sadar bahwa:

> **Tidak perlu membuat network manual**, karena Docker Compose otomatis mengisolasi service dalam satu network internal.

Contoh `docker-compose.yml`:

```yaml
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    ports:
      - "8081:8080"
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "8080:8080"
    depends_on:
      - backend
```

---

## ☁️ 8. **Upload ke Docker Hub**

Setelah build image, saya upload ke Docker Hub:

```bash
docker login
docker push dimas/todo-frontend
docker push dimas/todo-backend
```

Lalu update `docker-compose.yml` agar menggunakan image dari registry:

```yaml
  frontend:
    image: dimas/todo-frontend
```

---

## 🎯 9. **Rencana Selanjutnya**

Setelah sukses dengan Vue + Go, rencana saya selanjutnya adalah:

- Membuat aplikasi serupa dengan **Java + Maven + Spring Boot**
- Mempelajari deployment ke VPS/EC2
- Mengatur reverse proxy dengan Nginx
- Memasang SSL dengan Let’s Encrypt
- Mendaftarkan domain
- Scale aplikasi
- Mempelajari concurrency, thread, dan message broker (Redis/Kafka)

---

## 💡 Refleksi

Belajar Docker hari ini cukup intensif, terutama saat lupa command Linux panjang atau saat troubleshooting container. Alhamdulilah akhirnya bisa lancar, bahkan sudah bisa deploy aplikasi full-stack dengan Docker Compose.

Semoga ilmu ini terus berkembang dan bisa diterapkan di proyek nyata.

---

**#belajardocker #dimasblog #devopsjourney #dockercompose #vuejs #golang**

--- 

Kalau kamu ingin saya bantu tambahkan gambar arsitektur atau diagram Docker-nya, saya bisa bantu juga! 😊




