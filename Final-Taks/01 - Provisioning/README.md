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
