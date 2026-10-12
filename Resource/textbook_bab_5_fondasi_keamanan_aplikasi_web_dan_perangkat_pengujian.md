# TEXTBOOK CYBERSECURITY. PROGRAM SARJANA

## BAB 5

# Fondasi Keamanan Aplikasi Web dan Perangkat Pengujian

**Dari semantics HTTP dan browser security model menuju attack surface register yang dapat diaudit**

**Hatma Suryotrisongko**

*Materi latihan hanya untuk lingkungan milik sendiri atau yang memiliki izin tertulis.*

## Isi Bab

Bab ini bergerak dari arsitektur web dan semantics HTTP menuju browser security model, proxy intersep, fingerprinting, content/API discovery, evidence, kerangka JELAJAH, dan handoff hipotesis.

* 5.1 Kedudukan Pengujian Aplikasi Web
* 5.2 Kewenangan, Scope, dan Data Sensitif
* 5.3 Arsitektur Aplikasi Web dan Trust Boundary
* 5.4 URL, URI, dan Titik Masuk
* 5.5 Semantics HTTP: Request, Response, dan Method
* 5.6 Header, Representasi, dan Content Negotiation
* 5.7 Browser Security Model: SOP, CORS, dan CSP
* 5.8 Cookie, Session, dan Token
* 5.9 Server-Rendered, SPA, dan API-Driven
* 5.10 Proxy Intersep dan TLS
* 5.11 Workflow Burp Suite dan OWASP ZAP
* 5.12 Merekam Baseline dengan `curl` dan DevTools
* 5.13 Pemetaan Alur Pengguna dan Endpoint Inventory
* 5.14 Fingerprinting sebagai Hipotesis Teknologi
* 5.15 Content Discovery yang Terkendali
* 5.16 Enumerasi REST dan OpenAPI
* 5.17 GraphQL dan Introspection
* 5.18 WebSocket dan Traffic Asinkron
* 5.19 Dari Observasi ke Hipotesis Pengujian
* 5.20 Evidence, Provenance, dan Sanitasi
* 5.21 Passive Scan, Active Scan, dan DevSecOps
* 5.22 Perspektif Defender
* 5.23 Kerangka JELAJAH
* 5.24 Studi Kasus Terbimbing
* 5.25 Praktik Terbimbing

> *Catatan akademik. Contoh pada bab ini menggunakan aplikasi fiktif atau laboratorium vulnerable-by-design. Pembaca dilarang mempraktikkan teknik pada sistem publik tanpa otorisasi eksplisit.*

## Pendahuluan

Aplikasi web telah berkembang dari kumpulan halaman statis menjadi sistem terdistribusi yang menghubungkan browser, layanan API, penyedia identitas, penyimpanan objek, antrean pesan, layanan pihak ketiga, dan berbagai komponen cloud. Perubahan tersebut memperbesar manfaat bisnis sekaligus memperluas permukaan serangan. Sebuah tombol pada layar dapat memicu beberapa permintaan API, memperbarui state di browser, memanggil layanan eksternal, dan mengeksekusi keputusan otorisasi pada komponen yang berbeda.

Akibatnya, pengujian keamanan aplikasi web tidak boleh dimulai dengan daftar payload. Penguji terlebih dahulu perlu memahami cara aplikasi berkomunikasi, bagaimana browser menerapkan batas keamanan, di mana identitas dan state disimpan, serta bukti apa yang dapat mendukung suatu klaim. Alat seperti Burp Suite, OWASP ZAP, `curl`, WhatWeb, dan `ffuf` membantu observasi dan eksperimen, tetapi alat tidak menggantikan model mental tentang aplikasi.

Bab ini mengadaptasi kedalaman kompetensi PEN-200 pada modul *Introduction to Web Application Attacks* untuk tingkat sarjana. Fokusnya adalah metodologi assessment, pemetaan attack surface, penggunaan proxy intersep, fingerprinting, content discovery, serta enumerasi API secara aman. Eksploitasi kerentanan spesifik dibahas pada bab-bab berikutnya.

> **Batas etika dan operasional.** Seluruh aktivitas interaktif pada bab ini hanya boleh dilakukan terhadap aplikasi milik sendiri, laboratorium vulnerable-by-design, atau target yang tercantum dalam otorisasi tertulis. Content discovery, pengubahan request, fuzzing, dan active scan dapat menambah beban, mengubah state, memicu alarm, atau mengakses data yang tidak dimaksudkan untuk penguji.

### Capaian Pembelajaran Bab

Setelah mempelajari bab ini, mahasiswa mampu:

1. menjelaskan alur komunikasi browser, jaringan, reverse proxy, server aplikasi, dan layanan data;
2. membaca struktur URL, request, response, header, body, status code, cookie, serta token;
3. membedakan semantics HTTP dari kebiasaan implementasi framework;
4. menjelaskan same-origin policy, CORS, CSP, serta atribut cookie tanpa mencampuradukkan fungsinya;
5. membedakan aplikasi server-rendered, SPA, REST, GraphQL, dan komunikasi WebSocket;
6. menetapkan scope proxy intersep dan mengelola CA pengujian secara aman;
7. menggunakan Proxy/HTTP history, Repeater, Logger, dan fasilitas pengulangan request secara terukur;
8. melakukan fingerprinting serta content discovery dengan baseline dan rate limit;
9. menyusun endpoint inventory dan attack surface register yang memiliki provenance;
10. membedakan passive observation, manual active testing, fuzzing, dan active scanning; dan
11. merancang handoff menuju pengujian kerentanan spesifik dengan kerangka JELAJAH.

### Peta Konsep

Alur belajar bab ini dapat diringkas sebagai berikut:

$$
\text{kewenangan} \rightarrow \text{arsitektur aplikasi} \rightarrow \text{semantics HTTP} \rightarrow \text{browser security model} \rightarrow \text{state dan identitas} \rightarrow \text{proxy intersep} \rightarrow \text{pemetaan alur} \rightarrow \text{fingerprinting} \rightarrow \text{content/API discovery} \rightarrow \text{evidence} \rightarrow \text{hipotesis pengujian}
$$

Tiga prinsip menjadi pengendali seluruh kegiatan:

1. antarmuka pengguna bukan keseluruhan attack surface;
2. respons aplikasi adalah evidence yang harus dibaca dalam konteks request, state, peran, dan waktu; dan
3. keberhasilan alat menemukan suatu path atau teknologi belum membuktikan adanya kerentanan.

## 5.1 Kedudukan Pengujian Aplikasi Web

OWASP Web Security Testing Guide (WSTG) memandang pengujian web sebagai rangkaian kategori yang mencakup information gathering, konfigurasi, identitas, autentikasi, otorisasi, session management, input validation, error handling, kriptografi, business logic, client-side, dan API. Struktur ini menegaskan bahwa keamanan web bukan sinonim dari mencari injection.

PEN-200 menempatkan pengantar serangan web setelah information gathering dan vulnerability scanning. Urutan tersebut penting: mahasiswa membawa scope, kandidat aset, service register, dan temuan scanner ke konteks aplikasi. Pada tahap ini, target tidak lagi hanya dipandang sebagai `443/tcp open`, melainkan sebagai kumpulan origin, route, endpoint, parameter, state, peran pengguna, dan trust boundary.

| Kegiatan | Pertanyaan Utama | Keluaran yang Wajar |
| :--- | :--- | :--- |
| **Reconnaissance web** | Komponen dan jalur apa yang terlihat? | Technology hypotheses, origin, route, endpoint |
| **Web assessment** | Kontrol apa yang perlu diuji dan dalam konteks apa? | Test map, evidence, limitation, finding |
| **Active scanning** | Pola kelemahan apa yang terdeteksi secara otomatis? | Alert yang masih memerlukan triase |
| **Penetration testing** | Apakah kelemahan dapat dibuktikan dalam scope? | Bukti dampak dan jalur serangan terotorisasi |

OWASP Top 10:2025 berguna sebagai dokumen awareness mengenai risiko aplikasi web yang paling penting, sedangkan WSTG menyediakan skenario pengujian yang lebih operasional. Keduanya bukan checklist yang menjamin kelengkapan. Aplikasi tetap memerlukan threat model dan pengujian business logic yang sesuai dengan fungsi bisnisnya.

## 5.2 Kewenangan, Scope, dan Data Sensitif

Scope web perlu lebih rinci daripada daftar domain. Satu domain dapat mengarahkan pengguna ke penyedia identitas, CDN, payment gateway, analytics, atau API pihak ketiga yang tidak termasuk target. Sebaliknya, satu aplikasi dapat memakai beberapa subdomain dan origin yang semuanya perlu dinilai.

### 5.2.1 Komponen Web Test Plan

| Komponen | Pertanyaan Pengendali | Contoh Keputusan Laboratorium |
| :--- | :--- | :--- |
| **Target** | Origin, host, port, dan path apa yang diizinkan? | Hanya `https://<LAB-HOST>/app/` |
| **Akun** | Peran dan data uji apa yang tersedia? | Guest, student, dan admin-lab |
| **Aktivitas** | Apakah fuzzing, upload, atau active scan diizinkan? | Hanya content discovery ber-rate rendah |
| **State** | Aksi apa yang dapat mengubah atau menghapus data? | Checkout dan delete dilarang |
| **Pihak ketiga** | Origin mana yang harus dikecualikan? | Analytics dan payment sandbox eksternal |
| **Waktu** | Kapan pengujian boleh berjalan? | Jendela praktikum 90 menit |
| **Stop condition** | Indikator apa yang menghentikan pengujian? | Error rate, latency, atau instruksi pengajar |
| **Evidence** | Artefak apa yang disimpan dan disamarkan? | Request/response terpilih tanpa secret |

Proxy intersep dapat merekam password, session cookie, bearer token, data pribadi, dan respons API. Oleh sebab itu, proyek Burp/ZAP dan ekspor HAR diperlakukan sebagai data sensitif. Gunakan akun laboratorium, simpan artefak pada lokasi terkendali, samarkan secret, dan hapus CA pengujian dari trust store setelah praktikum apabila tidak lagi dibutuhkan.

### 5.2.2 Scope sebagai Kontrol pada Alat

Scope bukan hanya teks dalam *Rules of Engagement*. Terapkan include dan exclude rules pada proxy agar traffic pihak ketiga tidak ikut dicatat atau diuji. Matikan interception ketika tidak diperlukan, gunakan browser profile khusus, dan pastikan extension pribadi tidak mengirim data laboratorium ke layanan eksternal.

**Prinsip minimisasi.** Hanya rekam pesan yang relevan terhadap tujuan pengujian. Bukti yang baik bukan proyek proxy sebesar mungkin, melainkan artefak minimum yang cukup untuk mereproduksi observasi tanpa membocorkan secret.

## 5.3 Arsitektur Aplikasi Web dan Trust Boundary

Model paling sederhana terdiri atas browser dan web server. Dalam produksi, request dapat melewati DNS, CDN, web application firewall, load balancer, reverse proxy, service mesh, API gateway, dan server aplikasi sebelum mencapai database atau layanan lain. Setiap transisi dapat mengubah header, terminasi TLS, identitas sumber, cache, atau format pesan.

Penguji perlu membedakan komponen yang dapat diamati dari komponen yang baru diduga. Header `Server`, nama cookie, pola asset JavaScript, atau halaman error dapat memberi petunjuk teknologi, tetapi petunjuk tersebut dapat berasal dari reverse proxy, disamarkan, atau tertinggal setelah migrasi.

### 5.3.1 Model Alur Data Minimum

Untuk setiap fungsi penting, catat:

1. aktor atau peran pengguna;
2. origin dan halaman tempat aksi dimulai;
3. request yang dikirim;
4. komponen yang dipercaya memproses request;
5. perubahan state yang diharapkan;
6. respons dan data yang kembali; dan
7. boundary menuju layanan atau pihak ketiga.

Sebagai contoh, fungsi unggah tugas dapat melibatkan browser, endpoint metadata, presigned URL ke object storage, pemindai antivirus, worker asinkron, dan endpoint status. Menguji hanya form unggah akan melewatkan sebagian besar alurnya.

### 5.3.2 Origin, Site, dan Host Bukan Istilah yang Sama

Origin didefinisikan oleh kombinasi scheme, host, dan port. `https://app.example.test` dan `https://api.example.test` memiliki host berbeda dan karena itu origin berbeda. Istilah site pada kebijakan browser tertentu menggunakan registrable domain dan scheme, sehingga same-site belum tentu same-origin. Perbedaan ini memengaruhi cookie, CORS, iframe, dan komunikasi JavaScript.

## 5.4 URL, URI, dan Titik Masuk

URL dapat diurai menjadi scheme, authority, host, port, path, query, dan fragment. Setiap bagian memiliki implikasi pengujian.

```text
https://app.lab.test:8443/orders/314?view=detail#payment
 [scheme]     [host]  [port]  [path]     [query]  [fragment]
```

Fragment setelah `#` umumnya diproses di sisi klien dan tidak dikirim sebagai bagian request HTTP ke server. Namun SPA dapat memakai fragment untuk routing atau menyimpan state sensitif. Query string dikirim ke server dan sering muncul pada log, history, analytics, referer, atau screenshot; karena itu secret dan session ID tidak semestinya ditempatkan di URL.

Entry point bukan hanya field formulir. Daftarnya dapat mencakup:

1. path segment dan query parameter;
2. form field, JSON property, XML element, dan multipart field;
3. header seperti `Host`, `Origin`, `Referer`, atau header aplikasi;
4. cookie, bearer token, dan identifier objek;
5. file upload, filename, content type, dan metadata;
6. pesan WebSocket atau event stream;
7. redirect target dan callback URL; serta
8. data yang berasal dari webhook atau layanan pihak ketiga.

## 5.5 Semantics HTTP: Request, Response, dan Method

RFC 9110 mendefinisikan HTTP sebagai protokol lapisan aplikasi yang stateless. HTTP semantics tetap berlaku pada $\$HTTP/1.1\$$, $\$HTTP/2\$$, dan $\$HTTP/3\$$ meskipun framing dan transportnya berbeda. Penguji harus membaca makna method dan representasi, bukan hanya format baris teks $\$HTTP/1.1\$$.

### 5.5.1 Struktur Request dan Response

Contoh request laboratorium:
```http
POST /api/v1/profile HTTP/1.1
Host: <LAB-HOST>
Content-Type: application/json
Cookie: session=<REDACTED>
Origin: https://<LAB-HOST>

{"display_name":"Mahasiswa Uji"}
```

Contoh response:
```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Set-Cookie: session=<REDACTED>; Secure; HttpOnly; SameSite=Lax

{"status":"updated"}
```

Satu transaksi perlu dibaca sebagai pasangan. Request menjelaskan input, state, dan konteks; response menjelaskan keputusan server yang teramati. Menyimpan hanya screenshot halaman sering tidak cukup untuk mereproduksi temuan.

### 5.5.2 Safe, Idempotent, dan Implementasi Nyata

| Method | Semantics Umum | Safe | Idempotent | Catatan Pengujian |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | Mengambil representasi | Ya | Ya | Implementasi buruk tetap dapat mengubah state |
| **HEAD** | GET tanpa response content | Ya | Ya | Header dapat berbeda karena middleware |
| **POST** | Memproses content sesuai resource | Tidak | Tidak | Bukan selalu "membuat data baru" |
| **PUT** | Membuat atau mengganti state target | Tidak | Ya | Bukan sinonim universal untuk partial update |
| **PATCH** | Menerapkan modifikasi parsial | Tidak | Tidak selalu | Semantics bergantung patch document |
| **DELETE** | Meminta penghapusan asosiasi resource | Tidak | Ya | Response berulang dapat berbeda |
| **OPTIONS** | Menjelaskan opsi komunikasi | Ya | Ya | `Allow` atau CORS preflight tidak selalu lengkap |

Safe berarti semantics method dimaksudkan hanya untuk pembacaan, bukan jaminan bahwa tidak ada logging atau efek samping operasional. Idempotent berarti beberapa request identik memiliki intended effect yang sama dengan satu request, bukan berarti seluruh response harus identik. Penguji tidak boleh menganggap GET selalu aman untuk diulang atau PUT selalu berfungsi sebagai update; perilaku server harus diobservasi.

### 5.5.3 Status Code Bukan Kesimpulan Akhir

Kelompok 2xx, 3xx, 4xx, dan 5xx memberi kategori umum, tetapi arti bisnis tetap berada pada body, header, dan perubahan state. `200 OK` dapat membawa pesan gagal dalam JSON. `404 Not Found` dapat digunakan untuk menyamarkan resource yang sebenarnya ada. `302 Found` dapat menandai redirect autentikasi. `500 Internal Server Error` tidak otomatis membuktikan kerentanan, tetapi dapat menjadi bukti error handling yang perlu dianalisis.

## 5.6 Header, Representasi, dan Content Negotiation

Header memuat metadata tentang routing, representasi, cache, autentikasi, origin, dan kebijakan browser. Beberapa header penting untuk pemetaan awal adalah `Host`, `Content-Type`, `Accept`, `Authorization`, `Cookie`, `Set-Cookie`, `Location`, `Cache-Control`, `Origin`, `Access-Control-*`, dan `Content-Security-Policy`.

Content-Type menjelaskan media type representasi. Server yang mengharapkan JSON dapat memproses request secara berbeda ketika menerima form URL-encoded atau multipart. Accept menyatakan representasi yang diinginkan klien, tetapi server dapat mengabaikannya. Penguji mencatat variasi tersebut sebagai behavior, bukan langsung sebagai weakness.

Header keamanan merupakan sinyal kontrol, bukan bukti keamanan menyeluruh. CSP yang ketat dapat mengurangi dampak beberapa serangan client-side, tetapi tidak memperbaiki broken access control. `X-Content-Type-Options: nosniff` membantu mengendalikan MIME sniffing, tetapi tidak mengamankan file upload yang diproses secara salah.

## 5.7 Browser Security Model: SOP, CORS, dan CSP

Browser bukan klien HTTP pasif. Browser menerapkan security model yang memengaruhi apakah script dapat membaca response, apakah cookie dikirim, dan resource apa yang boleh dieksekusi.

### 5.7.1 Same-Origin Policy

Same-origin policy (SOP) membatasi interaksi script dari satu origin terhadap resource origin lain. Pembatasan ini terutama melindungi kerahasiaan data dari pembacaan lintas origin. SOP tidak berarti browser tidak dapat mengirim request lintas origin; form, image, navigation, dan beberapa mekanisme lain tetap dapat mengirim request dalam kondisi tertentu.

### 5.7.2 Cross-Origin Resource Sharing

CORS adalah mekanisme berbasis header yang memungkinkan server memberi tahu browser origin mana yang boleh membaca response lintas origin. Untuk request tertentu, browser mengirim preflight OPTIONS. CORS diterapkan oleh browser; klien non-browser seperti `curl` tidak "diblokir CORS". Oleh karena itu, keberhasilan `curl` membaca API bukan bukti misconfiguration CORS.

Kesalahan analisis umum adalah menyimpulkan `Access-Control-Allow-Origin: *` selalu kritis. Dampak bergantung pada sensitivitas response, credential mode, endpoint, dan apakah origin attacker dapat membaca data yang seharusnya terlindungi.

### 5.7.3 Content Security Policy

CSP mengirim instruksi kepada browser mengenai sumber script, style, frame, dan resource lain yang boleh digunakan. CSP merupakan defense in depth. Kehadiran CSP tidak meniadakan XSS; ketiadaannya juga tidak membuktikan XSS. Pengujian perlu menilai policy aktual, directive, nonce/hash, dan sink client-side yang relevan.

| Kontrol | Pertanyaan yang Dijawab | Bukan Fungsi Utamanya |
| :--- | :--- | :--- |
| **SOP** | Apakah script origin A boleh membaca origin B? | Autentikasi pengguna |
| **CORS** | Origin mana yang diizinkan server untuk membaca response? | Firewall terhadap klien non-browser |
| **CSP** | Resource dan eksekusi apa yang dibatasi browser? | Validasi input server-side |
| **SameSite cookie** | Kapan cookie dikirim pada konteks cross-site? | Pengganti token CSRF untuk semua kasus |

## 5.8 Cookie, Session, dan Token

Karena HTTP stateless, aplikasi menambahkan mekanisme state. Pola umum adalah session identifier dalam cookie, bearer token pada header `Authorization`, atau kombinasi access token dan refresh token. Penguji tidak boleh menganggap nama token menjelaskan format maupun jaminan keamanannya.

### 5.8.1 Atribut Cookie

`Secure` membatasi pengiriman cookie ke secure channel. `HttpOnly` mencegah akses cookie melalui JavaScript DOM, tetapi tidak menghentikan script berbahaya mengirim request dalam konteks pengguna. `SameSite` memengaruhi pengiriman cookie pada konteks cross-site. `Domain` dan `Path` menentukan jangkauan cookie, sedangkan `Max-Age` atau `Expires` memengaruhi persistensi.

Checklist observasi session:
1. catat kapan session dibuat, diperbarui, dan berakhir;
2. bandingkan identifier sebelum dan sesudah login;
3. periksa atribut setiap `Set-Cookie`;
4. amati logout dan invalidasi server-side;
5. bedakan cookie autentikasi dari preference/analytics; dan
6. samarkan seluruh secret pada catatan dan screenshot.

### 5.8.2 Authentication Bukan Authorization

Authentication menjawab identitas yang diklaim dan diverifikasi. Authorization menjawab tindakan atau data yang boleh diakses identitas tersebut. Session membawakan konteks antar-request. Ketiga konsep ini saling terkait tetapi tidak dapat dipertukarkan.

Attack surface register perlu mencatat peran minimal untuk setiap endpoint. Endpoint yang hanya muncul saat login sebagai admin belum tentu tidak dapat dipanggil akun biasa; antarmuka yang menyembunyikan tombol bukan bukti kontrol server-side.

## 5.9 Server-Rendered, SPA, dan API-Driven

Aplikasi server-rendered menghasilkan sebagian besar HTML di server. Navigasi sering memuat ulang dokumen penuh. SPA mengunduh shell JavaScript lalu mengambil data dan melakukan navigasi melalui API. Banyak sistem modern bersifat hibrida: server-side rendering pada request awal, hydration di browser, kemudian request API.

Perbedaan arsitektur mengubah cara pemetaan:

| Aspek | Server-Rendered | SPA / API-Driven |
| :--- | :--- | :--- |
| **Navigasi** | Link dan form terlihat pada HTML | Route dan request dibentuk JavaScript |
| **Data** | HTML atau form response | JSON/GraphQL dan state client |
| **Endpoint** | Sering mengikuti halaman | Dapat tersebar pada bundle dan runtime traffic |
| **Risiko terlewat** | Hidden field, alternate method | Background API, role route, WebSocket |
| **Bukti utama** | Form, redirect, HTML | HTTP history, JS asset, API response |

UI adalah satu klien dari API, bukan enforcement boundary. Tombol disabled, field hidden, dan validasi JavaScript membantu pengalaman pengguna tetapi tidak boleh menjadi satu-satunya kontrol keamanan.

## 5.10 Proxy Intersep dan TLS

Burp Proxy dan OWASP ZAP beroperasi di antara klien dan target untuk merekam, menahan, serta memodifikasi pesan. Agar HTTPS dapat diperiksa, browser pengujian mempercayai CA lokal yang digunakan proxy untuk membuat sertifikat per-host. Konfigurasi ini memberikan proxy kemampuan membaca plaintext aplikasi pada profile tersebut.

### 5.10.1 Praktik Aman CA Pengujian

1. gunakan browser profile terpisah yang hanya dipakai untuk laboratorium;
2. jangan memasang CA pengujian pada trust store organisasi tanpa kebutuhan dan persetujuan;
3. jangan membagikan private key CA proyek proxy;
4. verifikasi listener hanya terikat pada interface yang diperlukan;
5. hapus CA dan profile setelah engagement; dan
6. jangan login ke layanan pribadi ketika proxy aktif.

Certificate pinning pada aplikasi tertentu dapat mencegah proxy bekerja. Melewati pinning merupakan aktivitas terpisah yang memerlukan izin eksplisit dan tidak dibahas pada bab ini.

### 5.10.2 Proxy sebagai Sensor dan Aktuator

Ketika interception nonaktif, proxy tetap dapat berfungsi sebagai sensor melalui HTTP history atau logger. Ketika request ditahan dan diubah, proxy menjadi aktuator yang dapat memengaruhi aplikasi. Peralihan ini harus disengaja dan dicatat.

## 5.11 Workflow Burp Suite dan OWASP ZAP

Nama fitur dapat berubah antarversi, tetapi fungsi dasarnya relatif stabil.

### 5.11.1 Proxy History atau HTTP History

History digunakan untuk menjawab pertanyaan berikut:
1. origin dan endpoint apa yang benar-benar dihubungi browser?
2. method, content type, dan parameter apa yang digunakan?
3. cookie atau token apa yang menyertai request?
4. redirect atau call chain apa yang terjadi?
5. respons mana yang berbeda antarperan atau antarstate?

Filter history berdasarkan scope, MIME type, status code, atau kata kunci. Jangan menghapus traffic yang dianggap "tidak menarik" sebelum memahami fungsinya; background request sering mengungkap API, telemetry, feature flag, atau polling.

### 5.11.2 Repeater dan Manual Request Editor

Repeater memungkinkan pesan HTTP atau WebSocket dikirim ulang setelah dimodifikasi. Gunakan pendekatan satu variabel pada satu waktu:
1. kirim baseline request tanpa perubahan;
2. ubah satu parameter, header, identifier, atau state;
3. kirim ulang dengan laju rendah;
4. bandingkan status, header, body, panjang, dan perubahan state;
5. simpan request/response yang paling representatif; dan
6. kembalikan state laboratorium bila diperlukan.

Perbedaan response tidak otomatis berarti kerentanan. Token yang kedaluwarsa, data dinamis, load balancing, cache, atau request ID dapat menghasilkan variasi yang tidak relevan.

### 5.11.3 Intruder atau Fuzzer

Intruder dan fuzzer mengotomasi variasi payload pada posisi tertentu. Otomasi meningkatkan volume, sehingga harus memiliki tujuan, daftar input, rate, dan stop condition. Pada bab fondasi, penggunaannya dibatasi untuk enumerasi laboratorium yang aman—bukan brute force credential atau payload eksploitatif.

### 5.11.4 Burp dan ZAP Bukan Pilihan Mutlak

Burp Suite menyediakan workflow user-driven yang kuat; fitur dan batasnya bergantung edisi. ZAP bersifat open source dan mendukung proxy, passive scan, automation framework, serta active scan. Pilih alat berdasarkan kebutuhan pengajaran, lisensi, integrasi, dan kemampuan tim. Hasil dari keduanya tetap memerlukan validasi manusia.

## 5.12 Merekam Baseline dengan `curl` dan Browser DevTools

Proxy bukan satu-satunya sumber bukti. `curl` memberi request yang mudah direproduksi, sedangkan browser DevTools membantu melihat network timing, initiator, storage, service worker, dan console. Contoh laboratorium read-only:

```bash
curl -isk -D evidence/home.headers -o evidence/home.body https://<LAB-HOST>/
```

Opsi `-k` mengabaikan verifikasi sertifikat dan hanya layak pada laboratorium yang memang memakai sertifikat uji. Dalam assessment normal, kegagalan validasi TLS justru merupakan evidence yang harus ditangani, bukan dilewati secara otomatis.

Untuk provenance, catat command, versi alat, waktu, source IP, redirect policy, dan hash artefak bila relevan. Jangan menyalin token produksi ke command history.

## 5.13 Pemetaan Alur Pengguna dan Endpoint Inventory

Mulailah dari alur pengguna, bukan endpoint acak. Jalankan satu skenario secara lengkap—misalnya registrasi, login, melihat profil, mengubah profil, dan logout—sambil merekam history. Setelah itu, pecah setiap alur menjadi request yang membentuk keputusan server.

### 5.13.1 Field Endpoint Inventory

| Field | Isi Minimum |
| :--- | :--- |
| `endpoint_id` | Identifier stabil |
| `origin` | Scheme, host, port |
| `method_path` | Method dan normalized path |
| `content_type` | Request dan response media type |
| `parameters` | Path, query, header, cookie, body |
| `auth_state` | Unauthenticated / authenticated |
| `roles_seen` | Peran yang memanggil endpoint |
| `state_change` | Read / create / update / delete / unknown |
| `evidence_ref` | Request/response dan waktu |
| `confidence` | Confirmed / inferred / unknown |
| `next_test` | Hipotesis yang perlu divalidasi |

Normalisasi path penting agar `/api/orders/314` dan `/api/orders/315` dikenali sebagai pola `/api/orders/{id}` tanpa kehilangan contoh instance. Simpan evidence instance secara terpisah.

### 5.13.2 Perbandingan Antarperan

Gunakan akun uji yang berbeda dan profile/session terpisah. Bandingkan endpoint dan respons yang muncul untuk guest, pengguna biasa, dan administrator laboratorium. Jangan mencampur cookie antarperan tanpa pencatatan karena hasil dapat salah diatribusikan.

## 5.14 Fingerprinting sebagai Hipotesis Teknologi

Wappalyzer, WhatWeb, header, favicon, cookie name, HTML meta tag, file JavaScript, dan pola error dapat memberi petunjuk teknologi. OWASP WSTG merekomendasikan fingerprinting server, framework, dan aplikasi sebagai bagian information gathering.

Kualitas klaim fingerprint dapat digambarkan sebagai berikut:
$$\text{single header} < \text{multiple passive indicators} < \text{behavior match} < \text{authenticated artifact} < \text{inventory/source confirmation}$$

Header `X-Powered-By` dapat dipalsukan atau berasal dari komponen depan. Nama file `jquery-3.6.0.min.js` membuktikan asset dengan nama tertentu disajikan, bukan bahwa seluruh aplikasi menggunakan library tersebut pada jalur yang rentan. Version inference harus dilaporkan dengan confidence dan limitation.

Contoh laboratorium:
```bash
whatweb --log-json=evidence/whatweb.json https://<LAB-HOST>/
```

Jangan menjalankan aggression level yang menambah banyak request tanpa persetujuan. Konfirmasi hasil dengan raw response, asset, atau dokumentasi aplikasi bila tersedia.

## 5.15 Content Discovery yang Terkendali

Content discovery atau forced browsing mencoba candidate path dari wordlist untuk menemukan resource yang tidak terhubung pada navigasi normal. `ffuf`, Gobuster, dan feroxbuster mempunyai fitur berbeda, tetapi kualitas hasil lebih dipengaruhi scope, wordlist, baseline, filter, dan rate daripada nama alat.

### 5.15.1 Baseline Sebelum Wordlist

Beberapa aplikasi mengembalikan `200 OK` untuk seluruh path yang tidak ada, memakai halaman error berukuran serupa, atau mengarahkan semua request ke login. Sebelum scan:
1. minta beberapa path acak yang dipastikan tidak ada;
2. catat status, ukuran, jumlah kata/baris, title, dan redirect;
3. periksa apakah wildcard response stabil;
4. pilih filter berdasarkan baseline; dan
5. uji filter dengan path nyata yang diketahui.

Contoh laboratorium ber-rate rendah:
```bash
ffuf -w <LAB-WORDLIST> -u https://<LAB-HOST>/FUZZ -rate 5 -mc all -fs <BASELINE-SIZE> -of json -o evidence/content-discovery.json
```

`403 Forbidden` dapat berarti resource ada tetapi ditolak, atau merupakan respons generik WAF. `301/302` perlu ditelusuri dengan hati-hati karena redirect target dapat keluar scope. `200` perlu dibandingkan dengan wildcard baseline.

### 5.15.2 Wordlist dan Konteks

SecLists menyediakan banyak kategori wordlist, tetapi daftar besar bukan selalu pilihan terbaik. Mulailah dari daftar pendek yang sesuai teknologi, bahasa, dan fungsi aplikasi. Tambahkan kata yang berasal dari route, dokumentasi, JavaScript, atau istilah bisnis target. Relevansi mengurangi noise dan dampak.

### 5.15.3 Recursion dan Ekstensi

Recursion dapat mengalikan jumlah request secara drastis. Ekstensi seperti `.bak`, `.old`, atau arsip dapat menghasilkan akses terhadap data sensitif. Keduanya memerlukan persetujuan eksplisit, batas depth, maximum requests, dan prosedur penanganan data.

## 5.16 Enumerasi REST dan OpenAPI

REST adalah gaya arsitektur, bukan format URL yang wajib. API dapat menggunakan resource-oriented path dan method HTTP, tetapi implementasi nyata sering memiliki RPC-style endpoint. Pemetaan harus berdasarkan traffic dan dokumentasi, bukan asumsi nama.

OpenAPI Specification 3.2.0 mendefinisikan deskripsi antarmuka HTTP API yang dapat dibaca manusia dan mesin. Dokumen OpenAPI yang tersedia di dalam scope dapat membantu mengidentifikasi server, path, operation, parameter, request body, response, security scheme, dan schema. Keberadaan dokumentasi bukan otomatis "kebocoran"; dampaknya bergantung pada intended audience, data yang dipaparkan, dan kontrol akses.

Workflow aman:
1. temukan request API dari browser history;
2. kelompokkan path menjadi template resource;
3. cari dokumentasi resmi atau OpenAPI hanya pada path yang diizinkan;
4. bandingkan documented endpoint dengan observed endpoint;
5. catat auth scheme dan peran yang teramati;
6. identifikasi identifier objek dan perubahan state; dan
7. buat hipotesis untuk bab pengujian otorisasi dan input validation.

OWASP API Security Top 10:2023 menekankan broken object level authorization, broken authentication, broken object property level authorization, serta unrestricted resource consumption. Pada bab ini kategori tersebut digunakan sebagai lensa pemetaan, bukan sebagai klaim bahwa kelemahannya sudah ditemukan.

## 5.17 GraphQL dan Introspection

GraphQL umumnya menerima query pada satu atau beberapa endpoint dan memungkinkan klien memilih field yang dibutuhkan. Karena URL tunggal dapat mewakili banyak operasi, endpoint inventory perlu mencatat operation name, root field, argument, object type, auth state, dan response shape.

Introspection adalah fitur standar yang memungkinkan klien mempelajari schema dan mendukung tooling pengembangan. Introspection aktif bukan kerentanan otomatis. Risiko meningkat ketika schema yang tidak dimaksudkan untuk publik membuka informasi sensitif, ketika authorization lemah, atau ketika query yang mahal tidak dibatasi.

Pada laboratorium yang mengizinkan introspection, gunakan query minimum untuk mengonfirmasi tipe dasar, lalu hindari dump besar tanpa kebutuhan. Perhatikan juga persisted query, batching, alias, depth, complexity, timeout, dan rate limit sebagai bagian attack surface.

Bahasa laporan yang tepat. Hindari "GraphQL rentan karena introspection aktif." Gunakan: "Endpoint GraphQL pada `<path>` mengizinkan introspection tanpa autentikasi dan memaparkan nama tipe/field berikut; intended exposure serta dampak terhadap attack surface perlu dikonfirmasi."

## 5.18 WebSocket dan Traffic Asinkron

WebSocket memulai komunikasi melalui HTTP handshake lalu beralih menjadi kanal pesan dua arah. Proxy modern dapat merekam dan mengirim ulang pesan WebSocket. Pemetaan perlu mencatat URL handshake, Origin, cookie/token, subprotocol, arah pesan, tipe event, identifier, dan perubahan state.

Selain WebSocket, aplikasi dapat memakai Server-Sent Events, long polling, background sync, service worker, atau request asinkron. Bila penguji hanya melihat navigasi halaman, komunikasi tersebut mudah terlewat.

Gunakan satu alur pengguna untuk menghubungkan aksi UI dengan message timeline. Jangan mengirim ulang event yang bersifat transaksi—seperti pembayaran atau penghapusan—tanpa reset state dan persetujuan.

## 5.19 Dari Observasi ke Hipotesis Pengujian

Pemetaan yang baik menghasilkan hipotesis spesifik, bukan daftar tool output. Contoh transformasi:

| Observasi | Hipotesis yang Sah | Belum Terbukti |
| :--- | :--- | :--- |
| `/api/orders/{id}` dipanggil akun student | Server perlu memeriksa kepemilikan objek | Broken access control |
| Cookie tidak memiliki `HttpOnly` | Script dapat membaca cookie tersebut | Adanya XSS atau takeover |
| Source map tersedia | Source client mungkin lebih mudah dianalisis | Secret atau kerentanan |
| Endpoint GraphQL mengizinkan introspection | Schema dapat dipetakan | Akses data tanpa izin |
| `/backup/` memberi `403` | Suatu route atau kontrol mungkin ada | File sensitif dapat diunduh |

Hipotesis perlu memiliki prasyarat, request baseline, perubahan yang diusulkan, expected control, stop condition, dan evidence requirement. Struktur ini mencegah eksperimen tanpa tujuan.

## 5.20 Evidence, Provenance, dan Sanitasi

Artefak pengujian web dapat mengandung data lebih sensitif daripada output port scanner. Paket bukti minimum mencakup:

1. endpoint dan waktu dengan zona waktu;
2. akun/peran uji tanpa password;
3. raw request yang relevan;
4. raw response yang relevan;
5. langkah UI atau kondisi state;
6. alat dan versi;
7. perubahan yang dilakukan terhadap baseline;
8. hasil, limitation, dan confidence; serta
9. referensi ke screenshot atau video bila benar-benar membantu.

Gunakan placeholder seperti `<REDACTED-SESSION>`, tetapi pertahankan nama header, panjang relatif, atau claim non-sensitif yang diperlukan untuk analisis. Hash token bukan selalu anonimisasi yang aman apabila ruang token kecil atau token dapat dikorelasikan; lebih aman mengganti nilainya secara konsisten dengan identifier fiktif.

### 5.20.1 Diff yang Bermakna

Perbandingan response harus mengabaikan noise seperti timestamp, request ID, nonce, advertisement, dan urutan field yang tidak relevan. Catat perubahan status, header penting, schema, field authorization, serta perubahan state server. Jangan menyimpulkan dari perbedaan ukuran response saja.

## 5.21 Passive Scan, Active Scan, dan DevSecOps

ZAP passive scanner menganalisis pesan HTTP/WebSocket tanpa mengubahnya. Namun proses yang menghasilkan traffic—misalnya spider—tetap dapat melakukan request baru. Karena itu "passive scan" tidak selalu berarti keseluruhan workflow pasif terhadap target.

ZAP active scan menggunakan known attacks dan secara eksplisit merupakan serangan terhadap target. Dokumentasinya juga menegaskan bahwa automated scan tidak dapat menemukan seluruh logical vulnerability. Active scan hanya dijalankan pada target milik sendiri atau dengan izin, memakai policy dan scope yang telah disetujui.

ZAP Baseline Scan menjalankan spider dalam waktu terbatas lalu menunggu passive scan selesai. Ia berguna untuk CI/CD sebagai quality signal, tetapi hasil perlu dikonfigurasi: alert mana menjadi WARN, FAIL, atau IGNORE; target dan auth context apa yang dicakup; serta bagaimana baseline drift ditangani.

Shift-left bukan memindahkan seluruh penetration test ke pipeline. SAST, dependency scanning, secret scanning, DAST, unit security test, dan manual assessment memiliki visibility berbeda. Defender perlu menggabungkannya dengan threat modeling dan verifikasi perbaikan.

## 5.22 Perspektif Defender

Pemetaan attack surface memberikan nilai langsung bagi tim pengembang dan operasi. Data penguji dapat dipakai untuk:
1. membangun inventory origin, API, dan dependency eksternal;
2. menemukan endpoint yang tidak lagi memiliki owner;
3. memastikan route dan method mempunyai authorization policy;
4. mengurangi versi/banner yang tidak perlu;
5. menghapus backup, source map, debug route, atau dokumentasi yang tidak semestinya publik;
6. membatasi CORS dan cookie sesuai kebutuhan nyata;
7. menambahkan rate limit serta logging pada endpoint sensitif; dan
8. menghubungkan DAST dengan asset owner dan SLA remediasi.

Metrik yang lebih berguna daripada "jumlah URL" antara lain persentase endpoint dengan owner, persentase endpoint terdokumentasi, coverage alur bisnis kritis, peran yang diuji, authenticated coverage, jumlah endpoint yatim, median waktu penutupan misconfiguration, serta recurrence rate.

## 5.23 Kerangka JELAJAH

Bab ini memperkenalkan kerangka orisinal JELAJAH untuk menjaga pemetaan aplikasi web tetap terotorisasi, sistematis, dan dapat diaudit.

1. **J**amin kewenangan – tetapkan origin, akun, aktivitas, pihak ketiga, waktu, dan stop condition.
2. **E**ksplorasi alur pengguna – jalankan fungsi bisnis dengan state dan peran yang jelas.
3. **L**acak seluruh traffic – gunakan proxy history, DevTools, dan evidence yang memiliki waktu.
4. **A**nalisis arsitektur dan teknologi – bedakan observasi, fingerprint, dan konfirmasi.
5. **J**elajahi content dan API – gunakan wordlist, dokumentasi, serta rate yang disetujui.
6. **A**rsipkan evidence – simpan request/response minimum, provenance, dan sanitasi secret.
7. **H**andoff hipotesis – susun test map untuk otorisasi, session, input, dan business logic.

JELAJAH melanjutkan kerangka PINDAI pada Bab 4. PINDAI mengendalikan vulnerability scanning; JELAJAH mengubah traffic dan route aplikasi menjadi attack surface register serta hipotesis yang siap diuji.

## 5.24 Studi Kasus Terbimbing: Portal Akademik Fiktif

### 5.24.1 Skenario

Tim memperoleh izin menilai portal akademik laboratorium yang terdiri atas `portal.lab.test`, `api.lab.test`, dan `ws.lab.test`. Tersedia akun guest, student, dan lecturer. Payment gateway eksternal, penyedia SSO publik, dan analytics dikecualikan. Content discovery dibatasi lima request per detik; active scan dan brute force dilarang.

### 5.24.2 Observasi

1. Halaman awal server-rendered, lalu profile dan course dimuat melalui JSON API.
2. Login menghasilkan cookie `portal_session` dengan Secure dan HttpOnly; nilai SameSite belum terlihat.
3. Student dan lecturer memanggil pola `/api/v2/courses/{id}`.
4. Lecturer memiliki endpoint tambahan `/api/v2/grades/export`.
5. JavaScript merujuk `/graphql`, tetapi traffic normal belum memanggilnya.
6. WebSocket digunakan untuk notification event.
7. Content discovery menemukan `/docs/` dengan `200` dan `/admin/` dengan `302` menuju login.
8. WhatWeb mengindikasikan reverse proxy Nginx dan framework frontend, tetapi versi backend tidak terkonfirmasi.

### 5.24.3 Analisis

| Observasi | Status | Tindakan Berikutnya |
| :--- | :--- | :--- |
| Cookie tanpa nilai SameSite terlihat | Supported observation | Periksa seluruh Set-Cookie dan konteks auth |
| Endpoint course memakai identifier | Attack-surface hypothesis | Uji ownership pada bab otorisasi |
| `/grades/export` hanya muncul untuk lecturer | Role-specific route | Konfirmasi enforcement server-side |
| `/graphql` hanya muncul di JavaScript | Inferred endpoint | Lakukan baseline request bila scope mengizinkan |
| `/docs/` dapat diakses | Confirmed route | Identifikasi intended audience dan content |
| Nginx terdeteksi | Fingerprint hypothesis | Korelasikan header, TLS, dan behavior |

Tidak ada observasi di atas yang otomatis menjadi finding. Hasil tahap fondasi adalah peta yang mengurangi blind spot dan memandu pengujian berikutnya.

## 5.25 Praktik Terbimbing

### 5.25.1 Lingkungan

Gunakan OWASP Juice Shop, WebGoat, DVWA, atau aplikasi vulnerable-by-design lain pada jaringan host-only/laboratorium. Jangan mengekspos aplikasi secara tidak sengaja ke Internet. Sediakan minimal dua akun uji dengan peran berbeda apabila aplikasi mendukungnya.

Perangkat yang dapat dipilih:
1. Burp Suite Community Edition atau OWASP ZAP;
2. browser profile khusus laboratorium;
3. `curl` dan browser DevTools;
4. WhatWeb atau Wappalyzer;
5. `ffuf`, Gobuster, atau feroxbuster; dan
6. wordlist kecil yang berasal dari SecLists atau daftar khusus laboratorium.

### 5.25.2 Tugas

1. Susun web test plan yang menetapkan target, origin, akun, aktivitas, rate, evidence, dan stop condition.
2. Konfigurasikan scope proxy dan browser profile khusus.
3. Jelajahi minimal tiga alur pengguna sambil merekam HTTP history.
4. Pilih lima request penting dan uraikan method, path, parameter, state, serta response.
5. Inventarisasi cookie/token dan atribut keamanannya tanpa menyimpan secret asli.
6. Bandingkan traffic dua peran atau dua state autentikasi.
7. Lakukan fingerprinting pasif dan beri confidence pada setiap klaim.
8. Buat baseline random path, kemudian jalankan content discovery ber-rate rendah.
9. Petakan REST, GraphQL, WebSocket, atau komunikasi asinkron yang ditemukan.
10. Susun endpoint inventory dan attack surface register.
11. Tulis minimal lima hipotesis pengujian berikut expected control dan evidence requirement.
12. Lakukan sanitasi artefak sebelum dikumpulkan.

### 5.25.3 Rubrik Penilaian

| Aspek | Bobot | Kriteria |
| :--- | :--- | :--- |
| **Kewenangan dan scope** | 15% | Origin, pihak ketiga, aktivitas, rate, stop condition jelas |
| **Model arsitektur** | 10% | Komponen dan trust boundary tidak dicampur dengan asumsi |
| **HTTP dan browser model** | 15% | Method, state, SOP/CORS/CSP/cookie ditafsirkan tepat |
| **Proxy workflow** | 15% | History terfilter, baseline, perubahan satu variabel |
| **Attack surface coverage** | 20% | Route, API, parameter, role, WebSocket/asinkron terpetakan |
| **Evidence dan provenance** | 15% | Request/response, waktu, alat, sanitasi, limitation lengkap |
| **Hipotesis dan handoff** | 10% | Testable, terukur, tidak mengklaim kerentanan prematur |

### 5.25.4 Quality Gate

Sebelum pengumpulan, pastikan:
1. tidak ada target di luar scope;
2. tidak ada password, session cookie, bearer token, atau data pribadi asli;
3. CA pengujian hanya berada pada browser profile laboratorium;
4. semua endpoint memiliki evidence atau diberi label inferred;
5. fingerprint menyertakan confidence dan limitation;
6. wildcard baseline terdokumentasi sebelum content discovery;
7. 200, 302, 403, atau perbedaan ukuran tidak langsung disebut vulnerability;
8. GraphQL introspection tidak otomatis disebut finding;
9. hipotesis menjelaskan expected control dan perubahan yang akan diuji; dan
10. seluruh artefak dapat ditelusuri ke waktu, alat, akun/peran, dan alur pengguna.

## Rangkuman Bab

1. Pengujian web dimulai dari arsitektur, semantics HTTP, state, browser security model, dan alur bisnis—bukan dari payload.
2. Origin ditentukan oleh scheme, host, dan port; same-site tidak selalu sama dengan same-origin.
3. Safe dan idempotent adalah semantics method, bukan jaminan bahwa implementasi tidak memiliki efek samping.
4. SOP membatasi pembacaan lintas origin, CORS memberi izin lintas origin pada browser, dan CSP membatasi resource/eksekusi; ketiganya bukan kontrol yang saling menggantikan.
5. Cookie, session, authentication, dan authorization memiliki fungsi berbeda dan harus dipetakan secara terpisah.
6. Proxy intersep adalah sensor sekaligus aktuator; HTTPS inspection memerlukan CA pengujian yang harus dikelola sebagai aset sensitif.
7. Fingerprinting menghasilkan hipotesis teknologi. Content discovery membutuhkan wildcard baseline, filter, wordlist relevan, rate limit, dan scope.
8. OpenAPI dan GraphQL introspection membantu pemetaan, tetapi keberadaannya bukan kerentanan otomatis.
9. Passive scan tidak mengubah pesan yang dianalisis, tetapi spider yang menghasilkan traffic tetap aktif terhadap target.
10. Attack surface register menyatukan endpoint, parameter, peran, state, evidence, confidence, dan next test.
11. Kerangka JELAJAH menghubungkan kewenangan, alur pengguna, traffic, arsitektur, discovery, evidence, dan handoff hipotesis.

## Uji Pemahaman

### Pilihan Ganda

1. Pernyataan yang paling tepat mengenai method PUT adalah:
   * A. selalu memperbarui sebagian field pada resource
   * **B. meminta pembuatan atau penggantian state target sesuai representasi**
   * C. selalu aman karena bersifat idempotent
   * D. hanya digunakan oleh REST API

2. Dua URL disebut same-origin apabila memiliki ...
   * A. domain registrable yang sama
   * B. alamat IP yang sama
   * **C. scheme, host, dan port yang sama**
   * D. sertifikat TLS yang sama

3. Pernyataan paling tepat mengenai CORS adalah ...
   * A. firewall yang menghentikan semua request lintas origin
   * **B. mekanisme header agar server memberi izin baca lintas origin kepada browser**
   * C. mekanisme autentikasi API
   * D. pengganti same-origin policy

4. Fungsi utama atribut HttpOnly adalah:
   * A. mengenkripsi cookie pada server
   * B. mencegah cookie dikirim melalui HTTP
   * **C. mencegah JavaScript membaca cookie melalui DOM**
   * D. mencegah seluruh dampak XSS

5. Saat random path dan path nyata sama-sama menghasilkan status 200 serta ukuran serupa, tindakan pertama adalah ...
   * A. melaporkan seluruh path sebagai valid
   * B. menaikkan jumlah thread
   * **C. membuat wildcard baseline dan memilih filter yang tepat**
   * D. menjalankan exploit publik

6. GraphQL introspection yang aktif ...
   * A. selalu merupakan kerentanan critical
   * **B. merupakan fitur standar yang dampaknya harus dinilai berdasarkan exposure dan kontrol**
   * C. membuktikan authorization tidak ada
   * D. hanya tersedia pada GraphQL versi lama

7. Peran utama Repeater adalah ...
   * A. menjalankan port scan
   * **B. memodifikasi dan mengirim ulang request secara manual**
   * C. menyusun SBOM
   * D. mengganti CA server produksi

8. Bukti paling kuat untuk mengonfirmasi versi komponen backend adalah ...
   * A. satu header Server
   * B. favicon yang mirip
   * **C. inventory atau artefak terautentikasi yang dikorelasikan dengan behavior**
   * D. nama framework pada URL

9. Pernyataan yang tepat mengenai ZAP passive scan adalah ...
   * A. seluruh workflow pasti tidak mengirim request baru
   * **B. pesan yang dianalisis tidak diubah, tetapi spider dapat menghasilkan traffic**
   * C. dapat menemukan seluruh business logic flaw
   * D. aman dijalankan pada target apa pun

10. Field yang paling membantu membedakan route terkonfirmasi dari dugaan adalah ...
    * A. warna screenshot
    * **B. confidence dan evidence reference**
    * C. panjang wordlist
    * D. nama penguji

11. Tujuan utama endpoint inventory adalah ...
    * A. mengumpulkan URL sebanyak mungkin
    * **B. menghubungkan route, parameter, state, peran, dan evidence**
    * C. menggantikan laporan penetration testing
    * D. menghapus kebutuhan pengujian manual

12. Langkah terakhir kerangka JELAJAH adalah ...
    * A. Jalankan exploit
    * **B. Handoff hipotesis**
    * C. Hapus semua evidence
    * D. Aktifkan active scan

### Kunci Jawaban Pilihan Ganda
1. B | 2. C | 3. B | 4. C | 5. C | 6. B | 7. B | 8. C | 9. B | 10. B | 11. B | 12. B

## Pertanyaan Esai

1. Jelaskan mengapa browser dapat mengirim request cross-origin tetapi script belum tentu dapat membaca response-nya.
2. Bedakan fungsi Secure, HttpOnly, dan SameSite pada cookie serta berikan batas masing-masing kontrol.
3. Susun prosedur lima langkah untuk membuktikan apakah hasil content discovery merupakan resource nyata atau wildcard response.
4. Jelaskan mengapa OpenAPI publik dan GraphQL introspection aktif tidak otomatis menjadi kerentanan.
5. Bandingkan kegunaan Burp Suite dan OWASP ZAP untuk pembelajaran S1 tanpa menjadikan popularitas sebagai kriteria utama.
6. Rancang endpoint inventory untuk satu alur "unggah tugas" yang memakai API dan object storage.
7. Jelaskan bagaimana artifact proxy dapat menjadi risiko keamanan bagi penguji dan organisasi.
8. Gunakan kerangka JELAJAH untuk merancang assessment awal terhadap SPA dengan tiga peran pengguna.

## Glosarium

* **Active scan:** Pemindaian yang mengirim input atau pola serangan untuk mencari potensi kerentanan pada target.
* **Attack surface register:** Inventaris terstruktur tentang origin, route, endpoint, parameter, state, peran, evidence, dan hipotesis pengujian.
* **CORS:** Mekanisme header HTTP yang memungkinkan server menyatakan origin mana yang boleh membaca response lintas origin melalui browser.
* **CSP:** Kebijakan yang menginstruksikan browser untuk membatasi sumber dan jenis resource atau eksekusi tertentu.
* **Content discovery:** Pencarian resource yang tidak terhubung pada navigasi normal dengan candidate path dan observasi response.
* **Endpoint:** Alamat dan operasi yang menerima request atau pesan tertentu pada aplikasi atau API.
* **Fingerprinting:** Proses menginferensikan teknologi dari indikator seperti header, asset, cookie, behavior, dan artefak lain.
* **Idempotent:** Sifat semantics method ketika beberapa request identik mempunyai intended effect yang sama dengan satu request.
* **Introspection:** Kemampuan GraphQL untuk menjelaskan schema melalui query khusus.
* **JELAJAH:** Kerangka pemetaan web: Jamin kewenangan, Eksplorasi alur, Lacak traffic, Analisis arsitektur, Jelajahi content/API, Arsipkan evidence, dan Handoff hipotesis.
* **Origin:** Kombinasi scheme, host, dan port yang digunakan browser sebagai batas keamanan.
* **Passive scan:** Analisis pesan yang telah tersedia tanpa mengubah pesan tersebut; sumber traffic tetap perlu diperhatikan.
* **Proxy intersep:** Perantara yang dapat merekam, menahan, dan memodifikasi komunikasi antara klien dan aplikasi.
* **Same-origin policy:** Kebijakan browser yang membatasi interaksi script lintas origin.
* **Session:** State aplikasi yang menghubungkan serangkaian request dengan konteks pengguna atau proses.
* **SPA:** Aplikasi yang memuat shell client lalu memperbarui tampilan dan data melalui JavaScript serta API tanpa selalu memuat ulang dokumen penuh.
* **Wildcard response:** Respons generik yang diberikan untuk banyak path tidak valid sehingga dapat menyerupai resource nyata.

## Referensi

[1] OffSec. "OSCP+ Body of Knowledge." 2026. `https://help.offsec.com/hc/en-us/articles/38543335188756-OSCP-Body-of-knowledge`

[2] OffSec. "PEN-200: Penetration Testing with Kali Linux Syllabus." 2026. `https://manage.offsec.com/app/uploads/2026/03/PEN-200_Syllabus.pdf`

[3] OffSec. "PEN-200 Learning Plan - 12 Week." 2025. `https://help.offsec.com/hc/en-us/articles/15541765522196-OffSec-PEN-200-Learning-Plan-12-Week`

[4] IETF. "RFC 9110: HTTP Semantics." 2022. `https://www.rfc-editor.org/rfc/rfc9110.html`

[5] IETF. "RFC 9112: HTTP/1.1." 2022. `https://www.rfc-editor.org/rfc/rfc9112.html`

[6] IETF. "RFC 9113: HTTP/2." 2022. `https://www.rfc-editor.org/rfc/rfc9113.html`

[7] IETF. "RFC 9114: HTTP/3." 2022. `https://www.rfc-editor.org/rfc/rfc9114.html`

[8] IETF. "RFC 6265: HTTP State Management Mechanism." 2011. `https://www.rfc-editor.org/rfc/rfc6265.html`

[9] OWASP. "Web Security Testing Guide." `https://owasp.org/www-project-web-security-testing-guide/`

[10] OWASP. "WSTG: Information Gathering." `https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/01-Information_Gathering/`

[11] OWASP. "WSTG: Identify Application Entry Points." `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/06-Identify_Application_Entry_Points`

[12] OWASP. "WSTG: Map Execution Paths Through Application." `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/07-Map_Execution_Paths_Through_Application`

[13] OWASP. "OWASP Top 10:2025." `https://owasp.org/Top10/2025/`

[14] OWASP. "API Security Top 10-2023." `https://owasp.org/API-Security/editions/2023/en/0x11-t10/`

[15] Mozilla. "Same-origin policy." MDN Web Docs. `https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy`

[16] Mozilla. "Cross-Origin Resource Sharing." MDN Web Docs. `https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS`

[17] Mozilla. "Content Security Policy." MDN Web Docs. `https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP`

[18] OWASP. "Session Management Cheat Sheet." `https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html`

[19] OWASP. "Testing for Cookies Attributes." `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/02-Testing_for_Cookies_Attributes`

[20] PortSwigger. "Burp Proxy." `https://portswigger.net/burp/documentation/desktop/tools/proxy`

[21] PortSwigger. "Setting the Initial Test Scope." `https://portswigger.net/burp/documentation/desktop/testing-workflow/test-scope`

[22] PortSwigger. "Installing Burp's CA Certificate." `https://portswigger.net/burp/documentation/desktop/external-browser-config/certificate`

[23] PortSwigger. "Burp Repeater." `https://portswigger.net/burp/documentation/desktop/tools/repeater`

[24] PortSwigger. "Burp Intruder." `https://portswigger.net/burp/documentation/desktop/tools/intruder`

[25] ZAP. "Getting Started." `https://www.zaproxy.org/docs/desktop/start/`

[26] ZAP. "Manipulator-in-the-middle Proxy." `https://www.zaproxy.org/docs/desktop/start/features/intercept/`

[27] ZAP. "Passive Scan." `https://www.zaproxy.org/docs/desktop/start/features/pscan/`

[28] ZAP. "Active Scan." `https://www.zaproxy.org/docs/desktop/start/features/ascan/`

[29] ZAP. "Baseline Scan." `https://www.zaproxy.org/docs/docker/baseline-scan/`

[30] WhatWeb Project. "WhatWeb - Next Generation Web Scanner." `https://github.com/urbanadventurer/WhatWeb`

[31] Wappalyzer. "Technologies." `https://www.wappalyzer.com/technologies/`

[32] ffuf Project. "Fuzz Faster U Fool." `https://github.com/ffuf/ffuf`

[33] Gobuster Project. "Gobuster." `https://github.com/OJ/gobuster`

[34] feroxbuster Project. "feroxbuster Documentation." `https://epi052.github.io/feroxbuster-docs/overview/`

[35] SecLists Project. "SecLists." `https://github.com/danielmiessler/SecLists`

[36] OpenAPI Initiative. "OpenAPI Specification v3.2.0." 2025. `https://spec.openapis.org/oas/v3.2.0.html`

[37] GraphQL Foundation. "Introspection." `https://graphql.org/learn/introspection/`

[38] GraphQL Foundation. "Best Practices." `https://graphql.org/faq/best-practices/`

[39] curl Project. "curl Manual." `https://curl.se/docs/manpage.html`

*Seluruh sumber daring diperiksa pada 31 Agustus 2026. Materi PEN-200 digunakan sebagai rujukan cakupan kompetensi; narasi, kerangka JELAJAH, studi kasus, struktur latihan, dan contoh pada bab ini disusun secara mandiri.*