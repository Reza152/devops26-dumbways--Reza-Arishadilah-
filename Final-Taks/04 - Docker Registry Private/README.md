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

## 2. Instalasi Docker Menggunakan Ansible
Docker pada Gateway diinstall menggunakan Ansible.
Playbook yang digunakan:
`ansible/playbooks/docker.yaml`

Playbook dijalankan pada host gateway:
```
---
- name: Install and Configure Private Docker Registry on Gateway
  hosts: gateway
  become: true

Install Docker dan Docker Compose:
- name: Install Docker and Docker Compose
  ansible.builtin.apt:
    name:
      - docker.io
      - docker-compose-v2
    state: present
    update_cache: true

Konfigurasi tersebut menginstall:
- docker.io
- docker-compose-v2
Kemudian Docker diaktifkan dan dijalankan:
- name: Enable and start Docker
  ansible.builtin.systemd:
    name: docker
    enabled: true
    state: started

User reza juga ditambahkan ke group Docker:
- name: Add reza to docker group
  ansible.builtin.user:
    name: reza
    groups: docker
    append: true
```
Menjalankan Ansible
Playbook dijalankan menggunakan:
`ansible-playbook -i inventory/hosts.ini playbooks/docker.yaml -K`

Hasil instalasi Docker berhasil:
`
PLAY RECAP
15.232.170.170 : ok=4 changed=2 unreachable=0 failed=0 skipped=0
`
(./screenshots/01-install-docker.png)
