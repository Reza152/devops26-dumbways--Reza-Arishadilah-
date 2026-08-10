# Task 01 - DevOps dan Instalasi Ubuntu Server

Devops adalah menggabungkan devlopment dan oprations untuk mempercepat proses delivery software
tujuan devops ini untuk membuat proses pengembangan, testing, deployment, dan oprasional software agar menjadi lebih cepat dan konsisten.

## 2. Instalasi Ubuntu Server di dalam virtualBox

pada task ini saya akan melakukan instalasi ubuntu server menggunakan Oracle VM VirtualBox.

### step 1 
saya mengisi VM nama dan opration system dengan nama Dumbways dan memasukan ISO ubuntu server

### step 2
tampilan akan memunculkan untuk membuat virtual hardware seperti memory, cpu, dan disk

### step 3
muncul bagian summary artinya berhasil dibuat

### step 4
setalah itu klik icon start, akan masuk kedalam virtualbox ubuntu server

### step 5
lalu ada 3 pilihan saya klik (try or install Ubuntu Server)

### step 6
lalu diarahkan untuk memilih bahasa disini saya memilih bahasa english

### step 7
setelah dari bahasa akan muncul instailer update, saya langsung klik continue

### step 8 
lalu ada tampilan untuk keyboard saya meilih done

### step 9
akan mesuk ke tampilan network config, disini saya memilih untuk ip manual 
subnet: 192.168.100.0/24
Address: 192.168.100.208
getway: 192.168.100.1
dns: 8.8.8.8, 8.8.4.4

### step 10
selanjutnya saya mengcustome storage layout, dikarenakan masih ada free space yg banyak saya 
membuat partisi dan ternyata setelah saya melakukan partisi masih ada sisa sedikit dan saya membuat untuk swap 

### step 11 
lalu akan menampilkan profil configuration 

### step 12
di selanjutnya itu akan diarath kan untuk mengistar ssh dan featured, saya memilih tidak mengistal 

### step 13
setelah itu akan melakukan instaliation, ditunggu sampai muncul pilihan (reboot now)

# step 14 
menunggu sampai muncul masukan username dan password 

# step 15
setelah memasukan ping 8.8.8.8 namun ternyata itu gagal 

# step 16 
saya mencoba malakukan mengganti dibagian setting kolom network yg tadinya NAT saya ubah menjadi Bridge Adapter. setelah saya coba melakukan ping itu berhasil ke 8.8.8.8






