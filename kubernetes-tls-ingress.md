# Setup SSL on Ingress

Ingress bisa dikatakan sebagai gateway untuk meng expose layanan yang ada di cluster kubernetes. Ingress memiliki beberapa macam controller, namun yang paling populer adalah ingress-nginx-controller. Kali ini kita akan mengkonfigurasikan SSL pada ingress-nginx-controller agar bisa diakses melalui protokol HTTPS.

Sebelum kita melakukan setup untuk SSL nya, kita pastikan dulu ingress sudah bisa dijalankan melalui protokol HTTP. Apabila belum mengkonfigurasi platform pada ingress sama sekali, ikuti petunjuk ini dari awal. Apabila ingress sudah siap digunakan, silahkan [klik disini](#ssl).

## Setup Ingress

Untuk implementasinya, kita harus mengaktifkan ingress terlebih dahulu dengan menggunakan minikube.

    minikube addons enable ingress

![Enable ingress](./asset/kubernetes-tls-ingress/1.%20enable%20ingress.png)

Setelah ingress aktif, siapkan pod yang akan dibuka akses ke luar. Pastikan pod sudah berjalan dan sudah dijalankan dengan servicenya. Kita cukup menggunakan service dengan ClusterIP untuk melakukan expose port dari pod nya.

![Pods dan Service yang sudah dibuat](./asset/kubernetes-tls-ingress/2.%20pods%20dan%20service.png)

Setelah kedua hal tersebut sudah berjalan, sekarang kita siapkan ingress nya.

ubuntu-ingress.yaml

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ubuntu-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - host: ingress.local
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: ubuntu-service
                port:
                  number: 80

Terapkan konfigurasi tersebut dan konfigurasikan hostname agar bisa terbaca di host kita. Untuk host saya sendiri perlu mengkonfigurasi pada file /etc/hosts.

Disini saya akan menggunakan host 'ingress.local' sesuai dengan konfigurasi diatas.

/etc/hosts

![file konfigurasi /etc/hosts](./asset/kubernetes-tls-ingress/3.%20etc%20hosts.png)

Setelah itu kita coba akses melalui browser.

![percobaan menggunakan ingress](./asset/kubernetes-tls-ingress/4.%20percobaan%20ingress.png)

## SSL

Berikut adalah langkah-langkah yang bisa dilakukan untuk menggunakan ssl.

1.  Generate SSL

    Untuk melakukan generate kita perlu memastikan bahwa kita sudah menginstall openssl.

    ![openssl yang sudah terinstall](./asset/kubernetes-tls-ingress/5.%20openssl.png)

    Apabila sudah terinstall dengan baik, lakukan generate dengan peritah berikut.

        openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${KEY_FILE} -out ${CERT_FILE} -subj "/CN=${HOST}/O=${HOST}" -addext "subjectAltName = DNS:${HOST}"

    Kalau disesuaikan dengan kebutuhan saat ini, maka perintahnya menjadi seperti berikut

        openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout secret.key -out secret.crt -subj "/CN=ingress.local/O=ingress.local" -addext "subjectAltName = DNS:ingress.local"

    Hasilnya akan menjadi seperti berikut

    ![Hasil generate openssl](./asset/kubernetes-tls-ingress/6.%20generate%20ssl.png)

2.  Menambahkan key ke kubernetes

    Kubernetes memiliki layanan untuk menyimpan key yaitu secret. Pada secret, semua key disimpan dan bisa digunakan oleh berbagai macam layanan yang ada di kubernetes, salah satunya adalah ingress. Untuk menambahkan key tersebut, kita bisa menggunakan perintah berikut

        kubectl create secret tls ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE}

    Apabila disesuaikan dengan kebutuhan diatas, maka perintahnya akan menjadi seperti berikut

        kubectl create secret tls tls-ingress --key secret.key --cert secret.crt

    ![add secret](./asset/kubernetes-tls-ingress/7.%20add%20secret.png)

3.  Menambahkan secret pada ingress

    Secara default, ingress akan menggunakan koneksi tanpa tls. Maka dari itu, kita tambahkan secret yang sebelumnya sudah ditambahkan ke cluster kita. File konfigurasi ingress yang baru akan tampak seperti berikut

    ubuntu-ingress-ssl.yaml

        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: ubuntu-ingress
          annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
        spec:
          ingressClassName: nginx
          tls:
          - hosts:
            - ingress.local
            secretName: tls-ingress
          rules:
          - host: ingress.local
            http:
              paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: ubuntu-service
                    port:
                      number: 80

    Terapkan file tersebut dan hasilnya akan menjadi seperti berikut

    ![Hasil penerapan secret](./asset/kubernetes-tls-ingress/8.%20using%20secret.png)

    Ketika dilakukan percobaan pada browser, akan muncul warning. Hal tersebut tidak menjadi masalah karena kita sedang menggunakan platform ini di local dan sertifikat yang kita buat sebelumnya merupakan hasil generate secara private.

    ![Percobaan pada browser #1](./asset/kubernetes-tls-ingress/9.%20testing.png)

    ![Percobaan pada browser #2](./asset/kubernetes-tls-ingress/10.%20testing.png)

    Selamat!!! Ingress yang sebelumnya hanya berjalan pada protokol http, sudah bisa berjalan menggunakan protokol https dan kita bisa menggunakan protokol tersebut.
