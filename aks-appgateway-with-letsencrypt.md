# AKS Appgateway with ssl from LetsEncrypt

## List of Content

1. [Introduction](#introduction)
2. [Preparing Environment](#preparing-environment)
3. [Step by Step](#step-by-step)

## Introduction

Dalam mengakses sebuah aplikasi yang sudah terdeploy di dalam cluster kubernetes, kita dapat menggunakan salah satu dari fitur yang sudah disediakan oleh kubernetes yaitu melalui LoadBalancer, Gerbang aplikasi berbasis HTTP (ingress), atau Gateway API. Tentu saja masing-masing fitur memiliki kelebihan dan kekurangan. Untuk lebih lanjut mengenai hal tersebut sebelumnya sudah sempat dibahas. Untuk sekarang kita lebih fokus terhadap keamanan dalam menggunakan ingress.

Aspek yang perlu menjadi perhatian ketika kita menggunakan ingress sebagai sebuah gateway adalah akses terhadap ingress itu sendiri. Karena pada dasarnya, ingress bertindak sebagai reverse proxy dari layanan yang ada di belakangnya. Berikut adalah gambaran dari reverse proxy itu sendiri.

![Konsep reverse proxy yang digunakan oleh ingress](./asset/aks-appgateway-with-letsencrypt/1.%20reverse%20proxy%20concept.png)

Dengan adanya konsep tersebut, maka kubernetes menggunakan konsep yang sama yaitu dengan menggunakan ingress yang merupakan LoadBalancer untuk melakukan provisioning dari layanan yang ada di belakangnya.

Keamanan yang dapat diimplementasikan dalam bentuk tersebut adalah penambahan SSL ketika kita akan mengakses ingress. Karena konsepnya hampir sama dengan web server yang menyediakan konten, maka kita akan menggunakan metode itu juga dalam pemasangan SSL.

Dalam percobaan ini kita akan menggunakan platform dari azure yaitu AKS (Azure Kubernetes Cluster).

Azure Kubernetes Service itu sendiri adalah layanan dari untuk orkestrasi kubernetes yang disediakan oleh azure. Kita bisa melakukan deployment aplikasi yang sebelumnya sudah kita buat menggunakan kubernetes menggunakan layanan cloud dari azure.

Untuk mengakses aplikasi yang ada di dalam AKS, kita membutuhkan Load Balancer dari Azure yaitu Application Gateway.

Application Gateway dari azure mampu melakukan load balancing terhadap cluster dari AKS sampai layer 7. Yang mana kemampuan ini juga dapat dikombinasikan dengan ingress dari kubernetes.

Sebelumnya kita sudah membuat environment seperti berikut di lokal kita

![Kubernetes orchestration](./asset/aks-appgateway-with-letsencrypt/2.%20diagram.png)

Selanjutnya kita akan mengimplementasikan environment tersebut pada AKS dengan penambahan SSL. Platform yang akan membantu kita dalam membuat SSL Cert adalah lets encrypt karena bisa digunakan secara bebas.

## Preparing Environment

Sebelum nya kita siapkan terlebih dahulu satu set deployment dari environment sebelumnya.

php-kubernetes.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: php-deployment
      labels:
        app: php
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: php
      template:
        metadata:
          labels:
            app: php
        spec:
          containers:
          - name: php
            image: exdec/php-image:latest
            ports:
            - containerPort: 80
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: php-service
    spec:
      selector:
        app: php
      ports:
        - protocol: TCP
          port: 80
    ---
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: php-ingress
      annotations:
        kubernetes.io/ingress.class: azure/application-gateway
    spec:
      rules:
      - host: aks.krisnawahyu.my.id
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: php-service
                port:
                  number: 80

Jalankan file tersebut dan lakukan testing ke link yang sebelumnya sudah ada.

![Testing ingress non ssl](./asset/aks-appgateway-with-letsencrypt/3.%20testing%20ingress.png)

## Step by Step

Sekarang kita akan melakukan penambahan SSL pada environment yang sudah terbentuk. Sebelumnya mari kita pahami dulu konsep yang diterapkan pada pemasangan SSL ini.

Pada pemasangan SSL ini, kita akan menggunakan plugin dari [cert manager](cert-manager.io) sebagai controller dalam melakukan request certificate pada [letsencrypt](letsencrypt.org).

![Konsep request ssl dari cert-manager](./asset/aks-appgateway-with-letsencrypt/4.%20%20konsep%20cert%20manager.svg)

cert manager akan melakukan request ke issuer yang sudah didefine, salah satu issuer yang bisa dipakai adalah letsencrypt. Protokol yang akan digunakan oleh cert-manager dalam melakukan request adalah menggunakan protokol ACME.

ACME sendiri adalah sebuah protokol yang menggunakan sistem otomasi dalam melakukan generate certificate.

![ACME concept](./asset/aks-appgateway-with-letsencrypt/acme%20protocol.jpg)

Jadi semua proses dalam melakukan generate, pengecekan, dan pembaruan dilakukan secara otomatis. Tentunya, layanan ini free dan bebas untuk digunakan selama server kita memiliki IP Public dan bisa terkoneksi dengan internet.

Untuk ada beberapa langkah yang harus dilalui dalam mengimplementasikan hal ini.

1. Lakukan pemasangan cert-manager pada cluster kita.

    Untuk melakukan pemasangan, cert-manager telah memberikan setup yang bisa kita pakai dan akan melakukan setup secara otomatis.

        kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml

    Setelah setup tersebut selesai, maka akan terbentuk namespace baru dengan nama cert-manager serta pod yang akan digunakan untuk melakukan manajemen certificate.

2. Konfigurasikan issuer dengan lets encrypt

    Lakukan konfigurasi untuk issuernya. Terapkan file konfigurasi berikut sesuai dengan kebutuhan

    php-issuer.php

        apiVersion: cert-manager.io/v1
        kind: ClusterIssuer
        metadata:
          name: letsencrypt
        spec:
          acme:
            email: <email address>
            server: https://acme-v02.api.letsencrypt.org/directory
            privateKeySecretRef:
              name: tls-secret
            solvers:
              - http01:
                   ingress:
                      class: <ingress class>

    Setelah itu apply dan pastikan konfigurasi tersebut telah diterapkan dengan perintah berikut

        kubectl get ClusterIssuer

    Setelah itu ubah konfigurasi ingress sebelumnya menjadi seperti ini

    php-ingress-ssl.yaml

        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: php-ingress
          annotations:
            kubernetes.io/ingress.class: azure/application-gateway
            cert-manager.io/cluster-issuer: letsencrypt
        spec:
          tls:
          - hosts:
            - aks.krisnawahyu.my.id
            secretName: tls-secret
          rules:
          - host: aks.krisnawahyu.my.id
            http:
              paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: php-service
                    port:
                      number: 80

    Terapkan konfigurasi tersebut. Ketika konfigurasi tersebu telah diterapkan, maka akan terjadi proses request ke letsencrypt dan melakukan challenge HTTP untuk mendapatkan certificate yang dibutuhkan.

    Untuk melakukan pengecekan terhadap certificate yang telah dibuat, kita bisa menggunakan perintah berikut

        kubectl get secret

Setelah proses request selesai maka hasilnya akan menjadi seperti berikut

![Testing ssl](./asset/aks-appgateway-with-letsencrypt/5.%20testing%20ingress%20ssl.png)
