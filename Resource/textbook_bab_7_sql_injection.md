<div align="center">

# Institut Teknologi Sepuluh Nopember (ITS)
### Program Sarjana • Textbook Cybersecurity

## BAB 7
# SQL Injection: Dari Struktur Query menuju Eksploitasi Terkendali
*Dari pemisahan data dan perintah menuju evidence, mitigasi, dan regression test*

**Hatma Suryotrisongko**

> *Materi latihan hanya untuk lingkungan milik sendiri atau yang memiliki izin tertulis.*

</div>

---

## Isi Bab

Bab ini bergerak dari model relasional dan *source-to-sink* menuju teknik manual, inferensi *blind*, automasi terkendali, pencegahan struktural, kerangka **BINDING**, dan praktikum berbasis *evidence*.

- [7.1 Kedudukan SQL Injection dalam Keamanan Aplikasi](#71-kedudukan-sql-injection-dalam-keamanan-aplikasi)
- [7.2 Kewenangan, Scope, dan Minimisasi Data](#72-kewenangan-scope-dan-minimisasi-data)
- [7.3 Model Relasional dan Anatomi Pernyataan SQL](#73-model-relasional-dan-anatomi-pernyataan-sql)
- [7.4 Dialek DBMS dan Konsekuensi Pengujian](#74-dialek-dbms-dan-konsekuensi-pengujian)
- [7.5 Akar Masalah: Data Menjadi Struktur Query](#75-akar-masalah-data-menjadi-struktur-query)
- [7.6 Pemetaan Source-Transform-Query-Sink](#76-pemetaan-source-transform-query-sink)
- [7.7 Baseline dan Penemuan Kandidat Injeksi](#77-baseline-dan-penemuan-kandidat-injeksi)
- [7.8 Quote, Komentar, Tipe Data, dan Presedensi](#78-quote-komentar-tipe-data-dan-presedensi)
- [7.9 Pengujian Boolean-Based](#79-pengujian-boolean-based)
- [7.10 Pengujian Error-Based](#710-pengujian-error-based)
- [7.11 UNION-Based: Prasyarat dan Workflow](#711-union-based-prasyarat-dan-workflow)
- [7.12 Metadata dan System Catalog](#712-metadata-dan-system-catalog)
- [7.13 Blind SQL Injection sebagai Masalah Inferensi](#713-blind-sql-injection-sebagai-masalah-inferensi)
- [7.14 Time-Based Testing yang Terukur](#714-time-based-testing-yang-terukur)
- [7.15 Second-Order SQL Injection](#715-second-order-sql-injection)
- [7.16 Autentikasi, Otorisasi, dan Mitos Bypass Login](#716-autentikasi-otorisasi-dan-mitos-bypass-login)
- [7.17 Dampak Write, Transaksi, dan Integritas Data](#717-dampak-write-transaksi-dan-integritas-data)
- [7.18 Dari DBMS ke Sistem Operasi](#718-dari-dbms-ke-sistem-operasi)
- [7.19 Automasi Terkendali dengan sqlmap](#719-automasi-terkendali-dengan-sqlmap)
- [7.20 Evidence, Confidence, dan Batas Klaim](#720-evidence-confidence-dan-batas-klaim)
- [7.21 Pencegahan Primer dengan Parameterized Query](#721-pencegahan-primer-dengan-parameterized-query)
- [7.22 Identifier Dinamis, ORM, dan Stored Procedure](#722-identifier-dinamis-orm-dan-stored-procedure)
- [7.23 Defense in Depth dan Secure SDLC](#723-defense-in-depth-dan-secure-sdlc)
- [7.24 Kerangka BINDING](#724-kerangka-binding)
- [7.25 Praktikum Terpandu dan Penilaian](#725-praktikum-terpandu-dan-penilaian)

---

> **Catatan Akademik:** Contoh pada bab ini menggunakan aplikasi fiktif atau laboratorium *vulnerable-by-design*. Pembaca dilarang mempraktikkan teknik pada sistem publik tanpa otorisasi eksplisit.

---

## Pendahuluan

SQL injection (SQLi) terjadi ketika data yang tidak dipercaya ikut menentukan struktur perintah SQL. Masalahnya bukan semata-mata keberadaan tanda petik, kata `UNION`, atau fungsi penunda waktu. Akar persoalannya ialah hilangnya pemisahan antara data yang seharusnya diproses dan perintah yang akan ditafsirkan oleh *database management system* (DBMS). Ketika batas tersebut runtuh, penyerang dapat mengubah logika kueri, memperoleh data di luar kewenangannya, memodifikasi rekaman, atau—pada kondisi yang jauh lebih terbatas—menjangkau kemampuan sistem operasi.

Kerentanan ini telah lama dikenal, tetapi tetap relevan. OWASP Top 10:2025 menempatkan *injection* pada kategori A05 dan menegaskan bahwa *query* dinamis, pemanggilan tanpa parameterisasi, serta input berbahaya pada ORM maupun *stored procedure* tetap menjadi sumber risiko. CWE-89 mendefinisikan kelemahan ini sebagai kegagalan menetralkan elemen khusus yang digunakan dalam perintah SQL. Dalam PEN-200, cakupan pembelajaran bergerak dari teori SQL dan karakteristik DBMS menuju eksploitasi manual, teknik `UNION`, *error-based*, *blind SQLi*, pemanfaatan `xp_cmdshell`, serta automasi dengan `sqlmap`.

Bab ini tidak menyajikan *payload* sebagai daftar hafalan. Mahasiswa diajak membangun model mental *query*, merancang eksperimen terkontrol, membedakan indikasi dari bukti, dan menulis rekomendasi yang menghilangkan akar masalah. Seluruh contoh menggunakan aplikasi fiktif **Portal Akademik Arunika** pada host `<LAB-HOST>` dan akun laboratorium. Pengujian terhadap sistem lain memerlukan izin tertulis, ruang lingkup eksplisit, serta prosedur penanganan data yang disepakati.

### Capaian Pembelajaran Bab

Setelah mempelajari bab ini, mahasiswa mampu:
1. menjelaskan SQLi melalui pemisahan data dan struktur perintah;
2. membaca konteks dasar `SELECT`, `WHERE`, `ORDER BY`, `UNION`, fungsi, dan *subquery*;
3. membedakan teknik *in-band*, *error-based*, *boolean-based*, *time-based*, dan *second-order*;
4. menyusun pengujian manual yang *reproducible*, rendah dampak, dan sadar dialek DBMS;
5. menjelaskan prasyarat sebelum SQLi dapat berkembang menjadi perubahan data atau eksekusi perintah sistem operasi;
6. menggunakan `sqlmap` secara terkendali setelah hipotesis manual terbentuk;
7. menilai kekuatan *evidence* dan membatasi klaim sesuai hasil observasi;
8. merancang perbaikan menggunakan *parameterized query*, pemetaan *identifier*, *least privilege*, *logging*, dan *regression test*.

---

## Peta Konsep

> **Gagasan Pengikat:** SQLi adalah kegagalan pemisahan data dan bahasa. Penguji menelusuri aliran input menuju *query*, mengubah satu variabel, membandingkan respons, lalu berhenti ketika bukti minimum yang disepakati telah tercapai. Pengembang memperbaikinya dengan membuat struktur *query* tetap dan mengirim nilai melalui parameter terikat.

---

## 7.1 Kedudukan SQL Injection dalam Keamanan Aplikasi

SQLi merupakan salah satu bentuk *injection* pada *boundary* menuju *interpreter*. Browser atau klien mengirim data; aplikasi mentransformasi data tersebut; komponen akses data membentuk perintah; DBMS kemudian menafsirkan perintah itu. Jika data dapat menjadi token sintaksis, penyerang memperoleh pengaruh terhadap keputusan *database*. Karena itu, validasi input dan proteksi keluaran tidak boleh dipertukarkan dengan parameterisasi: ketiganya mempunyai tujuan berbeda.

OWASP WSTG membagi teknik SQLi menjadi *in-band*, *inferential* atau *blind*, dan *out-of-band*. *In-band* memanfaatkan kanal respons yang sama untuk memperoleh hasil. *Blind* tidak menampilkan data secara langsung; informasi disimpulkan melalui perubahan kondisi atau waktu. *Out-of-band* menggunakan kanal lain dan hanya layak diuji jika *egress* serta infrastruktur *callback* secara eksplisit masuk ruang lingkup. Kategori teknik tidak identik dengan dampak. *Boolean-based* dapat membuktikan akses baca, sementara sebuah *error-based test* dapat berhenti pada bukti bahwa input memengaruhi *parser*.

Dalam konteks S1 Cybersecurity, tujuan pembelajaran bukan sekadar mendapatkan keluaran. Mahasiswa harus mampu menjawab empat pertanyaan: input mana yang dikendalikan; pada konteks SQL apa input ditempatkan; *evidence* apa yang membedakan hipotesis; dan kontrol desain apa yang memutus jalur tersebut. Cara berpikir ini dapat digunakan saat pengujian *black-box*, *code review*, maupun verifikasi perbaikan.

---

## 7.2 Kewenangan, Scope, dan Minimisasi Data

SQLi bersentuhan langsung dengan data organisasi. Sebelum menguji, dokumen *rules of engagement* perlu menetapkan host, endpoint, akun, jenis operasi, periode uji, batas laju, *stop condition*, serta prosedur retensi bukti. Izin untuk menguji satu parameter baca tidak otomatis mencakup `INSERT`, `UPDATE`, `DELETE`, enumerasi massal, akses file, *callback* keluar, atau eksekusi perintah sistem.

Prinsip *proof-before-impact* menyatakan bahwa penguji memilih bukti dengan risiko terendah yang masih cukup untuk mendukung temuan. Untuk membuktikan pembacaan lintas objek, satu marker fiktif atau hitungan baris sering kali memadai; tidak perlu mengekstrak seluruh tabel pengguna. Bukti harus disamarkan sebelum masuk laporan. Nilai cookie, token, alamat pribadi, hash kata sandi, atau data akademik tidak boleh dipertahankan jika tidak diperlukan.

Kondisi penghentian juga harus konkret: respons tak terduga, kenaikan error, *latency* yang mengganggu, perubahan *state*, atau indikasi data produksi merupakan alasan untuk berhenti dan melakukan eskalasi. Bahaya uji OR perlu dipahami secara khusus. Kondisi yang tampak tidak berbahaya pada `SELECT` dapat mengenai `UPDATE` atau `DELETE` lain yang menggunakan input sama dan memengaruhi banyak baris. OWASP WSTG dan dokumentasi `sqlmap` sama-sama mengingatkan konsekuensi ini.

---

## 7.3 Model Relasional dan Anatomi Pernyataan SQL

Database relasional menyimpan data dalam tabel yang terdiri atas baris dan kolom. Kunci primer membedakan baris, sedangkan kunci asing menghubungkan tabel. Hak akses diberikan kepada *principal database* untuk operasi tertentu pada objek tertentu. SQLi mengeksploitasi cara aplikasi membangun pernyataan, bukan model relasional itu sendiri.

Kueri baca sederhana terdiri atas proyeksi, sumber, dan predikat:

```sql
SELECT nama_mata_kuliah, sks FROM mata_kuliah WHERE kode = ?
```

`SELECT` menentukan kolom keluaran, `FROM` menentukan sumber, dan `WHERE` membatasi baris. `ORDER BY` mengurutkan hasil. `JOIN` menggabungkan relasi. `GROUP BY` dan fungsi agregat merangkum data. `UNION` menggabungkan *result set* dengan jumlah kolom yang sama dan tipe yang kompatibel. Pernyataan *write* seperti `INSERT`, `UPDATE`, dan `DELETE` mengubah *state* dan karena itu mempunyai profil risiko pengujian berbeda.

Presedensi operator juga penting. Banyak DBMS mengevaluasi `AND` sebelum `OR`, tetapi penguji tidak boleh menyimpulkan perilaku hanya dari potongan *payload*. Tanda kurung, tipe kolom, *casting*, nilai `NULL`, *collation*, dan *query* tambahan dapat mengubah hasil. Sebuah respons login yang berbeda tidak membuktikan bahwa autentikasi berhasil; mungkin aplikasi hanya memilih baris berbeda, menampilkan error generik, atau mengaktifkan cabang kode lain.

---

## 7.4 Dialek DBMS dan Konsekuensi Pengujian

SQL memiliki standar, tetapi DBMS menerapkan dialek, fungsi, katalog, mekanisme komentar, dan fitur administratif yang tidak seragam. *Fingerprinting* harus diperlakukan sebagai hipotesis yang dikonfirmasi dengan beberapa sinyal, bukan tebakan dari satu error.

| Aspek | PostgreSQL | MySQL/MariaDB | Microsoft SQL Server | SQLite |
| :--- | :--- | :--- | :--- | :--- |
| **Penanda parameter umum** | `$1, $2` pada protokol/SQL tertentu | `?` pada *prepared statement* | `@nama` pada API T-SQL | `?`, `?NNN`, `:nama` |
| **Katalog umum** | `pg_catalog`, `information_schema` | `information_schema` | `sys`, `INFORMATION_SCHEMA` | `sqlite_schema` |
| **Pembatas hasil** | `LIMIT` | `LIMIT` | `TOP` atau `OFFSET FETCH` | `LIMIT` |
| **Konkatenasi string** | `||` | `CONCAT()` atau `+` (mode memengaruhi) | `+` | `||` atau `CONCAT()` |
| **Komentar baris** | `--` | `-- ` (memerlukan *whitespace*), `#` | `--` | `--` |
| **Fungsi tunda waktu** | bergantung fungsi/ekspresi | `SLEEP()` | `WAITFOR DELAY` | tidak memiliki padanan bawaan yang sama |

Tabel tersebut adalah orientasi, bukan kamus *payload*. Versi, mode SQL, driver, ORM, filter aplikasi, dan posisi input dapat mengubah sintaks yang sah. Pengetahuan dialek membantu mengurangi *request* yang tidak perlu dan mencegah salah interpretasi. Sebagai contoh, `UNION` yang gagal dapat disebabkan jumlah kolom, tipe tidak kompatibel, konteks numerik, pemrosesan aplikasi, atau kerentanan yang memang tidak ada.

---

## 7.5 Akar Masalah: Data Menjadi Struktur Query

Perhatikan pseudocode rentan berikut:

```python
sql = "SELECT id, role FROM users WHERE username = '" + input + "'"
db.execute(sql)
```

Masalahnya terletak pada konkatenasi. Nilai input ditempelkan ke string perintah sebelum diparse. Karakter yang memiliki makna bagi SQL berpotensi mengakhiri literal, menambah operator, atau mengubah bagian lain dari pernyataan. Pemeriksaan bahwa input "tidak mengandung kata berbahaya" tidak stabil karena SQL menyediakan banyak representasi, komentar, fungsi, encoding, dan perbedaan dialek.

Versi aman mempertahankan struktur *query* sebagai konstanta dan mengirim nilai melalui kanal parameter:

```python
sql = "SELECT id, role FROM users WHERE username = ?"
db.execute(sql, [input])
```

Pada model kedua, *parser* melihat *placeholder* sebagai posisi nilai. Driver mengikat data sesuai tipe; isi data tidak diperlakukan sebagai token SQL. Parameterisasi merupakan kontrol struktural, bukan sekadar sanitasi tambahan. Namun, parameter hanya mewakili nilai. Nama tabel, nama kolom, arah urut, dan fragmen sintaks biasanya tidak dapat digantikan oleh *bind variable* dan perlu dipilih melalui *mapping allowlist* atau desain ulang.

---

## 7.6 Pemetaan Source-Transform-Query-Sink

Pengujian yang matang memetakan aliran, bukan sekadar mengirim tanda petik ke setiap *field*. *Source* dapat berupa *query parameter*, *path parameter*, form, JSON, header, cookie, pesan antrean, atau data yang dibaca kembali dari *database*. *Transform* meliputi *decoding*, normalisasi, *casting*, konkatenasi, pembuatan filter, dan pembentukan *query* oleh ORM. *Query* adalah struktur yang akhirnya dibuat. *Sink* ialah API yang mengeksekusi pernyataan terhadap DBMS.

| Elemen | Pertanyaan Penguji | *Evidence* yang Dicari |
| :--- | :--- | :--- |
| **Source** | Data mana dapat dikendalikan? | Raw request dan lokasi parameter |
| **Transform** | Apakah terjadi decode, cast, atau concat? | Perbedaan respons dan, bila tersedia, *code trace* |
| **Query context** | Literal string, numerik, identifier, atau clause? | Error terkontrol, pasangan kondisi, log lab |
| **Sink** | API parameterized atau dynamic execute? | *Code review*, telemetry, perilaku DBMS |
| **Effect** | Read, write, admin, file, atau OS? | Bukti minimum yang disepakati |

Model ini juga menjelaskan *second-order SQLi*. *Source* awal disimpan dengan aman, tetapi nilai tersebut menjadi *source* baru ketika dibaca dan dikonkatenasi ke *query* berikutnya. Oleh sebab itu, "input sudah pernah divalidasi" tidak menjamin aman pada *sink* baru.

---

## 7.7 Baseline dan Penemuan Kandidat Injeksi

*Baseline* merekam perilaku normal sebelum manipulasi: status HTTP, panjang dan struktur respons, string penanda, jumlah *redirect*, cookie baru, waktu respons, serta efek pada *state*. Gunakan beberapa sampel karena konten dinamis dan jaringan mempunyai variasi alami. Setelah itu, uji kontrol negatif dan perubahan kecil yang sesuai tipe input.

1. Kirim nilai sah dan simpan *raw request/response*;
2. Kirim nilai tidak ada tetapi tetap valid secara tipe;
3. Ubah satu karakter struktural pada lingkungan lab;
4. Bandingkan status, body, header, *redirect*, dan waktu;
5. Kembalikan input kontrol untuk melihat apakah perilaku pulih;
6. Ulangi kandidat positif agar dapat direproduksi.

Pesan error database adalah sinyal bernilai tinggi, tetapi bukan satu-satunya indikator. Aplikasi dapat menyembunyikan error, memetakan semua kegagalan ke halaman yang sama, atau mengembalikan status 200 untuk error. Sebaliknya, error setelah tanda petik belum tentu SQLi; validasi aplikasi, *template engine*, atau komponen lain dapat menolak karakter tersebut. Klaim harus ditopang pasangan eksperimen yang menjelaskan sebab.

---

## 7.8 Quote, Komentar, Tipe Data, dan Presedensi

Konteks menentukan bentuk eksperimen. Input di dalam literal string berbeda dari input numerik, daftar `IN`, clause `ORDER BY`, atau nama *identifier*. Tanda petik tunggal lazim untuk string, tetapi *query* dapat menggunakan parameter numerik tanpa petik. Menambahkan komentar dimaksudkan untuk menetralkan sisa *query*, tetapi aturan *whitespace* dan sintaks berbeda antar-DBMS.

Tipe data menambah batas lain. `UNION` dapat mempunyai jumlah kolom benar namun gagal karena kolom target tidak menerima tipe ekspresi. *Casting* eksplisit atau penggunaan nilai `NULL` pada laboratorium dapat membantu membedakan masalah tipe. Nilai `NULL` sendiri mengikuti logika tiga nilai: perbandingan terhadap `NULL` biasanya memerlukan `IS NULL`, bukan `= NULL`.

Presedensi `AND` dan `OR` menjelaskan mengapa contoh bypass yang tampak sederhana tidak universal. *Parentheses query* asli, pemeriksaan status akun, pemilihan baris pertama, *hashing password*, dan logika aplikasi setelah *query* dapat mencegah hasil yang dibayangkan. Fokus penguji adalah membuktikan perubahan struktur *query*, bukan memaksakan satu *payload* terkenal.

---

## 7.9 Pengujian Boolean-Based

*Boolean-based testing* menggunakan dua kondisi yang hanya berbeda pada nilai kebenaran. Jika input memengaruhi predikat SQL, kondisi benar dan salah dapat menghasilkan respons yang konsisten berbeda. Perbandingan dilakukan terhadap indikator stabil—misalnya keberadaan elemen, jumlah hasil, status, atau pola *redirect*—bukan hanya panjang body.

| Kondisi | Struktur / Predikat | Nilai Bisnis |
| :--- | :--- | :--- |
| **Baseline** | Nilai sah normal | Sah |
| **True Condition** | Predikat bernilai benar | Sesuai aturan |
| **False Condition** | Predikat bernilai salah | Kosong / Berbeda |
| **Control** | Input tidak valid yang tidak mengandung struktur SQL | Ditolak |

Setiap pasangan perlu diulang dan diselingi untuk mengurangi pengaruh *cache*, sesi, iklan dinamis, atau perubahan backend. Bila respons *true* dan *false* tidak dapat dibedakan secara konsisten, hasilnya *inconclusive*, bukan otomatis aman dan bukan otomatis rentan. Pengujian manual sebaiknya berhenti setelah mekanisme terbukti dengan marker non-sensitif; ekstraksi karakter demi karakter terhadap data nyata tidak diperlukan untuk *finding* yang dapat ditindaklanjuti.

---

## 7.10 Pengujian Error-Based

*Error-based SQLi* memanfaatkan kesalahan database untuk mengungkap pengaruh input atau, pada beberapa DBMS, membawa data dalam pesan error. Dari sisi defensif, error mendetail mempercepat *fingerprinting* karena dapat memuat nama driver, fungsi, tabel, atau fragmen *query*. Produksi sebaiknya menampilkan pesan generik kepada pengguna dan menyimpan detail pada log terlindungi.

Penguji harus membedakan tiga lapisan error: penolakan pada edge/WAF, error aplikasi, dan error DBMS. Status 500 tidak otomatis berarti database. Bandingkan body, *correlation ID*, header, dan—jika tim biru menyediakan—log internal. Kesalahan sintaks yang dapat dipicu ulang oleh satu perubahan input mendukung hipotesis, tetapi *confirmation* idealnya menggabungkan bukti lain seperti pasangan boolean atau *code review*.

Jangan mengandalkan error sebagai jalur ekstraksi massal. Pesan error dapat berisi data pribadi dan mungkin masuk ke sistem *observability* pihak lain. Gunakan nilai marker laboratorium, dokumentasikan *request* minimal, dan redaksikan keluaran sebelum pelaporan.

---

## 7.11 UNION-Based: Prasyarat dan Workflow

`UNION` menggabungkan hasil beberapa `SELECT`. Teknik ini relevan jika injeksi berada pada *query* baca dan aplikasi merender bagian *result set*. Ada tiga prasyarat: jumlah ekspresi pada kedua sisi harus sama; tipe pada posisi yang bersesuaian harus kompatibel; dan setidaknya satu kolom harus tampak pada respons atau memengaruhi alur yang dapat diamati.

1. Konfirmasi kandidat dengan eksperimen minimal;
2. Perkirakan jumlah kolom melalui perubahan `ORDER BY` atau varian `UNION` bernilai `NULL`;
3. Identifikasi posisi yang dirender menggunakan marker unik;
4. *Fingerprint* DBMS secukupnya;
5. Baca satu nilai marker yang telah disiapkan;
6. Hentikan dan simpan *evidence*.

`ORDER BY n` yang gagal setelah nilai tertentu merupakan inferensi, bukan hukum universal. Aplikasi dapat menyortir ulang, menangkap *exception*, atau menggunakan *query* berbeda. Demikian pula, keberhasilan `UNION` tidak memberi hak untuk mengakses data organisasi. Praktikum harus memakai tabel `lab_marker` yang sengaja dibuat dan tidak berisi data personal.

---

## 7.12 Metadata dan System Catalog

DBMS memelihara metadata tentang *schema*, tabel, kolom, tipe, *role*, dan objek lain. `information_schema` menyediakan antarmuka yang relatif standar, tetapi cakupannya berbeda. PostgreSQL memiliki `pg_catalog`; SQL Server mempunyai katalog `sys`; MySQL/MariaDB memperluas `information_schema`; SQLite menyimpan deskripsi objek dalam `sqlite_schema`.

Enumerasi metadata berisiko memperluas dampak dengan cepat. Daftar tabel dapat mengungkap nama sistem internal, fungsi bisnis, dan jejak aplikasi. Karena itu, penguji hanya mengambil metadata yang diperlukan untuk membuktikan akses di luar konteks normal. Dalam laporan, tuliskan kategori objek dan jumlah terbatas, bukan menyalin seluruh *schema*.

*Fingerprinting* yang baik menggabungkan sintaks, perilaku fungsi, pola error, header teknologi, dan metadata yang diizinkan. Nilai versi lengkap sering meningkatkan risiko operasional jika bocor. Jika versi bukan syarat temuan, cukup dokumentasikan keluarga DBMS dan tingkat keyakinan.

---

## 7.13 Blind SQL Injection sebagai Masalah Inferensi

Pada *blind SQLi*, aplikasi tidak menampilkan hasil *query* atau error yang berguna. Penguji memperoleh sebuah *oracle*: sinyal yang mewakili benar/salah. *Oracle* dapat berupa konten, status, *redirect*, jumlah item, perubahan header, atau waktu. Informasi kemudian direkonstruksi melalui serangkaian pertanyaan.

Kualitas *oracle* lebih penting daripada kecanggihan ekspresi. Respons dinamis perlu dinormalisasi dengan memilih string atau struktur yang stabil. Jika *oracle* salah, automasi akan menghasilkan data keliru atau berhenti di tengah. Dokumentasikan cara membedakan *true* dan *false*, jumlah pengulangan, serta kondisi jaringan.

*Blind* tidak berarti tidak ada dampak. Ia hanya menjelaskan kanal observasi. Namun, biaya *request* biasanya lebih tinggi. Ekstraksi panjang karakter per karakter dapat membebani aplikasi dan menghasilkan salinan data sensitif. Pendekatan aman menggunakan pertanyaan biner tentang marker laboratorium, misalnya apakah nilai memiliki panjang tertentu, lalu berhenti setelah hipotesis terkonfirmasi.

---

## 7.14 Time-Based Testing yang Terukur

*Time-based SQLi* memakai perbedaan waktu sebagai *oracle*. Satu respons lambat tidak cukup: *latency* dipengaruhi jaringan, *load balancer*, *garbage collection*, *cache*, kueri sah, dan antrean *worker*. Eksperimen harus membandingkan *baseline*, kontrol, kondisi *true* dengan delay pendek, serta kondisi *false* tanpa delay.

| Seri | Sampel | Median | Interpretasi |
| :--- | :--- | :--- | :--- |
| **Baseline** | 5 | 0,42 s | Variasi normal |
| **Kontrol** | 5 | 0,45 s | Tidak berbeda bermakna |
| **Kondisi false** | 5 | 0,44 s | *Oracle* tidak aktif |
| **Kondisi true + delay lab 2 s** | 5 | 2,46 s | Perbedaan konsisten |

Sampel sebaiknya di-*interleave*, bukan dijalankan sebagai dua blok jauh terpisah. Catat median dan rentang, bukan hanya angka tercepat. Delay harus sekecil mungkin, *request rate* dibatasi, dan pengujian dihentikan bila *health metric* memburuk. Jika variasi dasar mendekati besarnya delay, hasil harus dinyatakan *inconclusive* atau dikonfirmasi melalui *telemetry* internal.

---

## 7.15 Second-Order SQL Injection

*Second-order SQLi* muncul ketika nilai berbahaya diterima dan disimpan, lalu digunakan secara tidak aman di tempat lain. *Request* pertama mungkin tidak memicu error karena *insert* dilakukan dengan parameter. Kerentanan terjadi kemudian ketika *job* pelaporan, panel admin, ekspor, atau *stored procedure* membaca nilai itu dan membangun *dynamic SQL*.

```
[Input Berbahaya] ──► [Disimpan ke DB (Prepared Statement)] ──► [Dibaca di Fitur Lain (Concatenation Sink)] ──► [Eksekusi SQLi]
```

Pengujian *second-order* memerlukan korelasi antartahap dan izin yang lebih jelas karena *payload* dapat bertahan. Gunakan akun dan marker khusus, catat *identifier* rekaman, dan hapus artefak setelah praktikum. Dari sisi perbaikan, setiap *sink* harus melakukan parameterisasi sendiri. Data yang berasal dari database tidak otomatis terpercaya.

---

## 7.16 Autentikasi, Otorisasi, dan Mitos Bypass Login

Contoh SQLi sering diperkenalkan melalui login, sehingga timbul anggapan bahwa setiap SQLi pasti melewati autentikasi. Padahal keberhasilan bergantung pada *query*, *hashing*, pemilihan baris, status akun, *multi-factor authentication*, dan keputusan aplikasi setelah hasil *query*. Bahkan jika aplikasi menerima satu baris, sesi mungkin tetap tidak dibuat.

SQLi pada endpoint setelah login juga tidak hanya terkait autentikasi. Ia dapat melanggar otorisasi objek, misalnya pengguna melihat catatan milik akun lain. Dalam laporan, bedakan *authentication bypass*, *authorization bypass*, dan *arbitrary query influence*. Ketiganya mempunyai bukti dan rekomendasi tambahan berbeda.

Pengujian login juga sensitif terhadap *lockout* dan monitoring. Gunakan akun lab, batasi percobaan, dan koordinasikan dengan pemilik sistem. Jangan menyimpulkan bypass hanya karena halaman, status, atau pesan berubah; buktikan sesi dan kewenangan yang benar-benar diperoleh tanpa melakukan aktivitas lanjutan yang tidak perlu.

---

## 7.17 Dampak Write, Transaksi, dan Integritas Data

Jika injeksi terjadi pada `INSERT`, `UPDATE`, atau `DELETE`, eksperimen dapat mengubah banyak baris. Tipe ini tidak boleh diuji dengan kondisi luas pada produksi. Database mungkin memakai *autocommit*, *trigger*, *queue*, replikasi, atau audit yang membuat *rollback* aplikasi tidak menghapus seluruh efek samping.

Pada laboratorium, gunakan tabel marker, transaksi eksplisit, dan verifikasi sebelum-sesudah. Namun, keberadaan `ROLLBACK` bukan jaminan keselamatan: DDL tertentu, *external side effect*, *stored procedure*, atau koneksi terpisah dapat melampaui transaksi. Penguji perlu memahami konteks sebelum mengaktifkan opsi automasi berisiko.

Dampak integritas tidak selalu berupa penghapusan. Perubahan *role*, saldo, nilai akademik, *workflow state*, atau hubungan antarrekaman dapat memengaruhi proses bisnis. *Severity* ditentukan oleh data, hak akun database, *reachability*, dan kontrol kompensasi—bukan oleh label "SQLi" saja.

---

## 7.18 Dari DBMS ke Sistem Operasi

SQLi tidak identik dengan *remote code execution*. Jalur menuju sistem operasi memerlukan rangkaian prasyarat: teknik injeksi mendukung pernyataan yang relevan; akun database mempunyai hak tinggi; fitur administratif aktif; konteks OS dan *service account* memungkinkan aksi; *egress* atau *filesystem* dapat dijangkau; serta kontrol endpoint tidak memblokir.

PEN-200 membahas SQL Server `xp_cmdshell`. Microsoft mendokumentasikan bahwa fitur ini kuat dan dinonaktifkan secara default pada instalasi baru. Proses yang diluncurkan menggunakan hak akun layanan SQL Server atau *proxy* yang dikonfigurasi. Penguji tidak boleh mengaktifkan fitur pada target hanya untuk meningkatkan dampak; tindakan itu mengubah konfigurasi keamanan. Jika fitur sudah aktif dan pengujian diizinkan, marker tetap seperti identitas akun lab biasanya sudah cukup.

Fitur penulisan file pada DBMS lain juga tunduk pada *permission*, konfigurasi direktori, dan lokasi yang dapat dijangkau. Kemampuan teoritis tidak boleh dilaporkan sebagai RCE terkonfirmasi. Tulis prasyarat secara eksplisit: "potensi eskalasi apabila A, B, dan C terpenuhi," lalu bedakan dari *evidence* yang benar-benar diamati.

---

## 7.19 Automasi Terkendali dengan sqlmap

`sqlmap` mengotomasi deteksi dan eksploitasi berbagai teknik SQLi. Automasi berguna setelah penguji memahami *request*, sesi, parameter, *oracle*, dan *stop condition*. Ia bukan pengganti *baseline*. Simpan *raw request* dari *proxy* agar *method*, header, cookie, body, dan *content type* dapat direproduksi.

1. Konfirmasi kewenangan dan jendela uji;
2. Bangun hipotesis manual serta indikator *true/false*;
3. Jalankan deteksi pada satu parameter dengan pengaturan default rendah;
4. Batasi *thread*, delay, *timeout*, dan jumlah data;
5. Gunakan `--string`, `--not-string`, `--regexp`, atau `--code` bila respons dinamis memerlukan *oracle* yang jelas;
6. Naikkan `--level` atau `--risk` hanya dengan alasan dan persetujuan;
7. Hentikan setelah marker atau metadata minimum terverifikasi;
8. Arsipkan *command*, versi alat, waktu, dan log yang telah disanitasi.

Dokumentasi `sqlmap` menyatakan bahwa `--level` meningkatkan jumlah *payload* dan lokasi yang diuji. `--risk = 3` menambahkan pengujian berbasis `OR` yang pada konteks `UPDATE` dapat memengaruhi seluruh entri. *Tamper script* bukan langkah default dan bukan bukti kualitas; penggunaannya dapat meningkatkan volume, mengaburkan *audit trail*, dan berhadapan dengan aturan WAF yang tidak masuk *scope*. Jangan menggunakan opsi *dump* massal atau *takeover* tanpa otorisasi khusus.

---

## 7.20 Evidence, Confidence, dan Batas Klaim

*Evidence* yang baik menghubungkan input, perubahan terkontrol, observasi, dan interpretasi. Paket minimal memuat endpoint tersanitasi, parameter, prasyarat akun, *raw request*, potongan respons, waktu UTC, marker, pengulangan, serta dampak yang tidak diuji. *Screenshot* membantu pembaca, tetapi tidak menggantikan *raw request/response*.

| Tingkat Keyakinan | Contoh Bukti | Bahasa Laporan |
| :--- | :--- | :--- |
| **Indikatif** | Satu error setelah karakter khusus | "Indikasi memerlukan konfirmasi" |
| **Probable** | Pasangan *true/false* konsisten berbeda | "Input sangat mungkin memengaruhi predikat" |
| **Confirmed / Extended impact** | Marker lab terbaca atau *code trace* menunjukkan konkatenasi ke *sink* | "SQL injection terkonfirmasi" / "Dampak X terkonfirmasi dalam prasyarat Y" |

Jangan menaikkan klaim dari akses baca menjadi *write* atau RCE tanpa *evidence*. Catat juga keterbatasan: WAF, akun rendah, respons dinamis, *latency* tinggi, endpoint tidak stabil, atau pengujian dihentikan karena *stop condition*. Laporan yang jujur tentang ketidakpastian lebih berguna daripada narasi eksploitasi yang berlebihan.

---

## 7.21 Pencegahan Primer dengan Parameterized Query

Kontrol utama adalah memisahkan kode SQL dan nilai. OWASP merekomendasikan *prepared statement* atau *parameterized query*. PostgreSQL mendokumentasikan parameter posisi `$1`, `$2`, dan seterusnya; driver lain memakai *placeholder* berbeda. Prinsipnya tetap: pernyataan diparse sebagai struktur yang tetap, sedangkan nilai dikirim terpisah.

```python
statement = db.prepare("SELECT id, status FROM users WHERE email = ?")
row = statement.execute([email])
```

Validasi tipe tetap penting untuk aturan bisnis, kualitas data, dan pengurangan *attack surface*, tetapi bukan pengganti parameterisasi. *Escaping* manual merupakan pilihan rapuh karena bergantung pada dialek, *encoding*, *collation*, dan konfigurasi driver. *Stored procedure* aman hanya jika tidak membangun *dynamic SQL* dari input.

Perbaikan harus menyentuh semua jalur *query*, termasuk pencarian, ekspor, batch, laporan, admin, dan *job* asinkron. Satu endpoint yang diparameterisasi tidak menutup *second-order sink*. *Regression test* perlu memverifikasi fungsi sah serta input struktural pada setiap konteks.

---

## 7.22 Identifier Dinamis, ORM, dan Stored Procedure

*Bind variable* umumnya tidak dapat menggantikan nama tabel, kolom, atau arah urut karena elemen tersebut merupakan struktur. Solusinya adalah memetakan pilihan pengguna ke konstanta yang didefinisikan program:

```python
allowed_sort = {"nama": "nama", "tanggal": "created_at"}
column = allowed_sort.get(user_choice)
if column is None:
    reject()
sql = "SELECT id, nama FROM dokumen ORDER BY " + column + " ASC"
```

Mapping tidak boleh mengembalikan input mentah. Jika desain mengharuskan pengguna memilih *arbitrary identifier*, arsitektur patut ditinjau ulang. Arah `ASC`/`DESC` dapat dipilih dari boolean atau enum yang ketat.

ORM mengurangi kebutuhan menulis SQL, tetapi tidak menjamin kebal. *Raw query*, *string interpolation*, *dynamic filter*, HQL/JPQL, dan API "literal" dapat tetap rentan. *Stored procedure* juga dapat rentan jika memakai `EXEC`, `EXECUTE IMMEDIATE`, atau konkatenasi dinamis. *Code review* harus mengikuti data hingga operasi eksekusi, bukan berhenti pada label framework.

---

## 7.23 Defense in Depth dan Secure SDLC

Setelah akar masalah ditutup, *defense in depth* membatasi dampak dan mempercepat deteksi. Akun database aplikasi harus memiliki hak minimum; akun *read-only* tidak boleh mempunyai hak DDL, file, atau administratif. Aplikasi yang berbeda sebaiknya tidak berbagi *principal* berhak luas. Secret dikelola dan dirotasi melalui mekanisme yang sesuai, bukan ditanam pada source code.

Pesan error eksternal dibuat generik, sedangkan log internal menyimpan *correlation ID* dan detail secukupnya. Monitoring dapat mencari pola lonjakan error, *request* berulang, delay *query*, atau akses *schema* yang tidak biasa, tetapi *alert* bukan substitusi perbaikan. WAF dapat mengurangi *noise* dan membeli waktu; ia tidak menjamin pemisahan data dan perintah.

Pada *secure SDLC*, organisasi menggabungkan *code review*, SAST, DAST, IAST bila relevan, *unit test* untuk repository/*data-access layer*, serta *negative security test* di CI/CD. *Dependency* dan driver harus dipelihara. Temuan ditutup hanya setelah patch diverifikasi pada *sink* yang sama, test otomatis ditambahkan, dan tidak ada regresi fungsi bisnis.

---

## 7.24 Kerangka BINDING

Kerangka BINDING membantu mahasiswa mengubah pengujian menjadi *finding* yang dapat diaudit:

| Huruf | Langkah | Keluaran |
| :--- | :--- | :--- |
| **B** | Batasi kewenangan | Scope, operasi diizinkan, *stop condition* |
| **I** | Identifikasi *source* dan *sink* | Peta input-transform-query-eksekusi |
| **N** | Normalisasi *baseline* | Respons normal dan variasi alaminya |
| **D** | Diferensiasikan respons | Pasangan *true/false/error/time* yang terkontrol |
| **I** | Inventarisasi *evidence* | Raw request, marker, waktu, dan reproduksi |
| **N** | Nilai konteks dampak | Hak DB, DBMS, data, transaksi, dan prasyarat |
| **G** | Ganti akar masalah | Parameterisasi, mapping, *least privilege*, *regression test* |

BINDING bukan prosedur eksploitasi universal. Ia adalah *guardrail* untuk menghindari tiga kegagalan umum: menguji tanpa kewenangan, menyimpulkan dari satu anomali, dan merekomendasikan filter tanpa memperbaiki desain. Kerangka ini dapat dipakai bersama OWASP WSTG, proses pengujian organisasi, dan struktur laporan pada Bab 1.

---

## 7.25 Praktikum Terpandu dan Penilaian

Praktikum dilakukan pada **Portal Akademik Arunika** yang *vulnerable-by-design*. Dataset hanya berisi akun sintetis dan tabel marker. Mahasiswa bekerja berpasangan: satu orang sebagai penguji dan satu sebagai *observer* yang memeriksa scope serta *evidence*.

### Skenario A: Query Read dan UNION
- **Tujuan:** Menemukan parameter kandidat, menyusun baseline, membuktikan perubahan logika, menentukan prasyarat `UNION`, dan membaca satu marker. Dilarang mengekstrak tabel pengguna.
- **Deliverable:** Peta *source-to-sink*, tiga *raw request*, tabel perbandingan respons, dan rekomendasi parameterisasi.

### Skenario B: Blind dan Time-Based
- **Tujuan:** Membangun *oracle* stabil. Ambil sekurang-kurangnya lima baseline, *interleave* kondisi *true/false*, dan gunakan delay maksimum dua detik.
- **Deliverable:** Ledger waktu, median, rentang, kesimpulan *confidence*, dan alasan penghentian.

### Skenario C: Verifikasi Perbaikan
- **Tujuan:** Mengubah *repository function* rentan menjadi *parameterized query*, memetakan pilihan sort melalui *allowlist*, dan menambahkan *regression test*.
- **Deliverable:** *Diff* kode, test positif/negatif, serta catatan bahwa fungsi sah tetap bekerja.

| Komponen Penilaian | Bobot | Kriteria Utama |
| :--- | :--- | :--- |
| **Kewenangan dan keselamatan** | 20% | Scope dipatuhi; tidak ada data berlebihan |
| **Model query dan hipotesis** | 20% | Context, DBMS, dan *sink* dijelaskan |
| **Desain eksperimen** | 25% | Baseline, kontrol, pengulangan, *oracle* stabil |
| **Evidence dan batas klaim** | 20% | *Reproducible*, tersanitasi, *confidence* tepat |
| **Mitigasi dan regression test** | 15% | Akar masalah ditutup tanpa regresi |

---

## Rangkuman Bab

1. SQLi terjadi ketika data tidak dipercaya dapat mengubah struktur perintah SQL.
2. Teknik *in-band*, *error-based*, *boolean-based*, *time-based*, dan *out-of-band* menjelaskan kanal pengujian, bukan otomatis tingkat dampak.
3. Dialek, tipe data, komentar, katalog, dan fungsi berbeda antar-DBMS; *payload* tidak universal.
4. *Baseline*, kontrol, pengulangan, serta perubahan satu variabel membedakan *evidence* dari anomali.
5. `UNION` memerlukan jumlah kolom sama, tipe kompatibel, dan posisi yang dapat diamati.
6. *Blind SQLi* adalah masalah inferensi; kualitas *oracle* menentukan kualitas hasil.
7. SQLi menuju OS memerlukan fitur, hak, konfigurasi, dan konteks tertentu; potensi harus dipisahkan dari dampak terkonfirmasi.
8. `sqlmap` digunakan setelah hipotesis manual terbentuk, dengan *level*, *risk*, *rate*, dan volume data yang dibatasi.
9. *Parameterized query* adalah pencegahan primer; *identifier* dinamis memerlukan *mapping allowlist* atau desain ulang.
10. ORM dan *stored procedure* tidak otomatis aman jika masih membangun *dynamic SQL*.
11. BINDING mengikat kewenangan, model aliran, eksperimen, *evidence*, dampak, dan perbaikan.

---

## Uji Pemahaman

### Pilihan Ganda

1. Pernyataan yang paling tepat tentang akar SQL injection adalah:
   - A. DBMS menerima terlalu banyak koneksi.
   - **B. Data yang tidak dipercaya ikut menentukan struktur perintah SQL.**
   - C. Aplikasi menggunakan tabel dengan terlalu banyak kolom.
   - D. Server menampilkan halaman error berwarna merah.

2. Bukti terkuat untuk kandidat *boolean-based* pada *black-box test* adalah:
   - A. Satu respons berstatus 500.
   - B. Satu *request* membutuhkan waktu lebih lama.
   - **C. Pasangan kondisi true/false yang konsisten berbeda dan dapat direproduksi.**
   - D. Ditemukannya kata SQL pada JavaScript.

3. Syarat utama penggunaan `UNION` adalah:
   - A. Kedua query memakai nama tabel yang sama.
   - **B. Kedua result set memiliki jumlah kolom sama dan tipe yang kompatibel.**
   - C. Pengguna database harus menjadi administrator.
   - D. Aplikasi harus memakai cookie session.

4. Mengapa `--risk = 3` pada `sqlmap` perlu kehati-hatian?
   - A. Hanya bekerja pada SQLite.
   - B. Menghapus log lokal secara otomatis.
   - **C. Dapat menambahkan uji berbasis OR yang berbahaya pada konteks write.**
   - D. Menonaktifkan TLS.

5. Pernyataan yang benar tentang *prepared statement* adalah ....
   - A. Dapat mengikat semua nama tabel secara bebas.
   - **B. Memisahkan struktur query dan nilai yang diikat.**
   - C. Hanya berguna untuk meningkatkan performa.
   - D. Membuat *least privilege* tidak diperlukan.

6. Jika satu respons *time-based* terlambat empat detik, kesimpulan yang tepat adalah ...
   - A. RCE terkonfirmasi.
   - B. Database pasti MySQL.
   - C. SQLi terkonfirmasi tanpa pengulangan.
   - **D. Ada indikasi yang memerlukan baseline, kontrol, dan pengulangan.**

7. *Second-order SQLi* terjadi ketika:
   - A. Dua database direplikasi.
   - **B. Input disimpan lalu digunakan secara tidak aman pada query berikutnya.**
   - C. Query memiliki dua `SELECT`.
   - D. SQLi diuji dari dua alamat IP.

8. Pernyataan yang tepat tentang ORM adalah:
   - A. Semua API ORM otomatis menghapus SQLi.
   - **B. Raw query dan dynamic filter pada ORM tetap dapat rentan.**
   - C. ORM tidak pernah berkomunikasi dengan database.
   - D. ORM menggantikan kebutuhan otorisasi.

9. Mengapa penguji tidak boleh mengaktifkan `xp_cmdshell` pada target hanya untuk membuktikan dampak?
   - A. Karena fitur hanya tersedia pada browser.
   - **B. Karena tindakan tersebut mengubah konfigurasi keamanan dan melampaui bukti minimum.**
   - C. Karena `xp_cmdshell` selalu aktif.
   - D. Karena fitur tidak menjalankan proses OS.

10. Kontrol paling tepat untuk pilihan kolom sort yang berasal dari pengguna adalah ...
    - A. Menempelkan input setelah menghapus spasi.
    - B. Mengganti tanda petik dengan dua tanda petik.
    - **C. Memetakan pilihan ke konstanta kolom yang diizinkan.**
    - D. Menyimpan input di cookie.

11. Kapan hasil pengujian sebaiknya dinyatakan *inconclusive*?
    - **A. Ketika true dan false tidak dapat dibedakan secara stabil.**
    - B. Ketika ditemukan satu marker laboratorium.
    - C. Ketika *code trace* menunjukkan konkatenasi ke *sink*.
    - D. Ketika *regression test* lulus.

12. Langkah terakhir kerangka BINDING menekankan:
    - A. Dump seluruh database.
    - **B. Penggantian akar masalah dan regression test.**
    - C. Pengaktifan fungsi administratif.
    - D. Penghapusan semua log.

#### Kunci Jawaban Pilihan Ganda
1. B | 2. C | 3. B | 4. C | 5. B | 6. D | 7. B | 8. B | 9. B | 10. C | 11. A | 12. B

---

### Pertanyaan Esai

1. Jelaskan perbedaan mendasar antara validasi input, *escaping*, dan *parameterized query*. Berikan contoh kapan masing-masing digunakan.
2. Rancang eksperimen *boolean-based* yang mempunyai *baseline*, *true condition*, *false condition*, dan kontrol negatif. Jelaskan kriteria konfirmasi.
3. Mengapa satu respons lambat tidak cukup untuk mengonfirmasi *time-based SQLi*? Susun metode pengukuran yang lebih kuat.
4. Bandingkan metadata catalog PostgreSQL, MySQL/MariaDB, SQL Server, dan SQLite serta implikasinya bagi *fingerprinting*.
5. Jelaskan prasyarat yang harus dipenuhi sebelum SQLi dapat mencapai eksekusi perintah OS. Bedakan potensi dan dampak terkonfirmasi.
6. Audit pseudocode fitur *sorting* dinamis dan usulkan desain yang aman bila nama kolom tidak dapat diikat sebagai parameter.
7. Susun *finding* SQLi lengkap yang memuat *evidence*, *confidence*, dampak, keterbatasan, akar masalah, dan *regression test*.
8. Analisis bagaimana *second-order SQLi* dapat muncul pada *batch report* meskipun form awal menggunakan *prepared statement*.

---

## Glosarium

- **Bind variable:** Placeholder yang nilainya dikirim terpisah dari struktur query.
- **Blind SQL injection:** SQLi yang tidak menampilkan hasil langsung; informasi disimpulkan dari oracle.
- **DBMS:** Perangkat lunak pengelola database, misalnya PostgreSQL, MySQL, SQL Server, atau SQLite.
- **Dynamic SQL:** Pernyataan SQL yang strukturnya dibentuk ketika program berjalan.
- **Error-based SQLi:** Teknik yang memakai perilaku error database sebagai sinyal.
- **Evidence:** Artefak yang dapat diverifikasi untuk mendukung suatu klaim.
- **Identifier:** Nama objek SQL, seperti tabel, kolom, schema, atau fungsi.
- **In-band:** Injeksi dan penerimaan hasil menggunakan kanal aplikasi yang sama.
- **Oracle:** Sinyal yang memungkinkan penguji menyimpulkan hasil true/false.
- **ORM:** Lapisan object-relational mapping yang memetakan objek program ke data relasional.
- **Parameterized query:** Query berstruktur tetap dengan nilai yang diikat melalui parameter.
- **Prepared statement:** Pernyataan yang dipersiapkan dengan placeholder untuk eksekusi satu atau beberapa kali.
- **Second-order SQLi:** SQLi ketika data tersimpan digunakan secara tidak aman pada sink kemudian.
- **Sink:** API atau operasi yang mengeksekusi query terhadap DBMS.
- **SQL injection:** Pengaruh input tidak dipercaya terhadap struktur perintah SQL.
- **System catalog:** Metadata internal DBMS tentang objek dan hak akses.
- **Time-based SQLi:** Teknik inferensi yang memakai perbedaan waktu sebagai oracle.
- **UNION-based SQLi:** Teknik in-band yang menggabungkan result set tambahan melalui UNION.

---

## Referensi

1. OffSec. *"OSCP+ Body of Knowledge."* 2026. [Link](https://help.offsec.com/hc/en-us/articles/38543335188756-OSCP-Body-of-Knowledge)
2. OffSec. *"PEN-200 Learning Plan - 12 Week."* 2025. [Link](https://help.offsec.com/hc/en-us/articles/15541765522196-OffSec-PEN-200-Learning-Plan-12-Week)
3. OWASP Foundation. *"A05:2025 Injection."* 2025. [Link](https://owasp.org/Top10/2025/A05_2025-Injection/)
4. OWASP Foundation. *"Testing for SQL Injection."* Web Security Testing Guide. 2026. [Link](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection)
5. OWASP Foundation. *"SQL Injection Prevention Cheat Sheet."* 2026. [Link](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
6. OWASP Foundation. *"Query Parameterization Cheat Sheet."* 2026. [Link](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html)
7. OWASP Foundation. *"Database Security Cheat Sheet."* 2026. [Link](https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html)
8. OWASP Foundation. *"Injection Prevention Cheat Sheet."* 2026. [Link](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
9. OWASP Foundation. *"Testing for ORM Injection."* 2026. [Link](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.7-Testing_for_ORM_Injection)
10. MITRE. *"CWE-89: Improper Neutralization of Special Elements used in an SQL Command."* 2026. [Link](https://cwe.mitre.org/data/definitions/89.html)
11. PostgreSQL Global Development Group. *"PREPARE" PostgreSQL 18 Documentation.* 2026. [Link](https://www.postgresql.org/docs/current/sql-prepare.html)
12. PostgreSQL Global Development Group. *"Command Execution Functions."* 2026. [Link](https://www.postgresql.org/docs/current/libpq-exec.html)
13. Oracle. *"Prepared Statements."* MySQL Reference Manual. 2026. [Link](https://dev.mysql.com/doc/refman/8.4/en/sql-prepared-statements.html)
14. Oracle. *"The INFORMATION_SCHEMA Tables."* MySQL Reference Manual. 2026. [Link](https://dev.mysql.com/doc/refman/8.4/en/information-schema.html)
15. Microsoft. *"sp_executesql (Transact-SQL)."* 2026. [Link](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql)
16. Microsoft. *"Server Configuration: xp_cmdshell."* 2026. [Link](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
17. Microsoft. *"xp_cmdshell (Transact-SQL)."* 2025. [Link](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql)
18. SQLite. *"Binding Values to Prepared Statements."* 2026. [Link](https://www.sqlite.org/c3ref/bind_blob.html)
19. SQLite. *"The Schema Table."* 2026. [Link](https://www.sqlite.org/schematab.html)
20. sqlmap project. *"sqlmap: Automatic SQL Injection and Database Takeover Tool."* 2026. [Link](https://github.com/sqlmapproject/sqlmap)
21. sqlmap project. *"Usage."* sqlmap Wiki. 2026. [Link](https://github.com/sqlmapproject/sqlmap/wiki/usage)
22. PortSwigger. *"SQL Injection."* Web Security Academy. 2026. [Link](https://portswigger.net/web-security/sql-injection)
23. PortSwigger. *"SQL Injection Cheat Sheet."* 2026. [Link](https://portswigger.net/web-security/sql-injection/cheat-sheet)
24. NIST. *"Technical Guide to Information Security Testing and Assessment."* SP 800-115. 2008. [Link](https://csrc.nist.gov/pubs/sp/800/115/final)
25. FIRST. *"Common Vulnerability Scoring System Version 4.0."* 2023. [Link](https://www.first.org/cvss/v4.0/)
26. IETF. *"HTTP Semantics."* RFC 9110. 2022. [Link](https://www.rfc-editor.org/rfc/rfc9110)
27. OWASP Foundation. *"Logging Cheat Sheet."* 2026. [Link](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
28. OWASP Foundation. *"Input Validation Cheat Sheet."* 2026. [Link](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

---
*Seluruh sumber daring diakses pada 31 Agustus 2026. Naskah ini merupakan sintesis orisinal untuk tujuan pendidikan; contoh disusun ulang dan dibatasi pada lingkungan laboratorium yang berizin.*