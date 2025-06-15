# 🔧 Otomasi Upgrade MikroTik dengan Ansible

Playbook ini digunakan untuk melakukan otomatisasi skala besar untuk upgrade RouterOS dan firmware RouterBOARD pada perangkat MikroTik menggunakan SSH melalui Ansible.

---

## 📁 Struktur Folder

```
mikrotik-automation/
├── inventory                # Daftar IP MikroTik
├── mikrotik_upgrade.yml    # Playbook utama
├── mikrotik_backup.yml    # playbook backup
├── group_vars/
│   └── mikrotik.yml        # Variabel koneksi (user/password)
└── vault/
    └── secrets.yml         # File terenkripsi dengan ansible-vault
```

---

## 📦 Kebutuhan

- Ansible v2.10 atau lebih baru
- Python module `paramiko` atau `ssh-python`
- SSH aktif pada perangkat MikroTik
- MikroTik RouterOS versi 6.x atau 7.x
- Akses ke perangkat dari host Ansible

---

## 🔐 Menyimpan Kredensial dengan Vault

Untuk menjaga keamanan kredensial, gunakan `ansible-vault`.

### 🛠️ Membuat file terenkripsi:
```bash
ansible-vault create vault/secrets.yml
```

### Contoh isi:
```yaml
vault_ansible_user: admin
vault_ansible_password: puyeng*
```

### Hubungkan dengan variabel di `group_vars/mikrotik.yml`:
```yaml
ansible_user: "{{ vault_ansible_user }}"
ansible_password: "{{ vault_ansible_password }}"
ansible_connection: network_cli
ansible_network_os: routeros
ansible_port: 22
```

---

## 🚀 Menjalankan Playbook

### 📌 Tanpa reboot:
```bash
ansible-playbook -i inventory mikrotik_upgrade.yml --ask-vault-pass
ansible-playbook -i inventory mikrotik_upgrade.yml --vault-password-file ~/.vault_pass.txt
```

### 📌 Dengan reboot otomatis:
```bash
ansible-playbook -i inventory mikrotik_upgrade.yml --vault-password-file ~/.vault_pass.txt -e "confirm_reboot=true"
```

> Reboot hanya dilakukan jika `confirm_reboot=true`.

---

## ✅ Apa yang Dilakukan Playbook Ini

- Mengecek apakah ada update RouterOS
- Mengunduh update jika tersedia (tanpa langsung restart)
- Meng-upgrade firmware RouterBOARD
- (Opsional) Melakukan reboot otomatis

---

## 🔐 Keamanan

- File `vault/secrets.yml` aman untuk di-commit karena telah dienkripsi.
- Jangan commit file `.vault_pass.txt`. Tambahkan ke `.gitignore`:

```
.vault_pass.txt
```

---



## 💾 Backup Konfigurasi MikroTik

Selain upgrade, proyek ini juga menyediakan playbook `mikrotik_backup.yml` untuk melakukan backup konfigurasi MikroTik dalam dua format:

- `backup.rsc` → format teks (hasil dari `/export`)
- `backup.backup` → format binary (hasil dari `/system backup save`)

### 📂 Struktur Folder Tambahan:
```
mikrotik-automation/
└── backups/
    ├── 10.0.0.1/
    │   ├── backup.rsc
    │   └── backup.backup
    └── 10.0.0.2/
        └── ...
```

### 🚀 Menjalankan Playbook Backup:
```bash
ansible-playbook -i inventory mikrotik_backup.yml --vault-password-file ~/.vault_pass.txt
```

### 🔄 Setelah Backup:
Secara default, file backup akan dihapus dari perangkat MikroTik setelah berhasil diambil.

## 👨‍💻 Pembuat

Dibuat oleh [@rskabc](https://github.com/rskabc)  
Digunakan untuk otomasi internal perangkat jaringan MikroTik.
---
---