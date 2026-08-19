# Task 05 - Deployment Application

## 1. NodeJS

Saya melakukan deployment aplikasi Wayshub Frontend menggunakan NodeJS.

Versi NodeJS yang digunakan:
- NodeJS 10
- NodeJS 12

Aplikasi berjalan pada port `3000`.

Untuk mengelola aplikasi agar berjalan di background, saya menggunakan PM2.

Aplikasi dapat diakses melalui:

`http://192.168.100.208:3000`

---

## 2. Python

Saya membuat aplikasi sederhana menggunakan Python Flask yang menampilkan nama saya.

Aplikasi berjalan pada port `5000`.

Text yang ditampilkan:

`Welcome, Reza Arishadilah`

Aplikasi dijalankan menggunakan PM2.

Aplikasi dapat diakses melalui:

`http://192.168.100.208:5000`

---

## 3. Golang

Saya membuat aplikasi sederhana menggunakan Golang.

Source code:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hallo, Reza!!")
}
```

## 4. PM2

PM2 digunakan untuk menjalankan aplikasi NodeJS dan Python sebagai background process.

Pengecekan dilakukan menggunakan:
```
pm2 status
```
Hasil:
```
wayshub       online
python-app    online
```

## 5. UFW

Firewall UFW digunakan untuk mengatur akses jaringan pada server.

UFW dalam kondisi aktif dan port yang digunakan sudah diizinkan:
```
3000/tcp → NodeJS / Wayshub
5000/tcp → Python
6969/tcp → Golang
```
Pengecekan:

sudo ufw status
