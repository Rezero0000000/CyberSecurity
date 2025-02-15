## Basic Linux Commands

### Perintah Dasar

| Perintah                    | Deskripsi                                      |
| --------------------------- | ---------------------------------------------- |
| `echo`                      | Menampilkan teks ke output                     |
| `whoami`                    | Menampilkan user yang sedang login             |
| `cd`                        | Berpindah direktori                            |
| `ls`                        | Menampilkan isi direktori                      |
| `pwd`                       | Menampilkan direktori saat ini                 |
| `man <tool>`                | Melihat manual suatu perintah                  |
| `less file.txt`             | Membaca file dengan navigasi                   |
| `more file.txt`             | Mirip `less`, tetapi hanya maju                |
| `cat file.txt`              | Menampilkan isi file                           |
| `grep "find this" file.txt` | 625Mencari teks dalam file                     |
| `dpkg`                      | Manajemen paket di Debian-based Linux          |
| `passwd`                    | Mengubah password                              |
| `apt`                       | Mengelola paket di Debian-based Linux          |
| `useradd`                   | Menambahkan user                               |
| `usermod`                   | Memodifikasi user                              |
| `userdel`                   | Menghapus user                                 |
| `addgroup`                  | Menambahkan group                              |
| `ps`                        | Menampilkan proses yang berjalan               |
| `journalctl`                | Melihat log systemd                            |
| `tree .`                    | Menampilkan struktur direktori                 |
| `touch h.txt`               | Membuat file kosong                            |
| `rm file.txt`               | Menghapus file                                 |
| `mv <file/dir> <tujuan>`    | Memindahkan atau mengganti nama file/direktori |
| `cp <file> <tujuan>`        | Menyalin file                                  |
| `                           | (pipe)`                                        |
| `&`                         | Menjalankan perintah di background             |
| `>`                         | Mengganti isi file                             |
| `>>`                        | Menambahkan isi file                           |

---

## Permission di Linux

### Struktur

1. Setiap file/direktori memiliki **owner, group, dan others**.
2. Terdapat **3 jenis permission**:
    - **Read (r)**: Membaca file/direktori
    - **Write (w)**: Mengubah isi file/direktori
    - **Execute (x)**: Menjalankan file (jika executable)
3. Format tampilan permission:
    
    ```bash
    -rwx r-x-wx
    ```
    
    - 3 pertama untuk **owner**
    - 3 kedua untuk **group**
    - 3 ketiga untuk **others**



### Mengubah Permission

#### A. Format Simbolik

```bash
chmod [who][operator][permission] file.txt
```

**Who:**

- `u`: Owner
- `g`: Group
- `o`: Others
- `a`: Semua

**Operator:**

- `+`: Menambah permission
- `-`: Menghapus permission
- `=`: Mengatur permission

**Permission:**

- `r`: Read
- `w`: Write
- `x`: Execute



#### B. Format Numerik

|Simbol|Numerik|
|---|---|
|`---`|0|
|`--x`|1|
|`-w-`|2|
|`-wx`|3|
|`r--`|4|
|`r-x`|5|
|`rw-`|6|
|`rwx`|7|

**Contoh:**

```bash
chmod 754 file.txt
```

---

## Systemctl (Mengontrol systemd)

|Perintah|Deskripsi|
|---|---|
|`sudo systemctl start <service>`|Menjalankan service|
|`sudo systemctl stop <service>`|Menghentikan service|
|`sudo systemctl restart <service>`|Merestart service|
|`sudo systemctl reload <service>`|Reload konfigurasi service tanpa menghentikannya|
|`sudo systemctl status <service>`|Menampilkan status service|
|`systemctl list-units --type=service`|Menampilkan daftar service|

---

## System Log

|Log|Lokasi|
|---|---|
|Kernel log|`/var/log/kern.log`|
|System log|`/var/log/sys.log`|
|Authentication log|`/var/log/auth.log`|
|Application log|Tergantung aplikasi, contoh: `/var/log/apache2/error.log`|

---

## Perintah Terkait Storage

| Perintah                              | Deskripsi                       |
| ------------------------------------- | ------------------------------- |
| `sudo fdisk -l`                       | Melihat daftar partisi          |
| `sudo mount`                          | Melihat drive yang ter-mount    |
| `lsblk`                               | Menampilkan informasi USB drive |
| `sudo mount <usb-name> <mount-point>` | Mount USB                       |
| `sudo blkid`                          | Melihat format/type USB         |

---

## DNS (Domain Name Server)

DNS berfungsi untuk menerjemahkan nama domain menjadi alamat IP. **Contoh:**

```bash
www.google.com -> 192.0.2.1
```

---

## TCP/IP (Transmission Control Protocol/Internet Protocol)

TCP/IP adalah kumpulan protokol yang digunakan dalam komunikasi jaringan.

### Cara Kerja:

4. Data dipecah menjadi paket kecil.
5. **IP** menentukan rute paket di jaringan.
6. **TCP** memastikan semua paket sampai di tujuan dan menyusunnya kembali.

### Layer TCP/IP:

7. **Application**: HTTP, FTP, SMTP
8. **Transport**: TCP, UDP
9. **Internet**: IPv4, IPv6, ICMP
10. **Network Interface**: Komunikasi fisik (WiFi/Ethernet)

---

## OSI Layer (Open System Interconnection)

Model ini memiliki 7 layer:

11. **Application**: HTTP, FTP, SMTP
12. **Presentation**: Format data, enkripsi, kompresi
13. **Session**: Menjaga sesi komunikasi tetap aktif
14. **Transport**: Transfer data (TCP/UDP)
15. **Network**: Routing data (IP)
16. **Data Link**: Transfer data antar perangkat (Ethernet)
17. **Physical**: Media fisik (kabel, gelombang radio)

**Catatan:**

- **TCP** lebih lambat tetapi data lebih aman.
- **UDP** lebih cepat tetapi ada potensi corrupt data.

---