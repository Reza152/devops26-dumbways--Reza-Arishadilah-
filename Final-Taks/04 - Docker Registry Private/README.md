# 04 - Docker Registry Private

## 1. Tujuan

Tahap ini bertujuan untuk membuat Private Docker Registry yang digunakan sebagai tempat penyimpanan Docker image milik project.

Private Docker Registry digunakan agar image Frontend dan Backend dapat disimpan pada registry milik sendiri dan nantinya digunakan dalam proses deployment dan CI/CD.

Registry dijalankan pada Gateway Server.

Domain yang digunakan:

```text
registry.reza.studentdumbways.my.id
```

Server yang digunakan:
| Server | Public IP | Fungsi |
|---|---|---|
| Gateway | `15.232.170.170` | Private Docker Registry + NGINX |

2. Instalasi Docker Menggunakan Ansible
Docker pada Gateway diinstall menggunakan Ansible.
Playbook yang digunakan:
