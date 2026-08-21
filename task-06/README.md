# Task 06 — Web Server & Reverse Proxy Configuration

Repository ini berisi dokumentasi dan panduan implementasi **Web Server**, **Reverse Proxy**, dan **Load Balancing** menggunakan **NGINX** untuk aplikasi **WaysHub** pada domain `reza.xyz`.

---

## 📌 Topologi & Spesifikasi Sistem

| Perangkat | IP Address | Service / Aplikasi | Port | Fungsi |
| :--- | :--- | :--- | :--- | :--- |
| **VM 1** | `192.168.100.208` | NGINX & PM2 (WaysHub) | `80`, `3000` | Reverse Proxy & App Server 1 |
| **VM 2** | `192.168.100.11` | PM2 (WaysHub) | `3000` | App Server 2 |
| **Domain** | `reza.xyz` | HTTP | `80` | Domain Utama |

---

## 📐 1. Diagram & Cara Kerja Reverse Proxy

![Diagram Reverse Proxy](Task%2006%20\(web\%20server\%20dan\%20reverse\%20proxy\).png)

### Alur Kerja:
1. **HTTP Request:** Client mengirim permintaan akses dari browser ke domain `reza.xyz` melalui port `80`.
2. **proxy_pass:** NGINX pada VM 1 menerima *request* tersebut dan meneruskannya via `proxy_pass` ke *upstream* backend WaysHub yang berjalan di port `3000`.
3. **HTTP Response:** Backend memproses *request* dan memberikan *response* kembali ke NGINX.
4. **Response:** NGINX meneruskan *response* tersebut ke browser Client hingga halaman aplikasi dimuat.

---

## 🚀 2. Konfigurasi NGINX

Berkas konfigurasi disimpan pada `/etc/nginx/sites-available/wayshub`:

```nginx
upstream wayshub-fe {
    server 192.168.100.208:3000;
    server 192.168.100.11:3000;
}

server {
    listen 80;
    server_name reza.xyz;

    location / {
        proxy_pass http://wayshub-fe;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Pengujian Konfigurasi:
Bash
```
# Cek sintaks konfigurasi
sudo nginx -t
```
```
# Apply konfigurasi NGINX
sudo systemctl reload nginx
```

### 3. Pengujian & Pembuktian
A. Pengujian Reverse Proxy (Terminal VM 1)
Menjalankan pengujian koneksi ke NGINX melalui terminal:

```
Bash
curl -sI -H "Host: reza.xyz" 192.168.100.208
Output mengembalikan status HTTP/1.1 200 OK.
```
B. Pengujian Akses Web Browser
Buka browser dan akses domain:

Plaintext
```
[http://reza.xyz](http://reza.xyz)
```
Aplikasi WaysHub berhasil dirender dan dimuat dengan sempurna melalui port 80.

C. Pengujian Load Balancing (1 per 1)
Menjalankan perintah curl secara berulang untuk memastikan NGINX mendistribusikan request secara otomatis (Round Robin) antara VM 1 dan VM 2:

```
Bash
curl -sI -H "Host: reza.xyz" 192.168.100.208
```
