# Instalasi minikube pada Opensuse Linux

Minikube adalah salah satu environment kubernetes yang bisa berjalan di komputer lokal. Jadi kita bisa membuat cluster kubernetes sendiri di laptop kita.

Persyaratan minimal yang harus dipenuhi oleh komputer kita adalah

- Memiliki 2 Cpu atau lebih
- 2 GB ram yang bebas digunakan
- 20 GB sisa storage
- Koneksi internet
- Kontainer atau VM Manager, seperti Docker, QEMU, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware Fusion/Workstation

Berikut adalah langkah-langkah untuk melakukan instalasi minikube pada Opensuse Linux

1.  Lakukan pengecekan terhadap persyaratan minimal seperti pada persyaratan diatas

    ![Instalasi Docker](./asset/minikube-installation//1.%20instalasi%20docker.png)

    ![Spesifikasi persyaratan](./asset/1.%20system%20requirement.png)

2.  Download file rpm dari minikube dengan curl

        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm

3.  Lakukan instalasi file rpm yang sebelumnya sudah didownload

        sudo rpm -Uvh minikube-latest.x86_64.rpm

    ![Instalasi minikube](./asset/minikube-installation//2.%20Instalasi%20minikube.png)

4.  Setelah instalasi selesai, jalankan minikube agar kubernetes bisa dipakai. Untuk proses pertama kali, minikube akan melakukan download dependency dari kubernetes. Perintah untuk menjalankan minikube adalah sebagai berikut

        minikube start

    ![Instalasi minikube sukses](./asset/minikube-installation//2.%20Instalasi%20minikube%202.png)

5.  Setelah minkube sukses diinstall, maka kubernetes siap dipakai. Kubernetes bisa diakses melalui beberapa cara, bisa melalui dashboard yang sudah terinstall dengan minikube. Untuk menjalankan dashboard tersebut, kita bisa menggunakan perintah berikut

        minikube dashboard

    ![Tampilan dashboard kubernetes](./asset/minikube-installation//3.%20Dashboard%20kubernetes.png)

6.  Selain itu, kubernetes juga bisa diakses melalui terminal dengan menggunakan kubectl. Kubectl pada opensuse dapat diinstall melalui snap secara langsung dengan perintah berikut

        snap install kubectl --classic

    ![Instalasi kubectl](./asset/minikube-installation//4.%20instalasi%20kubectl.png)
