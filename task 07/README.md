# 🚀 Final Task - Fullstack Application Deployment (WaysHub)

Dokumentasi ini berisi panduan dan spesifikasi teknis deployment aplikasi **WaysHub** (React Frontend & Express Backend) pada infrastruktur cloud **AWS EC2** menggunakan **Nginx**, **PM2**, dan **SSL Certbot**.

---

## 🏗️ Arsitektur Server & Spesifikasi

Infrastruktur terbagi menjadi dua Virtual Machine (VM) pada VPC AWS:

### 1. VM Gateway / Frontend Server
* **OS:** Ubuntu 26.04 LTS (AWS EC2)
* **IP Public:** `15.232.18.168`
* **IP Private:** `172.31.1.31`
* **System User:** `reza`
* **Domain:** `https://rezaarishadilah.studentdumbways.my.id`
* **Teknologi:**
  * Node.js (v14.21.3) & NPM
  * PM2 Process Manager (Port `3000`)
  * Nginx Web Server (Reverse Proxy)
  * Let's Encrypt / Certbot (SSL Certificate)

### 2. VM Backend Server
* **OS:** Ubuntu (AWS EC2)
* **IP Public:** `15.232.164.78`
* **IP Private:** `172.31.15.141`
* **System User:** `reza`
* **Subdomain API:** `https://api.rezaarishadilah.studentdumbways.my.id`
* **Teknologi:**
  * Node.js (v14.21.3) & Express.js
  * PM2 Process Manager (Port `5000`)
  * Database MySQL (User: `reza`, Database: `demo`)
  * Sequelize ORM (Migration & Seeder)

---

## ⚙️ Langkah Deployment

### A. Setup Backend Server (`172.31.15.141`)
1. **Konfigurasi Database & Sequelize:**
   Menyesuaikan file `config/config.json` menggunakan kredensial MySQL lokal (`user: reza`, `database: demo`).
2. **Migrasi Database:**
   ```bash
   npx sequelize-cli db:migrate
   ```
   
