---
layout: post
title:  "Setup Mysql Database + Spring Boot"
date:   2025-05-31 12:00:00 +0700
categories: docker belajar
---

Oleh Dimas | 31 Mei 2025


Hallo masih dengan saya dimas, kali ini saya akan melanjutkan series belajar saya tentang containerization dan untuk projectnya 
menggunakan Spring boot backendnya + Mysql untuk databasenya.

Pertama-tama kita pull images mysql dari docker hub, disini versi yang saya gunakan mysql:8.0.33

```bash

docker pull mysql:8.0.33

```

kemudian membuat koneksi agar databasenya terisolate dengan project backend dengan nama "app-network"

```bash

docker network create app-network 

```
Berikut adalah cara menjalankan Mysql di docker 

```bash

docker run --name mysql-container \
> -e MYSQL_ROOT_PASSWORD=root \
> -e MYSQL_DATABASE=mydb \
> -e MYSQL_USER=dimas \
> -e MYSQL_PASSWORD=rahasia \
> -v mysql_data:/var/lib/mysql \
> -d \
> -p 3306:3306 \
> --network app-network \
> mysql:8.0.33

```

Mungkin kalau kalian bingun apa yang guee lakukan diatas, ini bisa jadi gambaran

![visualization](../images/net.png)

