# Case 1: Setup Kubernetes Cluster on VPS/Cloud

## 🎯 Tujuan
Menyiapkan lingkungan Kubernetes yang berjalan di atas **VPS/Cloud Instance**, yang siap digunakan untuk proses **deployment aplikasi web containerized**.

## 🧰 Lingkungan & Prasyarat

### 1. Spesifikasi VPS/Cloud
Gunakan minimal konfigurasi berikut:
- **OS**: Ubuntu 22.04 LTS (64-bit)
- **RAM**: 2 GB atau lebih
- **vCPU**: 2 core atau lebih
- **Disk**: 20 GB+
- **Akses**: SSH dengan user `root` atau user dengan hak `sudo`

> Catatan: Anda dapat menggunakan provider seperti Google Cloud, AWS Lightsail, Linode, Vultr, atau Compute Engine dari GCP.  
> Pastikan port `6443`, `10250`, dan `80/443` terbuka di firewall (terutama jika nanti menggunakan Ingress Controller).

### 1.1 Infrastruktur
Pastikan kamu memiliki minimal:
- 1 VM untuk **control node** (tempat menjalankan Ansible)
- 1–2 VM tambahan sebagai **worker node**

| Peran | OS | IP Contoh | Catatan |
|--------|----|------------|----------|
| Control Node | Ubuntu 22.04 | `10.0.0.1` | Tempat Ansible diinstall |
| Worker Node 1 | Ubuntu 22.04 | `10.0.0.2` | Join ke cluster |
| Worker Node 2 | Ubuntu 22.04 | `10.0.0.3` | Join ke cluster |


---

## 🧩 2. Instalasi Kubernetes dengan K3s

K3s merupakan distribusi Kubernetes ringan yang cocok untuk VPS dengan resource terbatas.  
Berikut langkah-langkah instalasinya:

### 2.1 Update dan Persiapan Sistem
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl vim git net-tools

### 2.2 Nonaktifkan Swap

Kubernetes tidak mendukung swap.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

### 2.3 Install K3s (Single Node)

Gunakan perintah berikut untuk instalasi K3s di node tunggal:

```bash
curl -sfL https://get.k3s.io | sh -

Setelah instalasi selesai, pastikan status service k3s aktif:
```bash
sudo systemctl status k3s


### 2.4 Verifikasi Instalasi

Cek node dan komponen Kubernetes:

```bash
sudo kubectl get node
sudo kubectl get pods -A

Jika output menunjukkan status Ready pada node, maka cluster sudah berhasil diinstal.


## ⚙️ 3. Konfigurasi kubectl
Agar kubectl bisa digunakan tanpa sudo, jalankan:
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config

Uji koneksi:
```bash
kubectl get nodes


## ⚙️ 4. Tambahkan repository Ansible

```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible

### 4.1 Verifikasi instalasi
```bash
ansible --version

Output contoh:
```
ansible [core 2.17.2]
  config file = /etc/ansible/ansible.cfg
  python version = 3.10.12
```



