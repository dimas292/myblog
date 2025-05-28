---
layout: post
title:  "Belajar Docker Pertamaku"
date:   2025-05-28 20:00:00 +0700
categories: docker belajar
---

Hari ini saya mulai belajar Docker. Ini adalah langkah awal untuk memahami containerization dan bagaimana cara aplikasi dijalankan dalam lingkungan yang konsisten, cepat, dan portabel.

Docker memungkinkan developer untuk membungkus aplikasi beserta seluruh dependensinya ke dalam sebuah unit yang disebut _container_. Ini sangat berguna untuk deployment dan pengembangan lintas platform.

Untuk memulai, saya menginstal Docker Desktop dan menjalankan perintah berikut untuk mengecek versi:

```bash
docker --version

docker run hello-world

# Gunakan image dasar
FROM node:18-alpine

# Set direktori kerja
WORKDIR /app

# Salin file dan install dependencies
COPY package*.json ./
RUN npm install

# Salin semua file dan jalankan app
COPY . .
CMD ["node", "app.js"]

```




