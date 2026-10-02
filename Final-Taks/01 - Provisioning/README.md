# 01 - Provisioning

## 1. Tujuan

Tahap Provisioning bertujuan untuk menyiapkan infrastructure server
yang akan digunakan untuk Final Project DevOps.

Provisioning dilakukan menggunakan:

- Terraform sebagai Infrastructure as Code (IaC)
- AWS sebagai cloud provider
- Ansible untuk konfigurasi server
- SSH untuk akses ke server

Pada tahap ini dibuat 3 server:

| Server | Fungsi | CPU | RAM |
|---|---|---:|---:|
| Gateway | Gateway / Reverse Proxy | 1 CPU | 1 GB |
| App | Application Server | 2 CPU | 2 GB |
| DB | Database Server | 1 CPU | 1 GB |

Semua server menggunakan Ubuntu 24.04 LTS.

AWS Region yang digunakan:

```text
ap-southeast-3
```
Region tersebut merupakan AWS Jakarta.

## 2. Environment
Provisioning dilakukan dari local machine menggunakan WSL Ubuntu.
Tools yang digunakan:

```
Terraform v1.16.2
Ansible Core 2.20.1
AWS CLI
SSH
Git
WSL Ubuntu
```
Terraform digunakan untuk membuat infrastructure AWS.
Ansible digunakan untuk melakukan konfigurasi awal terhadap server
yang sudah dibuat oleh Terraform.

## 3. Terraform
Terraform digunakan sebagai Infrastructure as Code (IaC).
Dengan Terraform, infrastructure AWS dapat didefinisikan menggunakan
file konfigurasi sehingga proses pembuatan server, network, security
group, dan resource lainnya dapat dilakukan secara terstruktur.

### 3.1 Mengecek Versi Terraform
Sebelum melakukan provisioning, dilakukan pengecekan versi Terraform
pada local machine.
Perintah:
```terraform version```

Terraform yang digunakan:
```Terraform v1.16.2```
