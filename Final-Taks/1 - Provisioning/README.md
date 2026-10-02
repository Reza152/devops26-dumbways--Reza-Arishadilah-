# 01 - Provisioning

## 1. Tujuan

Tahap Provisioning bertujuan untuk menyiapkan server yang akan digunakan
untuk Final Project DevOps.

Provisioning dilakukan menggunakan:

- Terraform sebagai Infrastructure as Code (IaC) untuk membuat resource
  server di AWS.
- Ansible untuk melakukan konfigurasi awal server secara otomatis.

Pada tahap ini dibuat 3 server:

| Server | Fungsi | CPU | RAM |
|---|---|---:|---:|
| Gateway | NGINX Reverse Proxy dan Gateway | 1 CPU | 1 GB |
| App | Application Server | 2 CPU | 2 GB |
| DB | Database Server | 1 CPU | 1 GB |

Semua server menggunakan Ubuntu 24.04 LTS.

---

# 2. Environment

Provisioning dilakukan dari local machine menggunakan WSL Ubuntu.

Tools yang digunakan:

```text
Terraform
Ansible
AWS CLI
SSH
```
Versi yang digunakan:
terraform version

 
Terraform digunakan untuk membuat infrastructure AWS.
Kemudian mengecek versi Ansible:
```ansible --version```

 
Ansible digunakan untuk melakukan konfigurasi terhadap server yang
sudah dibuat oleh Terraform.

WSL Ubuntu
Git
