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


