# Kubernetes DNS Management

## List of Content

1. [DNS Management](#a-dns-management)
2. [Namespacing](#b-namespacing)
3. [Pods Usage](#c-pods-usage)

## DNS Management

DNS Management adalah salah satu tools yang sangat diperlukan untuk menjangkau semua resource yang kita miliki. Dalam konteks ini, maka DNS Management diperlukan untuk memudahkan kita untuk mengakses semua resource dalam kubernetes cluster kita. Semua cluster dapat kita akses dengan mudah tanpa harus menggunakan IP Address.

Dalam kubernetes, DNS management sudah dihandle oleh CoreDNS. Untuk melakukan pengecekan apakah CoreDNS sudah berjalan atau belum, kita dapat menggunakan perintah berikut

    kubectl get deployments.apps --namespace kube-system coredns

Secara defaul, domain yang bisa langsung digunakan dalam cluster tersebut adalah

    cluster.local

dengan opsi sub domain sebagai berikut

- \<service>.\<namespace>.cluster.local
- \<ip-pods>.\<namespace>.pod.cluster.local

Dalam implementasinya sendiri, file konfigurasi dns dari masing-masing pod sudah ada default value pada file /etc/resolv.conf seperti berikut

    nameserver <ip pod CoreDNS>
    search default.svc.cluster.local svc.cluster.local cluster.local
    options ndots:5

Dengan konfigurasi tersebut, kita bisa menuliskan secara langsung resource mana yang ingin kita tuju.

## Namespacing

Dalam kubernetes, terdapat metode pemberian tanda pada suatu resource yaitu dengan menggunakan namespace. Namespace disini akan memberikan tanda pada suatu resource agar resource tersebut tidak dapat diakses secara langsung oleh resource dengan namespace yang lain. Misal pada default dari instalasi kubernetes, terdapat beberapa namespace yang secara otomatis akan dibuat. Untuk melihatnya, kita bisa menggunakan perintah berikut

    kubectl get namespaces

![Namespace default yang terbentuk dengan instalasi kubernetes](./asset/kubernetes-dns-management/1.%20checking%20namespace.png)

Berdasarkan keterangan pada bab sebelumnya, kita bisa melihat bahwa beberapa resource harus didefinisikan namespacenya.

## Pods Usage

Sekarang kita akan coba mengakses resource yang lain menggunakan dns yang sebelumnya sudah ada.

Resource yang saya buat disini adalah

Pod 1:

- image : ubuntu
- name : ubuntu
- service : ubuntu-service

Pod 2:

- image : ubuntu
- name : ubuntu2
- service : -

Pod 1 akan berperan sebagai target dari Pod 2, yang mana kedua pod tersebut bisa digunakan sebagai representasi dari jaringan yang ada di dalam cluster.

![Pod 1](./asset/kubernetes-dns-management/2.%20pod%201%20describe.png)

![Pod 2](./asset/kubernetes-dns-management/3.%20pod%202%20describe.png)

Sekarang kita cek juga service yang sebelumnya sudah dipakai oleh Pod 1

![Ubuntu service](./asset/kubernetes-dns-management/4.%20service.png)

Dari keterangan diatas, maka dapat dibuat hipotesis bahwa domain dari resource tersebut adalah

- Pod 1 : 10-244-0-134.default.pod.cluster.local
- Pod 2 : 10-244-0-143.default.pod.cluster.local
- ubuntu-service : ubuntu-service.default.cluster.local

Sekarang kita jalankan terminal dari pod 2 dan lakukan akses ke Pod 1 dengan menggunakan domain tersebut

![Testing direct ke pod](./asset/kubernetes-dns-management/5.%20testing%20pod%201.png)

Akses menggunakan domain yang dimiliki oleh pod 1 bisa langsung dilakukan. Selanjutnya, kita akan melakukan akses melalui service dari Pod 1.

![Testing dengan service](./asset/kubernetes-dns-management/6.%20testing%20service.png)
