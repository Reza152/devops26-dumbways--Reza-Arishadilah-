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

![Terraform Version](./screenshots/11-folder%20terraform.png)

## 5. AWS Provider
Terraform membutuhkan AWS Provider agar dapat berkomunikasi dengan
AWS.
File yang digunakan:
```providers.tf```

Konfigurasi provider digunakan untuk menentukan AWS sebagai provider
dan region yang digunakan.
Contoh konfigurasi:
```
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```
Region yang digunakan:
```ap-southeast-3```
Region tersebut merupakan AWS Jakarta.

## 6. Variable Terraform
File:
```variables.tf```

digunakan untuk mendefinisikan variable yang digunakan oleh
Terraform.
Contoh:
```
variable "aws_region" {
  description = "AWS Region Jakarta"
  type        = string
  default     = "ap-southeast-3"
}

variable "allowed_ssh_cidr" {
  description = "CIDR yang diizinkan untuk SSH"
  type        = string
}
```
Variable digunakan agar konfigurasi Terraform lebih mudah dikelola
dan nilai tertentu tidak perlu ditulis berulang kali.

## 7. Network Infrastructure
Terraform digunakan untuk membuat network AWS yang digunakan oleh seluruh server.
Module network pada project ini terdiri dari:

| File | Fungsi |
|---|---|
| `vpc.tf` | Konfigurasi VPC |
| `subnet.tf` | Konfigurasi subnet |
| `routing.tf` | Konfigurasi Internet Gateway dan routing |
| `variables.tf` | Variable yang digunakan module network |
| `outputs.tf` | Output dari resource network |

Struktur module:
```
modules/
└── network/
    ├── outputs.tf
    ├── routing.tf
    ├── subnet.tf
    ├── variables.tf
    └── vpc.tf
```
Network tersebut digunakan oleh tiga server:

                         Internet
                            |
                    Internet Gateway
                            |
                           VPC
                            |
                     Public Subnet
                            |
          ┌─────────────────┼─────────────────┐
          │                 │                 │
       Gateway             App               DB
          │                 │                 │
    10.0.1.108         10.0.1.215        10.0.1.216

Ketiga server berada dalam VPC yang sama dan menggunakan subnet yang telah dibuat oleh Terraform.
    
### 8. VPC
VPC digunakan sebagai jaringan virtual untuk server yang dibuat
di AWS.
Konfigurasi VPC:
```
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags = {
    Name = "VPC"
  }
}
```
CIDR VPC:
```10.0.0.0/16```
VPC menjadi jaringan utama yang digunakan oleh Gateway, App, dan DB.

### 9. Subnet
Subnet digunakan untuk membagi jaringan VPC menjadi beberapa jaringan berdasarkan kebutuhan server.
Pada konfigurasi Terraform, terdapat dua subnet, yaitu Public Subnet dan Private Subnet.
Konfigurasi `subnet.tf`

```
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = true

  tags = {
    Name    = "${var.project_name}-public-subnet"
    Project = var.project_name
  }
}

resource "aws_subnet" "private" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = false

  tags = {
    Name    = "${var.project_name}-private-subnet"
    Project = var.project_name
  }
}
```
Konfigurasi Subnet
| Subnet | CIDR | Public IP |
|---|---|---|
| Public Subnet | `10.0.1.0/24` | Enabled |
| Private Subnet | `10.0.2.0/24` | Disabled |

Pada konfigurasi tersebut:
- Public Subnet 10.0.1.0/24 menggunakan map_public_ip_on_launch = true, sehingga instance yang dibuat pada subnet tersebut dapat memperoleh Public IP secara   otomatis.
- Private Subnet 10.0.2.0/24 menggunakan map_public_ip_on_launch = false, sehingga Public IP tidak diberikan secara otomatis.
- Kedua subnet berada pada VPC yang sama.
- Availability Zone ditentukan melalui variable var.availability_zone.
- Nama subnet menggunakan var.project_name.

Private IP Server
Server yang digunakan dalam deployment saat ini berada pada Public Subnet 10.0.1.0/24:

| Server | Private IP | Subnet |
|---|---|---|
| Gateway | `10.0.1.108` | Public |
| App | `10.0.1.215` | Public |
| DB | `10.0.1.216` | Public |

Private Subnet 10.0.2.0/24 dibuat oleh Terraform tetapi belum digunakan oleh ketiga server pada deployment saat ini.


### 10. Internet Gateway
Internet Gateway digunakan agar VPC dapat berkomunikasi dengan internet.
Konfigurasi terdapat pada:
```modules/network/routing.tf```
```
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name    = "${var.project_name}-igw"
    Project = var.project_name
  }
}
```
Internet Gateway dihubungkan dengan VPC yang telah dibuat.

### 11. Route Table
Route Table digunakan untuk menentukan jalur traffic jaringan pada VPC.
Pada konfigurasi terdapat Public Route Table dan Private Route Table.
Public Route Table
Public Route Table digunakan oleh Public Subnet `10.0.1.0/24`.
```
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name    = "${var.project_name}-public-rt"
    Project = var.project_name
  }
}
```
Route:
| Destination | Target |
|---|---|
| `0.0.0.0/0` | Internet Gateway |

Route tersebut digunakan untuk mengarahkan traffic dari Public Subnet menuju Internet Gateway.
Public Route Table dihubungkan dengan Public Subnet:
```
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```
Private Route Table
Private Route Table digunakan oleh Private Subnet `10.0.2.0/24`.

```
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name    = "${var.project_name}-private-rt"
    Project = var.project_name
  }
}
```
Private Route Table dihubungkan dengan Private Subnet:
```
resource "aws_route_table_association" "private" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private.id
}
```
Pada konfigurasi saat ini, Private Route Table tidak memiliki route 0.0.0.0/0 menuju Internet Gateway.

### 12. Security Group
Security Group digunakan sebagai firewall virtual untuk mengatur traffic yang masuk dan keluar dari EC2.
Konfigurasi Security Group terdapat pada:
```modules/security/security_group.tf```

Akses SSH menggunakan port:
```3333```

CIDR yang digunakan untuk akses SSH:
```119.235.222.96/32```

Port yang digunakan oleh server:
| Server | Port | Fungsi |
|---|---:|---|
| Gateway | 3333 | SSH |
| Gateway | 80 | HTTP |
| Gateway | 443 | HTTPS |
| App | 3333 | SSH |
| App | 80 | HTTP |
| App | 3000 | Backend |
| App | 3001 | Frontend |
| App | 8080 | Application |
| DB | 3333 | SSH |
| DB | 5432 | PostgreSQL |

PostgreSQL pada DB digunakan oleh Application Server melalui private network.

### 13. SSH Key
SSH key digunakan untuk melakukan koneksi dari local machine ke server AWS.
SSH key yang digunakan:
`~/.ssh/terraform-reza`

Satu SSH key digunakan untuk:
```
- Gateway
- App
- DB
```
Private key tidak disimpan di repository.

### 14. Provisioning Server
Terraform digunakan untuk membuat tiga EC2 instance.
## Gateway
| Item | Value |
|---|---|
| Instance | `i-0691c2578dfda2d8e` |
| Public IP | `15.232.170.170` |
| Private IP | `10.0.1.108` |
| CPU | 1 |
| RAM | 1 GB |

## App
| Item | Value |
|---|---|
| Instance | `i-05526d0531adab746` |
| Public IP | `108.137.111.48` |
| Private IP | `10.0.1.215` |
| CPU | 2 |
| RAM | 2 GB |

## DB
| Item | Value |
|---|---|
| Instance | `i-0977cf9289ac899da` |
| Public IP | `15.232.94.115` |
| Private IP | `10.0.1.216` |
| CPU | 1 |
| RAM | 1 GB |

### 15. Terraform Init
Setelah konfigurasi Terraform selesai dibuat, dilakukan initialization.
Perintah:
``` terraform init ```
Perintah tersebut digunakan untuk:
- Menginisialisasi working directory Terraform.
- Mengunduh provider yang dibutuhkan.
- Menyiapkan Terraform untuk menjalankan konfigurasi.

![Terraform Version](./screenshots/02-terraform%20init.png.png)

### 16. Terraform Validate
Setelah initialization selesai, konfigurasi diperiksa menggunakan:
```terraform validate```

Perintah ini digunakan untuk memastikan konfigurasi Terraform valid secara syntax dan struktur.

![Terraform Version](./screenshots/03-terraform%20validate.png)

