# DevOps Infrastructure & Monitoring

Repository ini berisi implementasi Infrastructure as Code (IaC) menggunakan Terraform dan konfigurasi server menggunakan Ansible.

Project mencakup provisioning infrastructure AWS, deployment Wayshub Frontend, Docker, Nginx Reverse Proxy, SSL, Node Exporter, Prometheus, cAdvisor, Grafana, serta alert notification menggunakan Discord.

---

## Architecture

```text
                         Internet
                            │
                            ▼
                       Cloudflare DNS
                            │
                            ▼
                  Monitoring Server
                    16.79.36.15
                            │
                  ┌─────────┴─────────┐
                  │                   │
                 Nginx              Docker
                  │                   │
        ┌─────────┼─────────┐    ┌────┴────┐
        │         │         │    │         │
        ▼         ▼         ▼    ▼         ▼
     Grafana  Prometheus  Node  Grafana Prometheus
      :3000      :9090   Exporter :3000    :9090
                           :9100
                              │
                              │ Metrics
                              ▼
              ┌───────────────────────────┐
              │        AWS Servers        │
              │                           │
              │ Terraform Ubuntu          │
              │ 15.232.170.170:9100       │
              │                           │
              │ Terraform Debian          │
              │ 15.232.94.115:9100        │
              │                           │
              │ Ansible App Server        │
              │ 108.137.111.48            │
              │ ├── Wayshub :3000         │
              │ └── cAdvisor :8080        │
              │                           │
              │ Ansible Monitoring        │
              │ 16.79.36.15               │
              └───────────────────────────┘
```

## 1. Infrastructure as Code

Terraform digunakan untuk membuat dan mengelola infrastructure AWS secara deklaratif.

Infrastructure yang dibuat:

- VPC
- Public Subnet
- Internet Gateway
- Route Table
- Security Group
- SSH Key Pair
- EC2 Ubuntu 24.04
- EC2 Debian 11
- EC2 Ubuntu untuk App Server
- EC2 Ubuntu untuk Monitoring Server
- Elastic IP
- EBS Block Storage

Terraform menggunakan AWS Provider dari Terraform Registry.
Region
``` ap-southeast-3 ```

Terraform Servers
```
| Server           | OS           | Public IP      |
| ---------------- | ------------ | -------------- |
| Terraform Ubuntu | Ubuntu 24.04 | 15.232.170.170 |
| Terraform Debian | Debian 11    | 15.232.94.115  |
```
Ansible Servers
```
| Server            | OS     | Public IP      |
| ----------------- | ------ | -------------- |
| App Server        | Ubuntu | 108.137.111.48 |
| Monitoring Server | Ubuntu | 16.79.36.15    |
```

## 2. Terraform

Terraform configuration berada di:
``` Automation/Terraform/aws/ ```

File Utama:
```
main.tf
variables.tf
providers.tf
terraform.tfvars
```
Terraform digunakan untuk membuat infrastructure secara otomatis sehingga konfigurasi dapat dibuat kembali tanpa melakukan provisioning secara manual.

Command yang digunakan:
```
terraform init
terraform validate
terraform plan
terraform apply
```


## 3. Network

VPC menggunakan:

```10.0.0.0/16```

Public subnet:

```10.0.1.0/24```

Internet Gateway digunakan agar instance dapat mengakses Internet.

Route public:

```0.0.0.0/0```

Security Group menyediakan akses yang dibutuhkan untuk deployment dan monitoring.

Port yang digunakan:
```
22    SSH
80    HTTP
443   HTTPS
3000  Wayshub / Grafana
8080  cAdvisor
9090  Prometheus
9100  Node Exporter
```

## 4. Ansible

Ansible digunakan untuk melakukan konfigurasi server setelah infrastructure selesai dibuat menggunakan Terraform.

Directory:

```Automation/Ansible/```

Playbook yang digunakan:
```
users.yaml
docker.yaml
frontend.yaml
reverse-proxy.yaml
```
Monitoring:
```
monitoring/
├── node-exporter.yaml
├── prometheus.yaml
├── grafana.yaml
└── cadvisor.yaml
```

## 5. User & SSH

Ansible membuat user:

``reza``

User diberikan akses sudo dan SSH public key.

SSH key yang digunakan:

``~/.ssh/terraform-reza``

Public key:

``~/.ssh/terraform-reza.pub``

Password authentication juga dikonfigurasi untuk user reza.

## 6. Docker

Docker di-install menggunakan Ansible pada:
```
App Server
Monitoring Server
```
Docker digunakan untuk menjalankan:
```
Wayshub Frontend
cAdvisor
Prometheus
Grafana
```
Docker Compose plugin juga tersedia pada server.


## 7. Wayshub Frontend

Wayshub Frontend dideploy pada:

``108.137.111.48``

Application berjalan menggunakan Docker.

Container:

``wayshub-frontend``

Port:
``
3000:3000
``
Repository frontend:

```https://github.com/Reza152/wayshub-frontend```

Frontend menggunakan Nginx sebagai web server di dalam container.

## 8. cAdvisor

cAdvisor digunakan untuk melakukan monitoring terhadap container Docker pada App Server.

Container:

``cadvisor``

Port:

``8080``

cAdvisor menyediakan metrics container yang kemudian diambil oleh Prometheus.

Metrics digunakan untuk monitoring container Wayshub Frontend.

9. Node Exporter

Node Exporter digunakan untuk mengumpulkan system metrics dari server.

Node Exporter berjalan pada:
```
Terraform Ubuntu
Terraform Debian
App Server
Monitoring Server
```
Port:
``
9100
``
Metrics yang dikumpulkan antara lain:
```
CPU
Memory
Disk
Filesystem
Network
System information
```

## 10. Prometheus

Prometheus berjalan pada Monitoring Server:

```16.79.36.15```

Port:

`9090`

Prometheus melakukan scraping metrics dari Node Exporter dan cAdvisor.
```
Node Exporter Targets
15.232.170.170:9100
15.232.94.115:9100
108.137.111.48:9100
16.79.36.15:9100
cAdvisor Target
108.137.111.48:8080
```
Semua target diverifikasi melalui halaman Prometheus Targets.

## 11. Grafana

Grafana berjalan pada Monitoring Server:

``16.79.36.15``

Port:

``3000``

Grafana menggunakan Prometheus sebagai datasource.

Datasource:

``http://16.79.36.15:9090``

Dashboard dibuat dengan nama:

``Server Monitoring``

Dashboard menampilkan:
``
CPU Usage
RAM Usage
Disk Usage
Wayshub Frontend CPU
Wayshub Frontend RAM
``
## 12. PromQL
CPU Usage
``100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)``

Query digunakan untuk menghitung persentase CPU usage berdasarkan waktu idle CPU.
``
RAM Usage
100 * (1 - (
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
))
``
Query digunakan untuk menghitung persentase penggunaan memory.
``
Disk Usage
100 * (
  1 -
  (
    node_filesystem_avail_bytes{
      mountpoint="/",
      fstype!="rootfs"
    }
    /
    node_filesystem_size_bytes{
      mountpoint="/",
      fstype!="rootfs"
    }
  )
)
``
Query digunakan untuk menghitung penggunaan filesystem root.

Wayshub Frontend CPU
``rate(container_cpu_usage_seconds_total{name="wayshub-frontend"}[5m]) * 100``
Wayshub Frontend RAM
``container_memory_working_set_bytes{name="wayshub-frontend"} / 1024 / 1024``

## 13. Grafana Alerting

Alert dibuat menggunakan Grafana Alerting.
CPU Alert
``CPU Usage Above 20%``

Condition:

``CPU > 20%``

Evaluation:

1 minute
RAM Alert
RAM Usage Above 75%

Condition:

RAM > 75%

Evaluation:

1 minute

Alert notification menggunakan Discord sebagai contact point.

## 14. Discord Notification

Grafana dikonfigurasi untuk mengirim notification ke Discord.

Format notification:
```
🚨 ALERT: CPU Usage Above 20%

Status: firing
Server: 108.137.111.48:9100
Value: 35.40%
Threshold: > 20%
Dashboard: Server Monitoring
```
Ketika kondisi kembali normal:
``
✅ RESOLVED: CPU Usage Above 20%
```
Status: resolved
Server: 108.137.111.48:9100
Value: 3.20%
Threshold: > 20%
Dashboard: Server Monitoring
```
Notification telah diuji menggunakan Grafana Alerting.

## 15. Nginx Reverse Proxy

Nginx digunakan sebagai reverse proxy pada Monitoring Server.

Service yang diproxy:

Domain	Service
```
monitoring-rezaarishadilah.studentdumbways.my.id	Grafana :3000
prom-rezaarishadilah.studentdumbways.my.id	Prometheus :9090
exporter-rezaarishadilah.studentdumbways.my.id	Node Exporter :9100
```
## 16. SSL

SSL menggunakan Let's Encrypt dengan Certbot dan Cloudflare DNS Challenge.

Wildcard certificate:
```
*.studentdumbways.my.id
```
Certificate digunakan oleh domain monitoring.

Certificate:
``
/etc/letsencrypt/live/studentdumbways.my.id/fullchain.pem
``
Private key:
``
/etc/letsencrypt/live/studentdumbways.my.id/privkey.pem
``
HTTPS digunakan pada reverse proxy Nginx.

## 17. DNS

DNS record dibuat pada Cloudflare dan menggunakan mode:

DNS Only

Records:
```
monitoring-rezaarishadilah.studentdumbways.my.id
prom-rezaarishadilah.studentdumbways.my.id
exporter-rezaarishadilah.studentdumbways.my.id
```
Ketiganya mengarah ke:
``
16.79.36.15

## 18. Verification
Beberapa hasil yang telah diverifikasi:
```
Terraform
terraform validate
Success
```
Terraform berhasil membuat dan mengelola infrastructure AWS.

Ansible
``ansible all -m ping``

Semua server berhasil merespons.

Docker

Container berjalan pada App Server dan Monitoring Server.

Prometheus

Node Exporter dan cAdvisor berhasil masuk sebagai target dan berstatus:

UP

Grafana

Dashboard Server Monitoring berhasil menampilkan metrics.

Alerting

CPU dan RAM alert berhasil dibuat dan diuji.

Discord

Notification firing dan resolved berhasil dikirim melalui Discord.

SSL

Domain monitoring dapat diakses menggunakan HTTPS.

## [Automation Repository](https://github.com/Reza152/Automation)
