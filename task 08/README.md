# WaysHub Infrastructure, Docker, SSL, & CI/CD Documentation — Reza Arishadilah

Dokumentasi ini merinci arsitektur infrastruktur multi-server (3 VM), konfigurasi kontainerisasi Docker, implementasi SSL Wildcard, serta pembagian otomatisasi Continuous Integration & Continuous Deployment (CI/CD) menggunakan Jenkins (untuk Frontend) dan GitHub Actions (untuk Backend) pada aplikasi WaysHub.

---

## 🌐 1. Arsitektur Infrastruktur (Multi-VM)

Infrastruktur ini dibagi secara terisolasi ke dalam 3 Virtual Machine (VM) untuk memastikan pemisahan layer layanan (Gateway, App/Staging, dan Database):

### VM 1 – Gateway Server
* **Fungsi Utama:** Berperan sebagai pintu gerbang trafik masuk (*reverse proxy*) dan pusat kontrol otomatisasi CI/CD Jenkins.
* **Komponen:**
  * **Nginx Reverse Proxy:** Mengarahkan akses domain publik atas nama pribadi ke layanan internal.
  * **Jenkins (Docker Container):** Menjalankan automation server yang memantau perubahan repositori frontend secara berkala dan mengeksekusi `Jenkinsfile`.

### VM 2 – Backend & Staging Server (IP: `172.31.15.141`)
* **Fungsi Utama:** Menjadi tempat *runtime* utama untuk menjalankan aplikasi (Frontend & Backend) di atas Docker Engine.
* **Komponen:**
  * **Frontend Container (`production_frontend`):** Melayani file statis aplikasi React yang dibungkus Nginx (port `3000` ke `80`).
  * **Backend Container (`staging_backend`):** Menjalankan layanan API aplikasi (`172.31.15.141:5000`) yang terhubung langsung ke database eksternal.
  * **Custom Docker Network:** Seluruh kontainer di VM ini terhubung dalam satu jaringan virtual khusus (`staging-wayshub_reza_network`) yang terisolasi.

### VM 3 – Database Server (IP: `172.31.19.81`)
* **Fungsi Utama:** Menyimpan seluruh data aplikasi secara aman dan terpusat.
* **Komponen:**
  * **MySQL Database (`staging_db`):** Dijalankan pada IP `172.31.19.81` dengan Docker Volume persisten (`mysql_data:/var/lib/mysql`) agar data tetap aman dan tidak hilang saat kontainer direstart/di-update.

---

## 🔒 2. Konfigurasi SSL Wildcard & Nginx Proxy

* **SSL Cloudflare Status:** Dimatikan (OFF).
* **SSL Wildcard:** Menggunakan sertifikat wildcard lokal (Let's Encrypt) yang terpasang pada Nginx Reverse Proxy di VM 1 untuk mengamankan seluruh subdomain secara terpusat.
* **Mapping Domain (Atas Nama Reza Arishadilah):**
  * **Frontend:** `rezaarishadilah.studentdumbways.my.id`
  * **Backend API:** `api.rezaarishadilah.studentdumbways.my.id`
  * **Jenkins Dashboard:** `jenkins.rezaarishadilah.studentdumbways.my.id`

---

## 🐳 3. Konfigurasi Docker & Multi-Stage Build

* **Custom Network:** Menggunakan jaringan virtual Docker khusus (`staging-wayshub_reza_network`) yang dipasang ke setiap service untuk mengamankan komunikasi antar-kontainer.
* **Multi-Stage Build (Frontend & Backend):**
  * `Dockerfile` menggunakan pendekatan *multi-stage* (tahap *builder* menggunakan Node.js untuk proses *compile dependencies*, lalu hasilnya disalin ke *image* ringkas berbasis Nginx Alpine). Pendekatan ini menekan ukuran *image* akhir sekecil mungkin.
* **Environment Configuration:**
  * Konfigurasi koneksi dari Backend ke Database diset menggunakan IP Address eksternal VM 3 (`DB_HOST=172.31.19.81`) untuk memastikan komunikasi lintas-VM berjalan stabil.
* **Docker Registry:** 
  * *Image* hasil *build* diberi penamaan sesuai environment (contoh: `reza1019/wayshub-frontend:latest` dan `reza1019/wayshub-backend:staging`) dan di-*push* ke Docker Hub (`reza1019`).

---

## 🔄 4. Alur CI/CD Pipeline (Jenkins & GitHub Actions)

### Frontend Pipeline (Jenkins di VM 1):
1. **Pull dari SCM:** Otomatis menarik kode terbaru dari repository frontend menggunakan SCM Polling.
2. **Dockerize & Build:** Menjalankan `docker build` dengan teknik *multi-stage*.
3. **Test Application:** Melakukan *smoke test* via kontainer uji sementara.
4. **Push ke Docker Hub:** Mengunggah *image* frontend terbaru ke *registry*.
5. **Deploy on top Docker (VM 2 via SSH):** Jenkins melakukan koneksi aman via SSH-KEY ke VM 2 untuk memperbarui kontainer aplikasi.
6. **Discord Notification:** Mengirimkan pesan status sukses (`✅`) atau gagal (`❌`) secara *real-time* ke Discord.

### Backend Pipeline (GitHub Actions):
* Otomatis melakukan *build*, *test*, *push* *image* backend ke Docker Hub, dan memperbarui layanan backend di VM 2 setiap kali ada pembaruan kode pada *branch* utama backend.
