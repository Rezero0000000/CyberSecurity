


## **1. Pengertian Path Traversal**

**Path Traversal** adalah teknik eksploitasi di mana penyerang memanipulasi input yang mengacu pada path file untuk mengakses file atau direktori di luar batas yang diizinkan. Teknik ini sering digunakan untuk membaca file sensitif seperti **`/etc/passwd`** atau mengakses konfigurasi server.

**Tujuan utama path traversal:**  
✅ Mengakses file sensitif (misalnya, kredensial atau konfigurasi)  
✅ Mengeksploitasi kelemahan sistem file dalam aplikasi web  
✅ Bypass mekanisme pembatasan direktori

---

## **2. Absolute Path (`/etc/passwd`)**

Absolute path adalah path yang menunjukkan lokasi file secara langsung di dalam sistem file.

**Contoh:**

```
/etc/passwd
```

- `/etc/passwd` adalah file yang berisi daftar akun pengguna di sistem UNIX/Linux.
- Jika sebuah aplikasi web memiliki kelemahan dalam validasi path file, penyerang bisa langsung mengakses file ini untuk mendapatkan informasi pengguna sistem.

**Hambatan:**  
Sebagian besar aplikasi tidak mengizinkan akses langsung ke file di luar direktori yang diizinkan (misalnya, `/var/www/`).

---

## **3. Relative Path (`../../../etc/passwd`)**

Relative path traversal menggunakan **`../`** untuk naik ke direktori atas dan mengakses file yang tidak seharusnya bisa diakses.

**Contoh eksploitasi:**

```
../../../etc/passwd
```

Jika aplikasi hanya mengizinkan akses ke `/var/www/images/`, maka:

```
/var/www/images/../../../etc/passwd
```

akan direduksi menjadi:

```
/etc/passwd
```

💡 **Trik ini memanfaatkan kurangnya validasi dalam path yang diberikan oleh pengguna.**

---

## **4. Double URL Encoding (`%252e%252e%252f`)**

Beberapa aplikasi melakukan filtering terhadap karakter **`../`**, tetapi kadang-kadang filtering tersebut bisa dilewati dengan **double URL encoding**.

**Contoh encoding standar:**

```
../ --> %2e%2e%2f
```

**Contoh double encoding:**

```
%252e%252e%252f
```

📌 **Kenapa ini bekerja?**

- Aplikasi mungkin mendekode URL hanya sekali, sehingga `"%252e"` masih terlihat sebagai string biasa.
- Tetapi jika aplikasi mendekode ulang, maka `"%252e%252e%252f"` akan berubah menjadi `"../"`, memungkinkan traversal bekerja.

---

## **5. Null Byte Injection (`%00`)**

Null Byte (`%00` atau `\0`) digunakan untuk mengecoh sistem yang menggunakan **C-based string handling**, di mana `\0` menandakan akhir string.

**Contoh eksploitasi:**

```
../../../etc/passwd%00.png
```

- Jika aplikasi memeriksa apakah input diakhiri dengan `.png`, filter bisa dibypass.
- Beberapa sistem lama akan membaca path hanya sampai **Null Byte (`\0`)**, sehingga `.png` diabaikan, dan file asli (`/etc/passwd`) tetap terbaca.

💡 **Teknik ini dulu efektif di PHP versi lama, tetapi sekarang sudah banyak dicegah.**

