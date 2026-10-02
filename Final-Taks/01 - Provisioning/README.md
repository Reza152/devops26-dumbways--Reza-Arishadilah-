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

![Terraform Version](./screenshots/01-Mengecek%20versi%20Terraform.png)

## 4. Struktur Folder Terraform

Terraform pada project ini menggunakan pendekatan modular untuk
memisahkan konfigurasi berdasarkan fungsi infrastructure.

Struktur directory Terraform:

```text
terraform/
├── main.tf
├── modules/
│   ├── compute/
│   │   ├── ec2.tf
│   │   ├── outputs.tf
│   │   ├── scripts/
│   │   │   ├── app.sh
│   │   │   ├── db.sh
│   │   │   └── gateway.sh
│   │   └── variables.tf
│   │
│   ├── network/
│   │   ├── outputs.tf
│   │   ├── routing.tf
│   │   ├── subnet.tf
│   │   ├── variables.tf
│   │   └── vpc.tf
│   │
│   └── security/
│       ├── outputs.tf
│       ├── security_group.tf
│       └── variables.tf
│
├── outputs.tf
├── providers.tf
├── terraform.tfvars
└── variables.tf
```
Struktur tersebut dibagi menjadi beberapa bagian:

| Directory/File | Fungsi |
|---|---|
| `main.tf` | Entry point konfigurasi Terraform dan pemanggilan module |
| `modules/compute/` | Konfigurasi EC2 server |
| `modules/network/` | Konfigurasi VPC, subnet, dan routing |
| `modules/security/` | Konfigurasi Security Group |
| `scripts/` | Script konfigurasi awal masing-masing server |
| `providers.tf` | Konfigurasi Terraform dan AWS Provider |
| `variables.tf` | Variable utama Terraform |
| `terraform.tfvars` | Nilai variable Terraform |
| `outputs.tf` | Output hasil provisioning |

Screenshot struktur Terraform:


