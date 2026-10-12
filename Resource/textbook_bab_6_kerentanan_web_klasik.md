# TEXTBOOK CYBERSECURITY BAB 6
## PROGRAM SARJANA

# Kerentanan Web Klasik: Path, File, Upload, dan Command Injection
**Dari input dan interpreter boundary menuju evidence serta mitigasi yang dapat diaudit**

**Hatma Suryotrisongko**

*Materi latihan hanya untuk lingkungan milik sendiri atau yang memiliki izin tertulis.*

---

## Isi Bab

Bab ini bergerak dari model input-transformasi-sink menuju path traversal, file inclusion, file upload, OS command injection, exploit chain, kerangka LINTAS, dan praktikum berbasis evidence.

- 6.1 OWASP Top 10 sebagai Peta, Bukan Daftar Payload
- 6.2 Model Mental: Data, Transformasi, Sink, dan Interpreter
- 6.3 Kewenangan, Stop Condition, dan Minimisasi Dampak
- 6.4 Absolute Path, Relative Path, dan Canonical Path
- 6.5 Anatomi Path Traversal
- 6.6 Workflow Pengujian Traversal yang Sistematis
- 6.7 Encoding, Decoding, dan Normalization Gap
- 6.8 Dampak dan Pencegahan Path Traversal
- 6.9 Directory Traversal dan File Inclusion Bukan Sinonim
- 6.10 Local File Inclusion (LFI)
- 6.11 PHP Stream Wrapper
- 6.12 Dari LFI ke Eksekusi Kode
- 6.13 Remote File Inclusion (RFI) dan Konteks Modern
- 6.14 File Upload adalah Pipeline
- 6.15 Workflow Pengujian File Upload
- 6.16 Unggahan Executable
- 6.17 Unggahan Non-Executable
- 6.18 Archive Extraction dan Resource Exhaustion
- 6.19 OS Command Injection
- 6.20 Operator, Quoting, dan Perbedaan Platform
- 6.21 Direct, Blind, Time-Based, dan Out-of-Band
- 6.22 Evidence, Reproduksibilitas, dan Batas Klaim
- 6.23 Mitigasi sebagai Perubahan Arsitektur
- 6.24 Exploit Chain dan Kerangka LINTAS
- 6.25 Praktikum Terpandu dan Penilaian

> *Catatan akademik. Contoh pada bab ini menggunakan aplikasi fiktif atau laboratorium vulnerable-by-design. Pembaca dilarang mempraktikkan teknik pada sistem publik tanpa otorisasi eksplisit.*

---

## Pendahuluan

Bab sebelumnya membangun fondasi untuk membaca aplikasi web sebagai sistem: request dan response, state, origin, proxy intersep, endpoint inventory, serta evidence. Bab ini menggunakan fondasi tersebut untuk menguji empat keluarga kerentanan yang tampak klasik, tetapi tetap relevan pada aplikasi modern: path traversal, file inclusion, file upload vulnerability, dan OS command injection.

Keempat keluarga tersebut memiliki pola akar masalah yang serupa. Data yang seharusnya hanya menjadi nilai bisnis—nama dokumen, bahasa antarmuka, berkas gambar, alamat host, atau opsi utilitas—menyeberangi trust boundary dan kemudian diperlakukan sebagai instruksi oleh interpreter lain: resolver path, mesin template, runtime aplikasi, web server, pengurai arsip, atau shell sistem operasi. Karena itu, pengujian yang baik tidak dimulai dengan menghafal payload, melainkan dengan menelusuri transformasi data dari input hingga security-sensitive sink.

Struktur bab ini mengadaptasi kompetensi PEN-200 Bab 9, *Common Web Application Attacks*: absolute dan relative path, directory traversal, encoding karakter khusus, LFI, PHP wrapper, RFI, penyalahgunaan unggahan executable dan non-executable, serta OS command injection [1][2]. Materi diperluas dengan OWASP Top 10:2025, OWASP Web Security Testing Guide (WSTG), OWASP Cheat Sheet Series, CWE, dan dokumentasi resmi PHP.

> **Batas etika dan keselamatan.** Seluruh eksperimen hanya boleh dijalankan pada aplikasi milik sendiri, laboratorium vulnerable-by-design, atau target yang secara eksplisit tercantum dalam otorisasi tertulis. Gunakan akun dan data uji. Jangan mengambil credential, data pribadi, atau berkas sistem lebih banyak daripada yang diperlukan untuk membuktikan risiko. Hentikan pengujian apabila terjadi degradasi layanan, perubahan state yang tidak direncanakan, atau indikasi bahwa request mencapai sistem di luar scope.

### Capaian Pembelajaran Bab

Setelah mempelajari bab ini, mahasiswa mampu:
1. menjelaskan kedudukan OWASP Top 10 sebagai dokumen awareness, bukan checklist pengujian lengkap;
2. menelusuri aliran data dari input menuju resolver path, fungsi include, penyimpanan file, atau shell;
3. membedakan absolute path, relative path, canonical path, path traversal, dan file inclusion;
4. merancang pengujian traversal yang bertahap, terukur, dan berbasis baseline;
5. menjelaskan pengaruh encoding, decoding berlapis, separator platform, dan normalisasi path;
6. membedakan LFI, RFI, pembacaan source code, dan eksekusi kode berdasarkan prasyaratnya;
7. memetakan ancaman file upload pada seluruh siklus hidup berkas;
8. menguji file executable dan non-executable secara aman di laboratorium;
9. membedakan direct, blind, time-based, dan out-of-band command injection;
10. menilai kekuatan evidence dan membatasi klaim sesuai hasil observasi;
11. merumuskan mitigasi berlapis pada desain, implementasi, konfigurasi, dan operasi; dan
12. menyusun finding yang dapat direproduksi tanpa meningkatkan dampak secara tidak perlu.

### Peta Konsep

Alur analisis bab ini dapat diringkas sebagai berikut:
$$\text{input tidak tepercaya} \rightarrow \text{transformasi/decoding} \rightarrow \text{security-sensitive sink} \rightarrow \text{interpreter} \rightarrow \text{efek teramati} \rightarrow \text{dampak} \rightarrow \text{evidence} \rightarrow \text{mitigasi}$$

Empat pertanyaan menjadi pengendali:
1. Data apa yang dapat dikendalikan pengguna?
2. Transformasi apa yang terjadi sebelum data digunakan?
3. Komponen mana yang akhirnya menafsirkan data?
4. Bukti minimum apa yang cukup untuk menunjukkan pelanggaran boundary?

---

## 6.1 OWASP Top 10 sebagai Peta, Bukan Daftar Payload

OWASP Top 10:2025 merupakan dokumen awareness mengenai sepuluh kategori risiko aplikasi web yang dinilai paling kritis. Versi 2025 mencantumkan A01 Broken Access Control, A02 Security Misconfiguration, A03 Software Supply Chain Failures, A04 Cryptographic Failures, A05 Injection, A06 Insecure Design, A07 Authentication Failures, A08 Software or Data Integrity Failures, A09 Security Logging and Alerting Failures, serta A10 Mishandling of Exceptional Conditions [3].

Versi tersebut berbeda dari OWASP Top 10:2021. Perubahan versi menunjukkan bahwa Top 10 berkembang mengikuti data dan tren; mahasiswa tidak boleh menyatakan versi 2021 sebagai versi paling mutakhir. Namun, perubahan urutan atau nama kategori tidak membuat kerentanan teknis lama menjadi tidak relevan.

| Keluarga Kerentanan | Kategori OWASP 2025 yang Sering Terkait | CWE Representatif | Catatan |
| :--- | :--- | :--- | :--- |
| Path traversal | A01 Broken Access Control, A05 Injection | CWE-22, CWE-73 | Klasifikasi bergantung pada mekanisme dan boundary yang dilanggar |
| File inclusion | A05 Injection, A01 Broken Access Control | CWE-98, CWE-73 | Inclusion tidak identik dengan traversal |
| File upload berbahaya | A02 Security Misconfiguration, A05 Injection, A08 Software or Data Integrity Failures | CWE-434 | Satu fitur dapat memunculkan beberapa kelas risiko |
| OS command injection | A05 Injection | CWE-78 | Data melewati boundary menuju command interpreter |

Top 10 membantu komunikasi risiko, sedangkan WSTG memberi tujuan dan teknik pengujian yang lebih operasional. Keduanya tetap perlu dilengkapi threat model, business logic testing, dan pemahaman arsitektur aplikasi. Tidak menemukan satu kategori Top 10 bukan bukti bahwa aplikasi aman.

---

## 6.2 Model Mental: Data, Transformasi, Sink, dan Interpreter

Kerentanan pada bab ini dapat dijelaskan tanpa bergantung pada bahasa pemrograman tertentu. Sebuah aplikasi menerima input, melakukan satu atau lebih transformasi, lalu menyerahkan hasilnya kepada komponen sensitif.

| Tahap | Contoh | Pertanyaan Pengujian |
| :--- | :--- | :--- |
| **Source** | query `file=manual.pdf`, filename multipart, JSON `host` | Bagian mana yang dikendalikan pengguna? |
| **Transformasi** | URL decode, concatenation, extension append, case folding | Berapa kali data di-decode dan dinormalisasi? |
| **Sink** | `open()`, `include()`, penyimpanan web root, `system()` | Komponen atau subsistem apa yang menerima data? |
| **Efek** | file terbaca, script dirender, file dieksekusi, proses berjalan | Observasi apa yang membedakan keberhasilan dari error biasa? |
| **Dampak** | confidentiality, integrity, availability, code execution | Boundary apa yang benar-benar dilanggar? |

Contoh konseptual yang rentan:
- **Pengguna:** `../../catatan.txt`
- **Aplikasi (Base Dir):** `/srv/app/public/`
- **Gabungan:** `/srv/app/public/../../catatan.txt`
- **Setelah Normalisasi:** `/srv/catatan.txt`

Risiko muncul bukan karena karakter titik selalu berbahaya, melainkan karena aplikasi mengizinkan data pengguna memilih objek file di luar root yang dimaksud. Prinsip yang sama berlaku saat input menjadi bagian command: karakter tertentu memperoleh makna baru ketika ditafsirkan oleh shell.

---

## 6.3 Kewenangan, Stop Condition, dan Minimisasi Dampak

Pengujian file dan command dapat mengubah sistem. Upload dapat memenuhi storage, menimpa nama, memicu antivirus, atau menjalankan worker asinkron. Command injection dapat menambah proses, mengakses jaringan, atau menghasilkan beban. Oleh sebab itu, test plan harus mendefinisikan target, akun, jenis bukti, batas request, dan stop condition.

### 6.3.1 Aturan Praktikum Minimum
1. Gunakan placeholder seperti `<LAB-HOST>`, bukan domain publik.
2. Pilih artefak bukti tidak sensitif, misalnya file laboratorium `proof.txt` atau nilai marker unik.
3. Gunakan command non-destruktif dan berdurasi pendek.
4. Jangan mengunggah web shell, reverse shell, atau payload persistensi pada sistem produksi.
5. Samarkan cookie, bearer token, alamat internal, dan data pribadi pada laporan.
6. Catat semua request yang berpotensi mengubah state.
7. Bersihkan berkas uji setelah eksperimen apabila aturan laboratorium mengizinkan.

**Prinsip proof-before-impact.** Setelah kontrol terbukti dapat dilewati, hentikan eskalasi apabila bukti yang tersedia sudah cukup untuk mendukung severity. Kemampuan teoritis mengambil banyak data tidak perlu dibuktikan dengan benar-benar mengambilnya.

---

## 6.4 Absolute Path, Relative Path, dan Canonical Path

Absolute path menunjuk lokasi mulai dari root atau volume. Contohnya `/var/www/app/config.yml` pada Linux dan `C:\inetpub\wwwroot\web.config` pada Windows. Relative path ditafsirkan terhadap working directory atau base directory tertentu, misalnya `images/avatar.png` atau `..\documents\guide.pdf`.

Canonicalization adalah proses menghasilkan representasi path yang dinormalisasi sesuai aturan platform atau pustaka. Proses dapat mencakup resolusi `.` dan `..`, separator, symbolic link, case sensitivity, mount point, drive letter, atau network path. Canonical path bukan sekadar menghapus substring `../`; ia harus dievaluasi terhadap base directory yang dipercaya.

| Konsep | Linux/Unix | Windows | Implikasi |
| :--- | :--- | :--- | :--- |
| **Root** | `/` | Drive seperti `C:\` | Absolute path berbeda antarplatform |
| **Separator umum** | `/` | `\` (sering `/` diterima API tertentu) | Filter satu separator dapat tidak lengkap |
| **Case** | Umumnya sensitif | Umumnya tidak sensitif pada NTFS default | Allowlist perlu mengikuti filesystem |
| **Link** | Symbolic link, mount | Junction, symlink, reparse point | Hasil lexical normalization belum tentu lokasi final |

Pencegahan yang kokoh memisahkan identifier bisnis dari path fisik. Sebagai contoh, klien mengirim `document_id = $id$`; server memetakan identifier tersebut ke objek yang telah terotorisasi. Jika nama file memang diperlukan, server membuat path canonical lalu memastikan hasil akhirnya tetap berada di direktori yang diizinkan.

---

## 6.5 Anatomi Path Traversal

Path traversal terjadi ketika aplikasi membiarkan input pengguna memengaruhi path sehingga objek di luar direktori yang dimaksud dapat diakses. OWASP menyebutnya juga *dot-dot-slash attack*, *directory climbing*, atau *backtracking* [4]. Dampaknya tidak selalu hanya membaca file; tergantung operasi sink, traversal dapat digunakan untuk menulis, menghapus, menimpa, atau memilih file yang salah.

Kandidat entry point mencakup:
1. parameter `file`, `path`, `page`, `template`, `lang`, `document`, `download`, atau `image`;
2. path segment seperti `/download/<name>`;
3. cookie atau header yang memilih tema dan template;
4. filename dalam `multipart/form-data`;
5. nama anggota archive ZIP/TAR;
6. parameter export/import dan lokasi backup; serta
7. API yang menerima storage key atau object path.

Contoh request laboratorium:
```http
GET /download?file=guide.pdf HTTP/1.1
Host: <LAB-HOST>
Cookie: session=<REDACTED>
```

Hipotesis awal bukan "aplikasi pasti rentan", melainkan "nilai file mungkin digunakan untuk memilih path pada server". Penguji perlu membandingkan response baseline, path yang tidak ada, input nested yang sah, dan variasi traversal.

---

## 6.6 Workflow Pengujian Traversal yang Sistematis

WSTG memisahkan pengujian menjadi enumerasi input vector dan evaluasi teknik [4]. Praktiknya dapat diubah menjadi workflow enam langkah.

1. **Bangun baseline.** Kirim request sah beberapa kali. Catat status, ukuran, content type, checksum atau marker body, redirect, dan waktu.
2. **Uji kontrol negatif.** Gunakan nama acak yang pasti tidak ada untuk memahami error model.
3. **Uji perubahan struktur.** Gunakan subdirektori sah atau variasi nama yang tidak sensitif untuk mengetahui cara aplikasi menyusun path.
4. **Uji boundary secara minimum.** Pada laboratorium, gunakan target bukti yang tidak sensitif seperti `/etc/hostname`, `/proc/version`, atau `C:\Windows\win.ini` sesuai OS.
5. **Variasikan encoding secara terukur.** Ubah satu dimensi pada satu waktu dan simpan raw request.
6. **Verifikasi dan batasi klaim.** Ulangi observasi, bandingkan dengan baseline, lalu nyatakan hanya operasi yang terbukti: *read*, *write*, *delete*, atau *include*.

Contoh request traversal:
```http
GET /download?file=../../../../etc/hostname HTTP/1.1
Host: <LAB-HOST>
Cookie: session=<REDACTED>
```

Response status `200 OK` saja tidak membuktikan traversal. Aplikasi dapat selalu mengembalikan halaman error dengan status `200`. Evidence yang lebih kuat adalah content yang konsisten dengan file target, perubahan deterministik saat target diubah, dan reproduksi pada request yang sama.

### Matriks Evidence

| Observasi | Kekuatan | Klaim yang Wajar |
| :--- | :--- | :--- |
| Status berbeda untuk `../` | Rendah | Input diproses berbeda; perlu pengujian lanjutan |
| Error membocorkan path server | Sedang | Informasi path terungkap; traversal belum terbukti |
| Marker file lab terbaca | Tinggi | Pembacaan file di luar root terbukti |
| File lab di luar root berubah | Sangat tinggi | Traversal tulis terbukti; segera hentikan eskalasi |

---

## 6.7 Encoding, Decoding, dan Normalization Gap

Encoding bukan "sihir bypass", melainkan cara merepresentasikan byte atau karakter pada lapisan berbeda. URL parser, reverse proxy, framework, dan aplikasi dapat melakukan decoding atau normalisasi pada urutan yang berbeda. Ketidaksesuaian antar-lapisan menciptakan *normalization gap*.

| Bentuk Konseptual | Contoh | Pertanyaan |
| :--- | :--- | :--- |
| Literal | `../` | Apakah filter memeriksa sebelum normalisasi? |
| Percent-encoded | `%2e%2e%2f` | Lapisan mana yang melakukan URL decode? |
| Double-encoded | `%252e%252e%252f` | Apakah terjadi decoding lebih dari sekali? |
| Separator alternatif | `..\/` | Apakah server/framework menerima separator Windows? |
| Mixed form | `..%2f` | Apakah validasi dan sink melihat representasi berbeda? |

Null byte (`%00`) sering muncul dalam materi historis. Pada runtime lama, terminator null dapat membuat validasi dan fungsi native melihat panjang string yang berbeda. Pada stack modern teknik tersebut umumnya tidak berlaku; kegagalan atau perubahan response tidak boleh dianggap bukti. Pembahasan null byte berguna untuk memahami perbedaan abstraksi, bukan sebagai payload universal.

Mitigasi tidak boleh hanya berupa denylist `../`. Lakukan satu proses decode yang terdefinisi, tolak encoding tidak valid, normalisasi dengan API filesystem yang sesuai, bandingkan hasil canonical terhadap root yang diizinkan, dan lakukan otorisasi atas objek final.

---

## 6.8 Dampak dan Pencegahan Path Traversal

Dampak dipengaruhi oleh operasi, hak proses, dan isi lingkungan. Pembacaan file konfigurasi dapat mengungkap secret; write traversal dapat menimpa content; delete traversal dapat mengganggu ketersediaan; dan traversal dalam ekstraksi archive dapat menulis file di luar direktori tujuan (Zip Slip).

Kontrol defensif berlapis meliputi:
1. gunakan object identifier yang dipetakan server, bukan path dari klien;
2. simpan file pada root khusus dengan permission minimum;
3. canonicalize lalu verifikasi containment terhadap root yang dipercaya;
4. gunakan allowlist nama atau tipe objek bisnis;
5. hasilkan filename acak di server;
6. tolak absolute path dan karakter/encoding ambigu bila tidak diperlukan;
7. cegah symlink/junction mengikuti lokasi di luar root sesuai API yang tersedia;
8. jalankan proses dengan least privilege; dan
9. log penolakan traversal tanpa menyimpan secret atau payload mentah secara berlebihan.

Validasi harus dilakukan pada server. Validasi JavaScript di browser hanya meningkatkan pengalaman pengguna dan dapat dilewati dengan mengirim request langsung.

---

## 6.9 Directory Traversal dan File Inclusion Bukan Sinonim

Traversal membahas pemilihan lokasi file di luar boundary. File inclusion membahas cara aplikasi memuat atau menafsirkan file melalui mekanisme dinamis. Satu endpoint dapat memiliki keduanya, tetapi dampak ditentukan oleh sink.

| Mekanisme | Sink Konseptual | Efek yang Mungkin |
| :--- | :--- | :--- |
| Download file | Read/stream file | Disclosure |
| Render template | Template/include engine | Disclosure atau interpretasi template |
| Include PHP | `include`/`require` | Interpretasi sebagai PHP sesuai konteks |
| Static file server | Response content | File dikirim sebagai data, bukan otomatis dieksekusi |

Kesalahan umum adalah menyimpulkan bahwa setiap LFI berarti *remote code execution* (RCE). LFI dapat hanya menampilkan file, menghasilkan error, atau menginterpretasikan file. RCE memerlukan prasyarat tambahan: konten yang dikendalikan penyerang, lokasi yang dapat direferensikan, hak baca, format yang diproses sebagai kode, serta konfigurasi runtime yang sesuai.

---

## 6.10 Local File Inclusion (LFI)

LFI muncul ketika input memengaruhi file lokal yang dimuat oleh aplikasi. Contoh pola rentan:
```php
<?php
include($_GET['page'] . '.php');
?>
```

Suffix `.php` bukan mitigasi yang memadai. Efek akhir dipengaruhi normalisasi, kemampuan traversal, wrapper, dan versi runtime. Dalam pengujian, bedakan tiga tujuan:
1. membuktikan pemilihan file di luar allowlist;
2. membuktikan disclosure source atau data; dan
3. membuktikan interpretasi kode—hanya jika scope dan prasyarat mendukung.

Sinyal LFI dapat berupa error yang menyebut `include_path`, perubahan template, isi file yang khas, atau marker file laboratorium. Error path merupakan information disclosure, tetapi tidak sendirinya membuktikan inclusion berhasil.

### 6.10.1 Pencegahan LFI
Pilihan desain terbaik adalah tidak pernah meneruskan input pengguna ke fungsi include. Gunakan pemetaan statis:
- `"id"` $\rightarrow$ `templates/id.php`
- `"en"` $\rightarrow$ `templates/en.php`
- *selain nilai tersebut* $\rightarrow$ tolak

Tambahkan least privilege, pisahkan code dari uploaded content, nonaktifkan konfigurasi yang tidak diperlukan, dan hindari error detail kepada klien.

---

## 6.11 PHP Stream Wrapper: Fitur yang Mengubah Cara File Dibaca

PHP menyediakan stream wrapper untuk mengakses resource dengan skema tertentu. `php://filter` adalah meta-wrapper yang menerapkan filter ketika stream dibuka; dokumentasi PHP menjelaskan penggunaannya bersama fungsi seperti `readfile()`, `file()`, dan `file_get_contents()` [12]. Dalam konteks LFI laboratorium, filter `convert.base64-encode` dapat membuat source PHP dikembalikan sebagai data terenkode sehingga tidak langsung diinterpretasikan.

```text
php://filter/convert.base64-encode/resource=<LAB-FILE>
```

Temuan harus mencatat fungsi yang teramati, wrapper yang diterima, resource yang diakses, serta apakah output dapat didekode menjadi source. Jangan menyimpulkan semua wrapper tersedia. Dukungan dan pembatasannya berbeda; misalnya dokumentasi PHP mencatat konfigurasi yang mengatur URL-aware wrapper, sedangkan wrapper tertentu memiliki aturan tersendiri [13][14].

Defensive review perlu mencari dynamic include, penggunaan wrapper yang tidak diperlukan, dan code path yang melakukan decode/transform sebelum validasi. Pembaruan versi runtime penting, tetapi tidak menggantikan perbaikan desain.

---

## 6.12 Dari LFI ke Eksekusi Kode: Analisis Prasyarat

Sebuah exploit chain LFI menuju RCE dapat terjadi apabila penyerang mampu menempatkan string yang ditafsirkan sebagai kode ke file yang kemudian di-include. Sumber konten dapat berupa upload, file sesi, cache, atau log. Namun setiap jalur memiliki prasyarat dan risiko operasional.

| Prasyarat | Pertanyaan Validasi | Jika Tidak Terpenuhi |
| :--- | :--- | :--- |
| Konten dapat dikendalikan | Apakah marker benar-benar tersimpan? | Chain berhenti |
| Lokasi dapat diketahui | Apakah path file dapat diprediksi/diobservasi? | Inclusion tidak terarah |
| Proses dapat membaca | Apakah permission memungkinkan? | File tidak dapat dimuat |
| Runtime menginterpretasi | Apakah sink mengeksekusi atau hanya menampilkan? | Disclosure saja |
| Request mencapai file | Apakah filter/canonicalization dapat dilewati? | Inclusion gagal |

Pada produksi, penguji sebaiknya tidak melakukan log poisoning atau menanam kode apabila pembuktian dapat dilakukan dengan marker non-eksekusi. Dokumentasikan chain sebagai "potensi dengan prasyarat" bila komponen tertentu belum terbukti. Severity tidak boleh didasarkan pada kata "LFI" semata.

---

## 6.13 Remote File Inclusion (RFI) dan Konteks Modern

RFI terjadi ketika aplikasi memasukkan resource jarak jauh sebagai bagian proses include. Pada PHP, URL include bergantung pada wrapper dan konfigurasi. Dokumentasi PHP mencantumkan `allow_url_include` dengan nilai default nonaktif dan status deprecated pada versi modern [13][15]. Karena itu, RFI tidak boleh diperlakukan sebagai perilaku default semua aplikasi PHP.

Prasyarat RFI mencakup:
1. input mencapai fungsi include;
2. skema remote diterima;
3. konfigurasi wrapper mengizinkan;
4. egress dari server menuju lokasi uji tersedia; dan
5. content remote diproses sesuai konteks yang berbahaya.

Pengujian remote membawa risiko tambahan karena dapat mengirim request keluar organisasi. Gunakan listener atau server laboratorium yang diizinkan, marker unik, dan jangan memakai domain pihak ketiga. Bila egress tidak termasuk scope, cukup lakukan code review atau verifikasi konfigurasi.

---

## 6.14 File Upload adalah Pipeline, Bukan Satu Endpoint

Fitur upload mencakup lebih dari pemeriksaan extension. Siklus hidup berkas dapat melibatkan browser, endpoint penerimaan, temporary directory, antivirus, image processor, document converter, object storage, database metadata, CDN, dan endpoint download.

$$\text{pilih file} \rightarrow \text{multipart request} \rightarrow \text{validasi} \rightarrow \text{rename/simpan sementara} \rightarrow \text{proses/scan} \rightarrow \text{publikasikan} \rightarrow \text{ambil/hapus}$$

Pada setiap tahap, tanyakan:
1. siapa yang boleh mengunggah dan mengambil file;
2. jenis, ukuran, jumlah, dan nama apa yang diizinkan;
3. di mana file disimpan dan dengan permission apa;
4. apakah file dapat diakses sebelum scan selesai;
5. apakah content diproses oleh parser atau converter;
6. apakah file dilayani dengan content type dan disposition yang aman; dan
7. bagaimana retensi, karantina, serta penghapusan dilakukan.

OWASP merekomendasikan pendekatan berlapis: allowlist extension, tidak mempercayai Content-Type, validasi signature/content, filename yang dibuat server, size limit, otorisasi, storage di luar web root atau host terpisah, antivirus/sandbox, dan perlindungan CSRF [7]. Tidak ada satu kontrol yang cukup.

---

## 6.15 Workflow Pengujian File Upload

Pengujian dimulai dengan file benign dan satu perubahan per eksperimen.
1. **Baseline format sah.** Unggah file kecil dengan nama unik dan content sesuai fungsi.
2. **Petakan validasi.** Ubah extension, MIME header, signature, nama, ukuran, dan struktur secara terpisah.
3. **Petakan storage.** Amati rename, URL pengambilan, permission, caching, dan keterdugaan identifier.
4. **Petakan penyajian.** Periksa Content-Type, Content-Disposition, X-Content-Type-Options, CSP, dan apakah file berada pada origin yang sama.
5. **Petakan pemrosesan.** Identifikasi image resize, document preview, unzip, OCR, transcoding, atau antivirus.
6. **Verifikasi otorisasi.** Uji akses lintas akun hanya dengan data laboratorium.
7. **Uji cleanup.** Hapus file uji dan konfirmasi artefak tidak tetap dapat diakses dari cache yang tidak semestinya.

Catat raw multipart request, response penerimaan, lokasi retrieval, dan response saat file diambil. Screenshot UI saja tidak menunjukkan validasi server.

---

## 6.16 Unggahan Executable: Eksekusi Memerlukan Rantai Konfigurasi

Mengunggah file yang mengandung kode tidak otomatis menghasilkan RCE. Server harus menyimpan file pada lokasi yang dapat dijangkau, memetakan extension atau handler ke runtime, memberikan permission yang sesuai, dan mengizinkan request mengeksekusi file tersebut.

| Kondisi | Pertanyaan | Bukti |
| :--- | :--- | :--- |
| Upload diterima | Apakah server menyimpan file? | Response dan retrieval |
| Lokasi terjangkau | Apakah URL atau route file diketahui? | Request GET/HEAD |
| Handler aktif | Apakah file diproses sebagai code? | Marker output laboratorium |
| Privilege proses | Akun OS apa yang menjalankan? | Jangan eskalasi bila tidak perlu |

Pada laboratorium khusus, proof dapat berupa script yang hanya mengembalikan string tetap seperti `LAB-EXECUTION-PROOF`, tanpa menerima command input. Ini cukup membedakan eksekusi dari penyajian source. Jangan mengubahnya menjadi web shell umum.

Bypass extension seperti perbedaan case, extension ganda, extension alternatif, trailing character, atau mismatch konfigurasi dapat dievaluasi pada lab. Namun daftar extension berbahaya bergantung pada web server, framework, OS, dan handler; tidak ada blacklist universal.

---

## 6.17 Unggahan Non-Executable Tetap Dapat Berbahaya

Berkas non-executable dapat menyerang pengguna atau komponen pemrosesan. SVG berbasis XML dapat memuat active content; HTML dapat menjalankan script bila disajikan pada origin aplikasi; office document dapat membawa macro atau external link; gambar atau media dapat mengeksploitasi parser yang rentan; dan file polyglot dapat lolos pemeriksaan dangkal.

Risiko juga muncul dari cara file disajikan. Browser dapat melakukan content sniffing jika header tidak tepat. File yang seharusnya diunduh dapat dirender inline. Filename dapat masuk ke HTML atau header tanpa encoding yang benar. Oleh sebab itu, validasi upload dan keamanan download harus dianalisis sebagai satu sistem.

Kontrol penting meliputi:
1. re-encode gambar melalui pustaka yang dipelihara;
2. gunakan `Content-Disposition: attachment` untuk jenis yang tidak perlu dirender;
3. set `X-Content-Type-Options: nosniff`;
4. layani untrusted content dari origin terpisah tanpa cookie aplikasi;
5. gunakan CSP yang sesuai sebagai defense in depth;
6. scan atau sandbox format berisiko; dan
7. batasi parser, timeout, ukuran, serta resource.

---

## 6.18 Archive Extraction, Filename, dan Resource Exhaustion

Archive menambah lapisan risiko: anggota archive dapat mengandung path traversal, symbolic link, ukuran ekstraksi sangat besar, rasio kompresi ekstrem, atau jumlah file berlebihan. OWASP WSTG secara eksplisit menyoroti archive directory traversal yang dapat menulis di luar lokasi tujuan [8].

Pipeline ekstraksi aman perlu:
1. menghitung estimasi ukuran dan jumlah anggota sebelum ekstraksi;
2. menolak absolute path, traversal segment, dan link yang tidak diizinkan;
3. canonicalize setiap target dan memverifikasi containment;
4. menggunakan direktori sementara yang unik;
5. menerapkan quota, timeout, dan least privilege;
6. memindai hasil sebelum dipublikasikan; dan
7. melakukan cleanup yang dapat diaudit.

Filename asli adalah metadata tidak tepercaya. Simpan nama tampilan terpisah dari storage key, lakukan output encoding saat menampilkan, dan hindari meneruskannya ke shell atau query tanpa API aman.

---

## 6.19 OS Command Injection: Memahami Boundary ke Shell

OS command injection terjadi ketika input tidak tepercaya memengaruhi command yang dieksekusi sistem operasi. WSTG menjelaskan bahwa input yang langsung diteruskan ke command tanpa validasi memadai dapat memungkinkan eksekusi command arbitrer [9].

Contoh pola konseptual rentan:
```c
system("ping -c 1 " + user_input);
```

Jika API memanggil shell, karakter seperti separator, pipe, redirection, command substitution, variable expansion, atau quote memperoleh makna sintaksis. Detail berbeda antara POSIX shell, Windows `cmd.exe`, PowerShell, dan API proses yang tidak memakai shell.

Pembedaan penting:
1. **argument injection:** pengguna mengubah argumen atau opsi program tanpa menambahkan command baru;
2. **command injection:** pengguna memengaruhi struktur command sehingga command lain atau shell construct dieksekusi;
3. **code injection:** input ditafsirkan sebagai kode bahasa/runtime lain.

Ketiganya dapat berdampak serius, tetapi mekanisme dan mitigasinya tidak identik.

---

## 6.20 Operator, Quoting, dan Perbedaan Platform

Payload tidak boleh diperlakukan universal. POSIX shell mengenal operator seperti `;`, `&&`, `||`, `|`, redirection, `$()`, dan backticks. Windows `cmd.exe` mengenal `&`, `&&`, `||`, `|`, serta aturan quoting sendiri. PowerShell memiliki parser dan semantics berbeda.

| Aspek | POSIX Shell | Windows Cmd / PowerShell | Implikasi |
| :--- | :--- | :--- | :--- |
| **Separator** | Semicolon dan newline | Ampersand dan newline (PowerShell juga semicolon) | Uji sesuai interpreter |
| **Chaining** | Double ampersand dan double vertical bar | Tersedia dengan perbedaan versi atau shell | Jangan menyimpulkan dari satu operator |
| **Command substitution** | Dollar-parentheses atau backtick | Konstruksi dan parser berbeda | Filter karakter bukan solusi umum |
| **Delay** | `sleep` | `timeout`, `Start-Sleep` | Baseline waktu harus disesuaikan |

Escaping sering rapuh karena developer harus mengetahui interpreter, encoding, locale, serta cara API membentuk command. OWASP menempatkan menghindari pemanggilan OS command langsung sebagai pertahanan utama; gunakan library atau API yang menyediakan operasi yang dibutuhkan [10]. Jika proses eksternal tidak dapat dihindari, gunakan API yang memisahkan executable dan argument array, allowlist nilai bisnis, terminator opsi bila didukung, least privilege, serta timeout.

---

## 6.21 Direct, Blind, Time-Based, dan Out-of-Band

Direct command injection mengembalikan output command di response. Pada lab, marker non-sensitif seperti `LAB-CMD-PROOF` lebih tepat daripada membaca file credential. Blind command injection tidak menampilkan output; efek dapat diamati melalui waktu, perubahan state laboratorium, atau callback out-of-band yang diizinkan.

Time-based evidence mudah menghasilkan false positive akibat latency, caching, load balancing, atau worker asinkron. Gunakan desain eksperimen:
1. kirim baseline beberapa kali;
2. gunakan kontrol negatif dengan struktur serupa tanpa delay;
3. gunakan delay pendek yang jelas tetapi tidak membebani;
4. lakukan urutan interleaved baseline-uji-baseline-uji;
5. catat median dan variasi, bukan satu pengukuran; dan
6. ulangi dengan durasi berbeda bila aman.

- **Baseline:** median 0,42 s dari 5 request
- **Kontrol:** median 0,45 s dari 5 request
- **Uji delay 3s:** median 3,47 s dari 5 request

Pola tersebut jauh lebih kuat daripada satu response lambat. OOB testing dapat memakai DNS/HTTP callback ke infrastruktur laboratorium, tetapi hanya bila egress dan pencatatan domain tersebut termasuk scope. Jangan memakai layanan publik yang menyebabkan data target dikirim ke pihak ketiga tanpa persetujuan.

---

## 6.22 Evidence, Reproduksibilitas, dan Batas Klaim

Satu paket evidence minimum berisi:
1. endpoint, method, dan parameter;
2. akun/peran serta state awal;
3. timestamp dan environment;
4. raw request dan response yang telah disamarkan;
5. baseline dan kontrol negatif;
6. perubahan tunggal yang diuji;
7. hasil observasi dan confidence;
8. dampak yang terbukti, bukan hanya kemungkinan;
9. limitation dan aktivitas yang sengaja tidak dilakukan; serta
10. rekomendasi yang memutus akar masalah.

| Observasi | Hindari Klaim | Klaim yang Lebih Presisi |
| :--- | :--- | :--- |
| Extension berbahaya diterima | "Server telah dikuasai" | "Validasi tipe upload dapat dilewati; eksekusi belum diuji" |
| Source terungkap via wrapper | "RCE" | "Source disclosure melalui LFI/wrapper terbukti" |
| Delay tunggal | "Blind command injection confirmed" | "Indikasi time-based; perlu reproduksi dan kontrol" |
| Marker command berulang | "Akses root" | "Eksekusi command dalam konteks proses aplikasi terbukti" |

Evidence yang kuat mengurangi kebutuhan eskalasi. Pada laporan komersial, cantumkan langkah reproduksi minimum dan jangan menyertakan secret aktif.

---

## 6.23 Mitigasi sebagai Perubahan Arsitektur, Bukan Filter Tambahan

Perbaikan terbaik menghilangkan interpreter boundary yang tidak perlu.

| Masalah | Perbaikan Primer | Defense in Depth |
| :--- | :--- | :--- |
| Path traversal | Identifier-to-object mapping, canonical containment | Least privilege, storage root khusus, logging |
| File inclusion | Pemetaan template statis, tidak ada dynamic include dari input | Runtime hardening, error suppression yang tepat |
| File upload | Pipeline validasi dan storage terpisah dari execution | Scanning, re-encode, quota, separate origin |
| Command injection | Ganti OS command dengan library/API | Argument array, allowlist, least privilege, timeout |

Filter substring atau blacklist mudah tertinggal karena representasi input, parser, dan konfigurasi berubah. Allowlist juga bukan jaminan bila diterapkan sebelum transformasi yang berbeda. Urutan aman adalah memahami format bisnis yang valid, melakukan parsing/normalisasi terdefinisi, memvalidasi representasi final, menerapkan otorisasi, dan menggunakan sink yang aman.

Operasi juga penting: patch dependency, pantau error dan pola penolakan, lindungi secret, pisahkan network egress, serta uji regression. Perbaikan dianggap selesai ketika test case yang semula gagal menjadi pass dan tidak merusak fungsi sah.

---

## 6.24 Exploit Chain dan Penilaian Risiko Kontekstual

Kerentanan jarang berdiri sendiri. Upload dapat menyediakan konten, traversal dapat memindahkan atau menemukan file, LFI dapat memuat file, dan command injection dapat memperluas dampak. Namun chain harus ditulis sebagai graf prasyarat, bukan narasi spekulatif.

Nilai risiko mempertimbangkan exploitability, privilege awal, interaksi pengguna, boundary, sensitivitas data, blast radius, deteksi, dan recoverability. Gunakan CVSS atau metode organisasi sebagai alat konsistensi, tetapi jelaskan konteks bisnis. Misalnya, disclosure satu file konfigurasi dummy pada lab tidak sama dengan disclosure secret produksi, walaupun mekanismenya mirip.

### 6.24.1 Kerangka Orisinal LINTAS
Bab ini menggunakan kerangka LINTAS untuk menjaga pengujian tetap sistematis:
1. **L**okalisasi input dan scope.
2. **I**dentifikasi transformasi dan interpreter.
3. **N**ormalisasi baseline serta kontrol negatif.
4. **T**es satu perubahan pada satu waktu.
5. **A**rsipkan evidence dan batas dampak.
6. **S**usun mitigasi yang memutus akar masalah.

LINTAS melengkapi JELAJAH dari Bab 5. JELAJAH menghasilkan attack surface register; LINTAS mengubah satu kandidat entry point menjadi eksperimen terkontrol dan finding yang dapat diaudit.

---

## 6.25 Praktikum Terpandu dan Penilaian

### 6.25.1 Tujuan dan Lingkungan
Gunakan salah satu laboratorium lokal yang memang dirancang rentan, misalnya OWASP Juice Shop, WebGoat, DVWA, atau PortSwigger Web Security Academy. Pilih challenge yang sesuai dan patuhi petunjuk platform. Jangan memindahkan teknik ke target publik.

Mahasiswa bekerja dengan browser profile khusus, proxy intersep, akun lab, dan folder evidence. Pengajar menentukan scope, batas request, serta apakah upload executable dan OOB callback diizinkan.

### 6.25.2 Skenario
Sebuah portal praktikum menyediakan:
1. endpoint download materi dengan parameter `filename`;
2. pemilihan bahasa/template;
3. upload avatar dan archive tugas;
4. fungsi diagnostik konektivitas; dan
5. dua peran: mahasiswa serta dosen.

### 6.25.3 Tugas
1. Petakan source-transformasi-sink untuk empat fungsi.
2. Buat baseline dan kontrol negatif pada endpoint download.
3. Buktikan satu pelanggaran path boundary menggunakan marker file lab.
4. Bedakan traversal biasa dari inclusion berdasarkan effect.
5. Petakan pipeline upload dan temukan sedikitnya tiga control point.
6. Uji file benign dengan mismatch extension/MIME tanpa malware.
7. Lakukan eksperimen command injection menggunakan marker non-destruktif atau time delay pendek.
8. Susun dua finding: satu confirmed dan satu observation/inconclusive.
9. Berikan mitigasi primer serta defense in depth.
10. Bersihkan artefak uji sesuai prosedur lab.

### 6.25.4 Deliverable
| Artefak | Isi Minimum |
| :--- | :--- |
| **Test map** | Endpoint, parameter, peran, sink hypothesis |
| **Experiment log** | Baseline, perubahan, hasil, timestamp |
| **Evidence bundle** | Raw request/response terpilih, screenshot pendukung |
| **Finding** | Judul, severity, prasyarat, bukti, dampak, rekomendasi |
| **Reflection** | False positive yang dihindari dan batas pengujian |

### 6.25.5 Rubrik
| Kriteria | Bobot | Indikator Unggul |
| :--- | :--- | :--- |
| Model teknis | 20% | Source, transformasi, sink, dan interpreter tepat |
| Metode eksperimen | 25% | Baseline, kontrol negatif, satu variabel, reproduksi |
| Evidence | 20% | Minimum, jelas, tersamarkan, dapat direproduksi |
| Analisis risiko | 15% | Klaim sesuai bukti dan prasyarat chain eksplisit |
| Mitigasi | 15% | Memutus akar masalah dan mencakup defense in depth |
| Etika/cleanup | 5% | Scope dipatuhi dan artefak ditangani aman |

---

## Rangkuman Bab

1. OWASP Top 10:2025 adalah peta awareness; WSTG dan threat model diperlukan untuk pengujian operasional.
2. Path traversal terjadi ketika input memungkinkan operasi file melampaui direktori yang dimaksud.
3. Encoding relevan karena beberapa lapisan dapat men-decode dan menormalisasi data secara berbeda.
4. File inclusion berbeda dari pembacaan file biasa; RCE memerlukan prasyarat tambahan.
5. `php://filter` adalah fitur stream PHP yang dapat mengubah cara resource dibaca, bukan "payload ajaib" universal.
6. RFI pada PHP modern tidak boleh diasumsikan aktif karena bergantung pada wrapper dan konfigurasi.
7. File upload harus dianalisis sebagai pipeline dari penerimaan hingga penyajian dan penghapusan.
8. File non-executable dapat tetap berbahaya melalui active content, parser, content sniffing, atau archive extraction.
9. Command injection terjadi pada boundary menuju interpreter; menghindari OS command merupakan mitigasi primer.
10. Time-based testing memerlukan baseline berulang dan kontrol negatif.
11. Exploit chain harus dinyatakan sebagai rangkaian prasyarat yang telah atau belum terbukti.
12. Kerangka LINTAS menjaga eksperimen terukur, evidence cukup, dan rekomendasi berorientasi akar masalah.

---

## Uji Pemahaman

### Pilihan Ganda

1. Pernyataan paling tepat mengenai OWASP Top 10 adalah:
   - A. standar sertifikasi yang menjamin aplikasi aman
   - **B. dokumen awareness yang perlu dilengkapi metodologi dan threat model**
   - C. daftar payload wajib untuk pentest
   - D. pengganti code review

2. Tindakan yang paling kuat mencegah path traversal adalah:
   - A. memblokir substring `../`
   - B. mengganti semua titik dengan spasi
   - **C. memetakan identifier ke objek server dan memverifikasi canonical containment**
   - D. menyembunyikan pesan error saja

3. Response 500 setelah input traversal berarti:
   - A. traversal pasti berhasil
   - B. RCE telah terjadi
   - **C. input memicu error; perlu evidence tambahan**
   - D. server aman

4. Perbedaan utama traversal dan file inclusion adalah:
   - A. traversal hanya pada Windows
   - **B. traversal berkaitan dengan boundary path; inclusion berkaitan dengan mekanisme memuat/menafsirkan file**
   - C. inclusion tidak menggunakan file
   - D. keduanya selalu identik

5. Pernyataan tepat mengenai LFI menuju RCE adalah:
   - A. setiap LFI otomatis RCE
   - **B. RCE memerlukan konten terkontrol, path, permission, dan interpretasi runtime yang sesuai**
   - C. LFI hanya berdampak availability
   - D. extension `.php` selalu mencegah LFI

6. Fungsi konseptual `php://filter` adalah:
   - A. firewall PHP
   - **B. meta-wrapper yang menerapkan filter pada stream**
   - C. scanner malware
   - D. pengganti otorisasi

7. Mengapa extension allowlist saja belum cukup untuk upload?
   - A. browser selalu menghapus extension
   - **B. risiko juga berasal dari content, storage, parser, penyajian, dan otorisasi**
   - C. extension tidak pernah dikirim
   - D. semua file memiliki extension sama

8. Bukti paling tepat bahwa upload executable benar-benar dieksekusi adalah:
   - A. file diterima server
   - B. URL file menghasilkan 200
   - **C. marker output tetap dihasilkan oleh script lab tanpa menerima command**
   - D. extension terlihat pada UI

9. Pertahanan utama terhadap OS command injection adalah:
   - A. memblokir titik koma saja
   - B. menyembunyikan output
   - **C. menggunakan library/API dan menghindari shell bila memungkinkan**
   - D. menambah delay

10. Satu response yang terlambat lima detik adalah:
    - A. bukti final blind command injection
    - **B. indikasi lemah yang memerlukan baseline, kontrol, dan reproduksi**
    - C. bukti akses root
    - D. selalu gangguan jaringan

11. Zip Slip merupakan contoh:
    - A. session fixation
    - **B. traversal pada nama anggota archive saat ekstraksi**
    - C. SQL injection
    - D. clickjacking

12. Langkah "A" pada LINTAS adalah:
    - A. aktifkan scanner
    - **B. arsipkan evidence dan batas dampak**
    - C. ambil semua data
    - D. abaikan baseline

### Kunci Jawaban Pilihan Ganda
1. B | 2. C | 3. C | 4. B | 5. B | 6. B | 7. B | 8. C | 9. C | 10. B | 11. B | 12. B

---

## Pertanyaan Esai dan Diskusi

1. Jelaskan bagaimana normalization gap dapat terjadi antara reverse proxy dan framework aplikasi.
2. Rancang eksperimen untuk membedakan file-not-found biasa dari path traversal yang berhasil.
3. Buat tabel prasyarat LFI menuju RCE dan jelaskan cara membatasi klaim bila satu prasyarat belum terbukti.
4. Analisis mengapa file SVG yang valid tetap dapat menimbulkan risiko ketika disajikan pada origin aplikasi.
5. Rancang pipeline upload aman untuk tugas kuliah berbentuk PDF dan ZIP.
6. Jelaskan perbedaan argument injection dan OS command injection beserta mitigasinya.
7. Susun desain eksperimen time-based yang mengurangi false positive akibat latency.
8. Gunakan LINTAS untuk menulis outline finding dari satu challenge laboratorium yang telah diselesaikan.

---

## Glosarium

- **Allowlist:** Daftar nilai atau pola yang secara eksplisit diizinkan.
- **Argument injection:** Manipulasi argumen atau opsi program tanpa harus menambahkan command baru.
- **Canonicalization:** Proses menghasilkan representasi path yang dinormalisasi menurut aturan filesystem atau API.
- **Command injection:** Penyisipan elemen sehingga struktur command atau shell construct dapat dipengaruhi input tidak tepercaya.
- **Containment check:** Verifikasi bahwa path final tetap berada di bawah root yang diizinkan.
- **Control negative:** Input pembanding yang diperkirakan tidak menghasilkan efek target.
- **Directory/path traversal:** Manipulasi path untuk mengakses objek di luar boundary direktori yang dimaksud.
- **File inclusion:** Pemuatan file secara dinamis oleh interpreter atau runtime aplikasi.
- **LFI:** *Local file inclusion*; inclusion terhadap resource lokal.
- **Normalization gap:** Perbedaan representasi atau hasil normalisasi antara dua lapisan pemrosesan.
- **Out-of-band (OOB):** Observasi efek melalui kanal berbeda dari response utama, misalnya callback DNS laboratorium.
- **Polyglot file:** Berkas yang dapat ditafsirkan sebagai lebih dari satu format.
- **RFI:** *Remote file inclusion*; inclusion terhadap resource jarak jauh.
- **Security-sensitive sink:** Fungsi atau komponen yang menafsirkan data dalam operasi sensitif, seperti filesystem atau shell.
- **Stream wrapper:** Abstraksi PHP untuk mengakses stream/resource melalui skema tertentu.
- **Time-based testing:** Pembuktian efek dengan membandingkan pola waktu terhadap baseline dan kontrol.
- **Zip Slip:** Traversal melalui nama/path anggota archive saat ekstraksi.

---

## Daftar Pustaka

1. OffSec. "OSCP+ Body of Knowledge." 2026. `https://help.offsec.com/hc/en-us/articles/38543335188756-OSCP-Body-of-knowledge`
2. OffSec. "PEN-200 Learning Plan 24 Week." 2025. `https://help.offsec.com/hc/en-us/articles/15545672357780-OffSec-PEN-200-Learning-Plan-24-Week`
3. OWASP Foundation. "OWASP Top 10:2025." 2025. `https://owasp.org/Top10/2025/`
4. OWASP Foundation. "Testing Directory Traversal File Include." WSTG Latest. `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web Application Security Testing/05-Authorization Testing/01-Testing Directory Traversal File Include`
5. OWASP Foundation. "Testing for File Inclusion." WSTG Latest. `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web Application Security Testing/07-Input Validation Testing/11.1-Testing for File Inclusion`
6. OWASP Foundation. "Testing for Remote File Inclusion." WSTG. `https://owasp.org/www-project-web-security-testing-guide/v42/4-Web Application Security Testing/07-Input Validation Testing/11.2-Testing for File Inclusion`
7. OWASP Foundation. "File Upload Cheat Sheet." `https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html`
8. OWASP Foundation. "Test Upload of Malicious Files." WSTG. `https://owasp.org/www-project-web-security-testing-guide/v42/4-Web Application Security Testing/10-Business Logic Testing/09-Test Upload of Malicious Files`
9. OWASP Foundation. "Testing for Command Injection." WSTG Latest. `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web Application Security Testing/07-Input Validation Testing/12-Testing for Command Injection`
10. OWASP Foundation. "OS Command Injection Defense Cheat Sheet." `https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html`
11. OWASP Foundation. "Input Validation Cheat Sheet." `https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html`
12. PHP Documentation. "php:// - Accessing Various I/O Streams." `https://www.php.net/manual/en/wrappers.php.php`
13. PHP Documentation. "Filesystem and Streams Configuration Options." `https://www.php.net/manual/en/filesystem.configuration.php`
14. PHP Documentation. "Supported Protocols and Wrappers." `https://www.php.net/manual/en/wrappers.php`
15. PHP Documentation. "List of php.ini Directives." `https://www.php.net/manual/en/ini.list.php`
16. PHP Documentation. "include." `https://www.php.net/manual/en/function.include.php`
17. MITRE. "CWE-22: Improper Limitation of a Pathname to a Restricted Directory." `https://cwe.mitre.org/data/definitions/22.html`
18. MITRE. "CWE-73: External Control of File Name or Path." `https://cwe.mitre.org/data/definitions/73.html`
19. MITRE. "CWE-78: Improper Neutralization of Special Elements used in an OS Command." `https://cwe.mitre.org/data/definitions/78.html`
20. MITRE. "CWE-98: Improper Control of Filename for Include/Require Statement." `https://cwe.mitre.org/data/definitions/98.html`
21. MITRE. "CWE-434: Unrestricted Upload of File with Dangerous Type." `https://cwe.mitre.org/data/definitions/434.html`
22. IETF. "RFC 3986: Uniform Resource Identifier (URI): Generic Syntax." 2005. `https://www.rfc-editor.org/rfc/rfc3986`
23. PortSwigger. "Path Traversal." Web Security Academy. `https://portswigger.net/web-security/file-path-traversal`
24. PortSwigger. "File Upload Vulnerabilities." Web Security Academy. `https://portswigger.net/web-security/file-upload`
25. PortSwigger. "OS Command Injection." Web Security Academy. `https://portswigger.net/web-security/os-command-injection`
26. NIST. "SP 800-115: Technical Guide to Information Security Testing and Assessment." 2008. `https://csrc.nist.gov/pubs/sp/800/115/final`
27. FIRST. "Common Vulnerability Scoring System v4.0." 2023. `https://www.first.org/cvss/v4-0/`
28. OWASP Foundation. "Web Security Testing Guide." `https://owasp.org/www-project-web-security-testing-guide/`
29. OWASP Foundation. "Test Upload of Unexpected File Types." WSTG. `https://owasp.org/www-project-web-security-testing-guide/v42/4-Web Application Security Testing/10-Business Logic Testing/08-Test Upload of Unexpected File Types`
30. OWASP Foundation. "Path Traversal." `https://owasp.org/www-community/attacks/Path_Traversal`
31. OWASP Foundation. "OWASP Juice Shop." `https://owasp.org/www-project-juice-shop/`
32. OWASP Foundation. "OWASP WebGoat." `https://owasp.org/www-project-webgoat/`
33. DVWA Project. "Damn Vulnerable Web Application." `https://github.com/digininja/DVWA`
34. PortSwigger. "Web Security Academy Learning Paths." `https://portswigger.net/web-security/learning-paths`
35. OWASP Foundation. "Secure Code Review Cheat Sheet." `https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html`