
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

---

Case 1: Setup Kubernetes Cluster on VPS/Cloud

Case 2: Deploy MQTT Broker (Mosquitto)

Case 3: Deploy Database PostgreSQL

Case 4: CI/CD Pipeline via GitLab Runner

Case 5: Zero Trust Architecture Simulation


📚 Referensi

K3s Official Documentation
Kubernetes Concepts
Helm Installation Guide