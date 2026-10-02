# 02 - Repository

## 1. Tujuan

Tahap Repository bertujuan untuk menyiapkan repository source code yang digunakan dalam Final Project DevOps.

Repository dipisahkan menjadi dua bagian aplikasi:

- Frontend
- Backend

Repository dibuat private untuk menjaga source code project.

Pada masing-masing repository digunakan dua branch:

- `staging`
- `production`

Branch tersebut digunakan untuk memisahkan environment staging dan production.

---

## 2. Frontend Repository

Frontend menggunakan repository private:

`fe-dumbmerch`

Repository digunakan untuk menyimpan source code frontend aplikasi DumbMerch.


![Frontend Private Repository](./screenshots/task-2-01-fe-private-repository.png)



## 3. Frontend Branch

Frontend memiliki dua branch utama:

- `staging`
- `production`

| Branch | Environment |
|---|---|
| `staging` | Staging |
| `production` | Production |

### Bukti Screenshot

![Frontend Branches](./screenshots/task-2-02-fe-branches.png)

---

## 4. Frontend Node.js dan NPM

Frontend menggunakan Node.js dan NPM sebagai environment untuk menjalankan dan melakukan build aplikasi.

Versi yang digunakan:

```text
Node.js : v20.20.2
npm     : 10.8.2
