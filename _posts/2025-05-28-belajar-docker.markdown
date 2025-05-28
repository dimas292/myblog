---
layout: post
title:  "Belajar Docker Dengan Study Case Todo-List"
date:   2025-05-28 20:00:00 +0700
categories: docker belajar
---

---

Hari ini saya ingin berbagi cerita tentang perjalanan saya dalam mempelajari Docker. Ini bukan sekadar belajar teori, tapi langsung saya praktikkan dalam sebuah studi kasus nyata: membangun aplikasi to-do list dengan frontend Vue.js dan backend Golang.

Awalnya, saya belajar 8 perintah dasar Docker, seperti bagaimana cara mengecek kontainer yang sedang aktif. Kemudian, saya mendalami fungsi dari flag-flag seperti -p, -d, -f, dan lain sebagainya. Setelah itu, yang tak kalah penting adalah memahami langkah-langkah ketika kontainer kita mengalami troubleshoot, contohnya penggunaan docker logs <id-container>. Saya juga mencoba memahami perbedaan antara virtualisasi dan Docker, terutama bagaimana abstraksi layer bekerja dengan Docker dan VM.

Nah, di studi kasus aplikasi to-do list saya, saya mempraktikkan bagaimana Docker Network bekerja, dengan menghubungkan backend Golang saya menggunakan flag --network ke dalam database PostgreSQL yang sudah berjalan kontainernya. Saya juga belajar alur pembuatan aplikasi dengan Docker dari sisi development, production, CI/CD pipeline, orkestrasi, hingga mengunggah ke Docker Registry.

Setelah semua teori saya pelajari, langkah selanjutnya adalah membuat Dockerfile di proyek frontend dan backend saya. Dari cara memilih base Alpine yang ringan, membuat directory, mengekspos port, menentukan CMD, hingga bagaimana aplikasi berjalan ketika Docker dijalankan. Setelah semua selesai, saya mencoba membuat Docker images dari proyek saya. Di tengah-tengah pembelajaran, saya juga sempat sedikit lupa mengenai beberapa command Linux yang panjang.

Selanjutnya, saya membuat Docker Compose. Di sini saya memahami bahwa dengan Docker Compose, kita tidak perlu lagi membuat network secara manual. Mengapa? Karena di dalam Docker Compose, layanan-layanan sudah terisolasi dalam satu network yang sama. Lalu, saya membuat Docker repository dan mengunggahnya ke Docker Hub. Terakhir, saya memperbarui Docker Compose yang tadinya build menjadi menggunakan image yang sudah saya unggah. Dan hasilnya? Aplikasi saya berjalan dengan lancar!

Sampai sini, inilah perjalanan saya hari ini dalam belajar Docker dengan studi kasus aplikasi to-do list. Rencananya, saya akan mencoba membuat hal serupa dengan Java, Maven, dan Spring Boot. Dan yang belum saya ulik masih banyak sekali, seperti deploy ke VPS/EC2, menggunakan Nginx reverse proxy, memasang SSL, mendaftarkan ke domain, scale up apps, concurrency, thread, message broker, dan masih banyak lagi.

Kalian bisa komen dibawah yaa dengan akun github kalian 
