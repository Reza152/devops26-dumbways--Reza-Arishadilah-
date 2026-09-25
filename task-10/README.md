# Kubernetes WaysHub

## Task

Deploy aplikasi WaysHub menggunakan Kubernetes K3s.

## 1. Kubernetes Cluster

Membuat Kubernetes cluster menggunakan 3 node:

- Node 1: Control Plane + Worker
- Node 2: Worker
- Node 3: Worker

Check node:

```bash
kubectl get nodes
```

## 2. NGINX Ingress

Install NGINX Ingress Controller menggunakan Helm.

Check:
```
kubectl get pods -n nginx-ingress
```
3. Deploy WaysHub

Deploy frontend dan backend menggunakan Docker image dari Docker Hub.

Frontend
```reza1019/wayshub-frontend:k3s````

Backend
```reza1019/wayshub-backend:k3s```

Check:
```
kubectl get deployment
kubectl get pods
kubectl get svc
```
4. Persistent Volume

Membuat PersistentVolumeClaim untuk storage MySQL sebesar 5Gi.

Check:
```
kubectl get pvc
kubectl get pv
```
5. MySQL StatefulSet & Secret

MySQL dijalankan menggunakan StatefulSet.

Credential MySQL disimpan menggunakan Kubernetes Secret.

File:
```
mysql-statefulset.yaml
mysql-secret.yaml
mysql-service.yaml
```
Check:
```
kubectl get statefulset
kubectl get secret
```
6. cert-manager & SSL

Install cert-manager dan membuat ClusterIssuer menggunakan Let's Encrypt dan Cloudflare DNS-01.

Certificate digunakan untuk:
```
*.reza.kubernetes.studentdumbways.my.id
reza.kubernetes.studentdumbways.my.id
```
Check:
```
kubectl get clusterissuer
kubectl get certificate
```
7. Ingress

Membuat Ingress untuk menghubungkan domain ke frontend dan backend.
```
Frontend
https://reza.kubernetes.studentdumbways.my.id
```
```
Backend
https://api.reza.kubernetes.studentdumbways.my.id
```
Check:
```
kubectl get ingress
```
8. Database Migration

Setelah backend dan MySQL berjalan, dilakukan Sequelize migration untuk membuat tabel database.

```
kubectl exec -it deploy/wayshub-backend -- sh
npx sequelize-cli db:migrate
```
Setelah migration selesai, aplikasi dapat digunakan dan login berhasil.

9. Testing

Frontend:
```
curl -I https://reza.kubernetes.studentdumbways.my.id
```
Backend:
```
curl -I https://api.reza.kubernetes.studentdumbways.my.id
```
Frontend berhasil memberikan response 200 OK dan backend berhasil menerima request melalui Express.
