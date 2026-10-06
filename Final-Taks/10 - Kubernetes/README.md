## Overview
Aplikasi production DumbMerch dijalankan menggunakan K3s Single Node.
K3s digunakan sebagai Kubernetes distribution pada satu server. Node tersebut berfungsi sebagai control-plane sekaligus tempat menjalankan workload production.
Application production dijalankan pada namespace:
production

Komponen production yang dijalankan di dalam Kubernetes terdiri dari:
```
Frontend
Backend
PostgreSQL
```
Frontend dan backend masing-masing memiliki dua replica, sedangkan PostgreSQL menggunakan satu replica.
Kubernetes dijalankan menggunakan single node, sehingga seluruh workload production berada pada node yang sama.

## Kubernetes / K3s Single Node
Kubernetes menggunakan K3s.
Node yang digunakan:
```
Node Name  : ip-10-0-1-59
Private IP : 10.0.1.59
Role       : control-plane
Status     : Ready
OS         : Ubuntu 24.04.5 LTS
K3s Version: v1.36.5+k3s1
```
K3s berjalan menggunakan container runtime:
```containerd```

Pengecekan node dilakukan menggunakan:
```kubectl get nodes -o wide```

Hasil pengujian menunjukkan:
```
NAME           STATUS   ROLES          VERSION
ip-10-0-1-59   Ready    control-plane  v1.36.5+k3s1
```
Status Ready menunjukkan bahwa node dapat digunakan untuk menjalankan workload Kubernetes.
Node tersebut memiliki private IP:
```10.0.1.59```

