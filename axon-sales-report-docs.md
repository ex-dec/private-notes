# Dokumentasi Aplikasi Axon Sales Report

## List of Content

1. [Introduction](#a-introduction)
2. [Setup Project](#b-setup-project)
3. [How To Use](#c-how-to-use)

## A. Introduction

Aplikasi ini dibuat untuk memudahkan kita dalam menggunakan data sales dari Axon. Dengan menggunakan data ini, kita bisa melakukan analisis dari data yang sudah ada. Yang mana nantinya kita bisa menggunakan hasil analisa tersebut dalam membuat keputusan untuk perusahaan. Pada aplikasi ini, kita akan menggunakan PHP sebagai API dan kong sebagai Gateway nya. Untuk gambaran dari platform yang sudah terbentuk adalah sebagai berikut

![Diagram Project](./asset/axon-sales-report-docs/1.%20diagram.png)

API yang kita buat untuk melakukan query akan kita implementasikan menggunakan bahasa PHP dan database yang kita siapkan menggunakan MySQL. Dari API tersebut, kita dapat mengakses melalui Kong sebagai API Gateway nya. Dari API Gateway tersebut, kita bisa mengakses mendapatkan hasil query yang sudah kita bentuk dari API tadi.

Project ini akan langsung bisa digunakan dan tinggal memasukkan query pada API apabila setup dari aplikasi ini sudah dilakukan dari awal hingga akhir.

## B. Setup Project

Untuk menggunakan aplikasi ini, kita perlu menyiapkan beberapa hal.

1. Docker. Merupakan implementasi dari platform ini. Apabila masih belum ada, silahkan klik [disini](#1-instalasi-docker)
2. Text Editor. Digunakan untuk menambahkan query yang akan dilakukan pada database. Kita bisa menggunakan VSCode agar lebih mudah, atau kita bisa menggunakan notepad agar tidak terlalu banyak instalasi yang dilakukan.
3. Web Browser. Digunakan untuk mendapatkan hasil query kita secara langsung. Kita bisa menggunakan browser seperti Microsoft Edge, Mozilla Firefox, atau Google Chrome.
4. Internet yang stabil. Karena nantinya kita akan menggunakan internet sebagai setup awal dari project ini.
5. Github. Kita akan menggunakan github untuk mendistribusikan project ini.

Lakukan step berikut untuk melakukan setup dari project ini.

### 1. Instalasi Docker

Instalasi docker dapat kita ikuti pada link berikut

1. [Setup Docker Windows](https://docs.docker.com/desktop/install/windows-install/)
2. [Setup Docker Mac](https://docs.docker.com/desktop/install/mac-install/)
3. [Setup Docker Linux](https://docs.docker.com/engine/install/)

Secara garis besar, kita akan membuat environment dalam docker ini agar bisa digunakan dalam komputer lokal kita. Untuk memastikan instalasi tersebut gunakan perintah ini dalam CMD/Terminal

    docker --version

Apabila perintah tersebut sudah dilakukan dengan benar, maka kita bisa menggunakan docker dari mana pun direktori yang kita gunakan.

### 2. Menjalankan Aplikasi

Berikut adalah langkah-langkah yang dapat kita ikuti dalam menjalankan aplikasinya.

1.  Silahkan clone repository dari link [berikut](https://github.com/ex-dec/axon-sales-report) menggunakan Github.

2.  Jalankan terminal ke dalam folder yang sebelumnya dilakukan cloning dan jalankan perintah berikut

        docker compose up -d

    Silahkan tunggu sampai perintah tersebut sudah selesai dilakukan. Untuk melakukan testing, Kita bisa mencoba akses panel admin dari kong menggunakan link [berikut](http://localhost:8002)

## C. How to Use

Dalam menggunakan project ini, ada beberapa file yang perlu diperhatikan yaitu

    /docker-compose.yml --> File ini digunakan untuk melakukan setup environment yang sudah kita miliki
    /src/Model/ApiModel.php --> File ini akan kita gunakan untuk menambahkan SQL ke API

Sebelumnya saat setup project, kita telah menggunakan docker-compose.yml untuk menjalankan setup dari project. Sekarang kita akan langsung mengimplementasikan SQL yang akan kita gunakan untuk query.

1.  Buka file ApiModel.php
2.  Tambahkan query dengan cara menambahkan method baru menggunakan format

        public function <nama method>()
        {
            return $this->query("<query>");
        }

3.  Setelah method tersebut sudah terbentuk, langsung lakukan testing pada browser. Kali ini saya akan menggunakan method berikut untuk melakukan testing

        public function example()
        {
            return $this->query("SHOW databases;");
        }

    Hasilnya akan menjadi seperti berikut

    ![Example method](./asset/axon-sales-report-docs/2.%20example%20of%20method.png)

Api sudah siap digunakan untuk implementasi dari query.
