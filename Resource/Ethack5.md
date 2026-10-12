## Daftar Isi

1. [Ringkasan Textbook](#ringkasan-textbook)
2. [Step by Step & Link](#step-by-step--link)
3. [Readme dan Isi Paket](#readme-dan-isi-paket)
4. [Template Peta Serangan](#template-peta-serangan)


## Ringkasan Textbook

Mulai bab ini, kita akan menghabiskan cukup banyak waktu — sekitar lima bab ke depan sebenarnya — di dunia aplikasi web. Ada alasan kenapa porsinya sebesar ini: dari semua permukaan serangan yang bakal kamu temui sebagai pentester, aplikasi web hampir selalu jadi yang paling sering muncul, paling cepat berubah, dan paling beragam variasinya. Server bisa di-patch rutin oleh tim infrastruktur, tapi aplikasi web yang dikembangkan sendiri oleh developer internal organisasi? Itu wilayah yang jauh lebih "liar".

Sebelum kita mulai cari-cari kerentanan spesifik (itu baru mulai Bab 6), bab ini fokus ke dua hal dasar yang wajib banget kamu kuasai dulu: memahami cara kerja komunikasi web itu sendiri, dan menguasai tools yang akan jadi "mata dan tangan" kamu sepanjang pengujian aplikasi web.

## 5.1 Anatomi Komunikasi HTTP: Fondasi yang Sering Dianggap Remeh

Saya sering ketemu orang yang bisa pakai Burp Suite dengan lincah tapi giliran ditanya "kenapa method PUT beda dari POST secara semantik?", jawabannya cuma diam. Padahal pemahaman dasar ini yang bikin kamu bisa membaca traffic dengan cerdas, bukan cuma klak-klik tombol di tool.

Setiap komunikasi web pada dasarnya adalah pertukaran **request** dan **response**. Klien (browser, atau tool seperti curl/Burp) mengirim request yang terdiri dari **method** (GET, POST, PUT, DELETE, dan sebagainya — masing-masing punya makna semantik: GET untuk mengambil data, POST untuk mengirim data baru, PUT untuk update, DELETE ya jelas untuk menghapus), **header** (metadata tambahan seperti jenis konten, autentikasi, cookie), dan kadang **body** (data sungguhan yang dikirim, misalnya isi form).

Server merespons dengan **status code** (200 untuk sukses, 404 untuk tidak ditemukan, 500 untuk error server, dan puluhan kode lain yang masing-masing punya arti spesifik), header response, dan body response (biasanya HTML, JSON, atau format lain tergantung jenis aplikasinya).

**Cookie** dan **session** adalah mekanisme yang dipakai untuk "mengingat" siapa kamu di request-request selanjutnya — karena HTTP itu sendiri sebenarnya *stateless* (tidak punya memori antar request). Server biasanya memberi kamu sebuah session ID lewat cookie setelah login berhasil, dan browser otomatis mengirim ulang cookie itu di setiap request berikutnya supaya server tahu "oh, ini orang yang sama yang tadi sudah login".

## 5.2 Server-Rendered vs SPA/API-Driven: Kenapa Ini Mengubah Cara Kamu Bekerja

Aplikasi web zaman sekarang nggak semuanya dibangun dengan cara yang sama, dan ini punya implikasi langsung ke cara kamu melakukan pengujian.

Model **server-rendered** klasik (pikirkan aplikasi PHP lawas) menghasilkan HTML lengkap di server, lalu mengirimkannya utuh ke browser. Setiap kali kamu klik link atau submit form, seluruh halaman biasanya di-reload dari server.

Model **SPA (Single Page Application)** yang lebih modern (dibangun dengan React, Vue, Angular, dan semacamnya) bekerja beda — browser cuma mengunduh "kerangka" aplikasi sekali, lalu selanjutnya berkomunikasi dengan server lewat panggilan API (biasanya format JSON) di belakang layar, tanpa reload halaman penuh.

Kenapa ini penting buat kamu? Karena permukaan serangan pada aplikasi SPA/API-driven jauh lebih "tersembunyi" dari pandangan sekilas — kalau kamu cuma browsing manual seperti pengguna biasa, kamu akan melewatkan banyak endpoint API yang sebenarnya jadi jantung logika aplikasi tersebut. Di sinilah proxy intersep jadi krusial — dia menangkap semua komunikasi "di belakang layar" itu yang nggak kelihatan kalau cuma lihat tampilan browser.

## 5.3 Proxy Intersep: Mata Kamu untuk Melihat "Di Balik Layar"

**Burp Suite** (versi Community gratis) dan **OWASP ZAP** (sepenuhnya open-source) adalah dua tool proxy intersep paling populer di dunia pentest web. Cara kerjanya sederhana secara konsep: kamu setting browser untuk mengirim semua traffic-nya lewat proxy tool ini dulu, sebelum diteruskan ke server tujuan. Dengan begitu, kamu bisa melihat, menahan, bahkan memodifikasi setiap request sebelum benar-benar terkirim.

Beberapa fitur kunci yang wajib kamu kuasai:

**Proxy History** mencatat semua traffic yang lewat — ini "buku catatan" utama kamu untuk memahami bagaimana aplikasi berkomunikasi dengan server-nya.

**Repeater** memungkinkan kamu mengambil satu request tertentu, memodifikasinya, lalu mengirim ulang berkali-kali untuk melihat bagaimana server merespons variasi input yang berbeda-beda — ini akan jadi tool andalanmu mulai Bab 6 saat mulai mencari kerentanan spesifik.

**Intruder** (di Burp) mengotomasi pengiriman banyak variasi payload sekaligus ke satu titik tertentu dalam request — berguna untuk fuzzing parameter atau brute force sederhana.

## 5.4 Fingerprinting: Kenali Teknologi di Baliknya Sebelum Menyerang

Sebelum menyerang, ada baiknya kamu tahu dulu "senjata" apa yang dipakai musuh. **Wappalyzer** (ekstensi browser) otomatis mendeteksi teknologi yang dipakai sebuah website — mulai dari framework frontend, CMS, analytics tool, sampai kadang versi spesifiknya — hanya dengan menganalisis pola di HTML, header, dan script yang di-load.

**WhatWeb** melakukan hal serupa tapi dari command line, cocok untuk automasi atau saat kamu bekerja tanpa GUI. Kombinasikan dengan analisis header HTTP manual (`curl -I`) untuk konfirmasi tambahan — kadang header `Server` atau `X-Powered-By` secara terang-terangan mengumumkan teknologi backend-nya (meski admin yang lebih hati-hati biasanya menonaktifkan header semacam ini).

## 5.5 Content Discovery: Mencari yang "Tidak Terlihat"

Website yang kamu lihat dengan mata biasa cuma menampilkan link-link yang memang sengaja ditampilkan developer. Tapi seringkali ada halaman, direktori, atau file yang tidak di-link dari manapun, tapi tetap bisa diakses kalau kamu tahu persis URL-nya — entah itu halaman admin lama yang lupa dihapus, file backup, atau endpoint API yang belum didokumentasikan publik.

**Content discovery** adalah teknik untuk menemukan hal-hal tersembunyi ini lewat brute-force sistematis. Tools seperti **gobuster**, **ffuf**, dan **feroxbuster** bekerja dengan cara yang mirip: mencoba ribuan (atau jutaan) kemungkinan nama path dari sebuah wordlist, lalu melihat mana yang memberi response selain 404.

Soal wordlist, kamu nggak perlu bikin sendiri dari nol. **SecLists** adalah koleksi wordlist yang dikurasi komunitas keamanan selama bertahun-tahun — mulai dari daftar nama direktori umum, subdomain, sampai payload untuk berbagai jenis kerentanan. Ini akan jadi salah satu resource paling sering kamu pakai sepanjang buku ini.

## 5.6 Enumerasi API: REST dan GraphQL

Ingat pembahasan soal SPA tadi? Nah, di baliknya biasanya ada API — dan memahami cara enumerasi API adalah skill wajib zaman sekarang.

**REST API** biasanya terstruktur berdasarkan resource (`/api/users`, `/api/users/123`, `/api/orders`) dengan method HTTP yang berbeda menunjukkan aksi berbeda pada resource yang sama. Kalau kamu beruntung, kadang ada spesifikasi **OpenAPI/Swagger** yang "bocor" ke publik (biasanya di path seperti `/api-docs` atau `/swagger.json`) yang secara harfiah memberimu peta lengkap seluruh endpoint yang tersedia — lengkap dengan parameter yang diharapkan.

**GraphQL** bekerja beda — biasanya cuma ada satu endpoint (`/graphql`), tapi klien mengirim query terstruktur yang menentukan data apa saja yang diminta. Banyak implementasi GraphQL yang lupa menonaktifkan fitur **introspection** di lingkungan produksi — fitur ini pada dasarnya membiarkan siapa saja bertanya ke API "hei, skema data apa saja yang kamu punya?" dan mendapat jawaban lengkap. Tools seperti Postman atau Insomnia bisa membantu eksplorasi manual kedua jenis API ini dengan lebih nyaman dibanding curl polos.

## 5.7 Automasi Passive Scanning: Sekilas soal DevSecOps

Sebagai wawasan tambahan, banyak organisasi modern sudah mengintegrasikan scanning keamanan langsung ke pipeline CI/CD mereka — konsep yang sering disebut **DevSecOps**. **ZAP baseline scan**, misalnya, bisa dijalankan otomatis setiap kali ada perubahan kode, melakukan passive scan cepat (tanpa benar-benar menyerang, cuma mengamati traffic normal aplikasi) untuk menangkap masalah keamanan dasar sebelum kode tersebut sampai ke produksi.

Kamu nggak perlu jadi ahli DevSecOps dari bab ini, tapi penting untuk sadar bahwa dunia security testing bergerak ke arah "shift-left" — semakin dini masalah ditemukan dalam siklus development, semakin murah dan mudah diperbaiki dibanding ditemukan setelah aplikasi sudah live di produksi.

## Rangkuman Bab

- Paham method HTTP, header, cookie/session itu fondasi yang bikin kamu bisa membaca traffic dengan cerdas, bukan cuma klak-klik tool.
- Aplikasi SPA/API-driven punya permukaan serangan yang lebih "tersembunyi" — proxy intersep jadi krusial untuk melihatnya.
- Burp Suite/ZAP dengan fitur Proxy History, Repeater, dan Intruder akan jadi tool andalanmu sepanjang buku ini.
- Fingerprinting (Wappalyzer, WhatWeb) membantu kamu kenali teknologi sebelum menyerang lebih jauh.
- Content discovery (gobuster/ffuf/feroxbuster + SecLists) menemukan yang tidak terlihat dari browsing biasa.
- REST dan GraphQL API punya pola enumerasi berbeda — kenali keduanya.

## Checkpoint Mandiri

1. Bisakah kamu menjelaskan kenapa HTTP disebut *stateless*, dan bagaimana cookie/session mengatasi keterbatasan ini?
2. Kenapa aplikasi SPA butuh pendekatan pengujian yang berbeda dibanding aplikasi server-rendered klasik?
3. Apa risiko keamanan dari fitur introspection GraphQL yang masih aktif di lingkungan produksi?

## Tantangan Mastery

Lakukan pemetaan permukaan serangan **selengkap mungkin** terhadap target lab (lihat modul praktikum) — mencakup semua endpoint, parameter, dan teknologi yang terdeteksi — **tanpa melihat kode sumber aplikasinya terlebih dahulu**. Gunakan kombinasi proxy intersep (untuk menangkap traffic natural saat kamu browsing), fingerprinting, dan content discovery. Simpan hasilnya sebagai baseline yang akan kamu pakai terus di Bab 6 dan 7.

## Sumber Bacaan Lanjutan

- PortSwigger Web Security Academy (gratis, sangat direkomendasikan): https://portswigger.net/web-security
- OWASP Web Security Testing Guide, bagian "Information Gathering": https://owasp.org/www-project-web-security-testing-guide/
- Dokumentasi resmi ffuf: https://github.com/ffuf/ffuf
- SecLists repository: https://github.com/danielmiessler/SecLists

## Readme dan Isi Paket

Isi Paket

| File | Fungsi |
|---|---|
| `docker-compose.yml` | Menjalankan OWASP Juice Shop + OWASP WebGoat (+ WebWolf) |
| `Vagrantfile` | Provisioning VM Kali attacker |
| `provision.sh` | Instalasi ffuf, gobuster, feroxbuster, whatweb, SecLists, Burp Suite, ZAP, Docker |
| `content_discovery.sh` | Automasi content discovery dengan ffuf (umum + pola API) |
| `fingerprint.sh` | Automasi fingerprinting (WhatWeb, header, cek spesifikasi API bocor) |
| `template_peta_serangan.md` | Template dokumentasi peta permukaan serangan |

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
- WebGoat: `http://localhost:8082/WebGoat` (jika path ini tidak berfungsi, coba `http://localhost:8082` langsung — tergantung versi image, WebGoat kadang menyajikan halaman awal di root)
- WebWolf (companion WebGoat, dipakai di beberapa modul WebGoat lanjutan): `http://localhost:9090/WebWolf`

## Langkah 3 — Setup Proxy Intersep

1. Buka Burp Suite Community (`burpsuite` dari terminal atau menu aplikasi Kali).
2. Pastikan Proxy listener aktif di `127.0.0.1:8080`.
3. Set proxy manual Firefox ke `127.0.0.1:8080`.
4. Akses `http://burp` sambil proxy aktif untuk install sertifikat CA (penting untuk traffic HTTPS).

Kalau ingin pakai OWASP ZAP sebagai alternatif:
```bash
zaproxy
```
Ikuti quick-start wizard bawaan ZAP untuk konfigurasi serupa.

## Langkah 4 — Jalankan Automasi

```bash
chmod +x content_discovery.sh fingerprint.sh
./fingerprint.sh http://localhost:3000
./content_discovery.sh http://localhost:3000
```

## Langkah 5 — Dokumentasikan

```bash
cp template_peta_serangan.md ~/Lab-Notes/Bab-05-Web-Recon/02-peta-serangan.md
```

Lengkapi berdasarkan gabungan hasil observasi manual (Proxy History) dan hasil automasi.

## Catatan Keamanan Lab

- Semua target berjalan lokal di jaringan lab — jangan expose port 3000/8082/9090 ke internet.
- Matikan container setelah selesai:
  ```bash
  docker compose down
  ```

## Step by Step & Link

Saatnya kita pindah fokus dari jaringan/service level (Bab 3-4) ke level aplikasi web. Target kita kali ini dua aplikasi latihan yang sangat populer di komunitas: **OWASP Juice Shop** dan **OWASP WebGoat** — keduanya sengaja dibuat penuh kerentanan modern, dan sudah dipelihara komunitas OWASP secara resmi.

## 5.1 Menjalankan Target

```bash
docker compose up -d
```

Ini akan menjalankan:
- **Juice Shop** di `http://localhost:3000`
- **WebGoat** di `http://localhost:8082/WebGoat` (WebGoat butuh registrasi user baru saat pertama akses — silakan buat akun sendiri)

## 5.2 Konfigurasi Proxy Intersep

Kali sudah dilengkapi Burp Suite Community secara default. Kalau kamu ingin memakai OWASP ZAP sebagai alternatif open-source penuh, sudah kami pasangkan juga lewat provisioning.

Langkah setup Burp Suite:
1. Buka Burp Suite, biarkan project sementara ("Temporary project").
2. Pastikan proxy listener aktif di `127.0.0.1:8080` (default).
3. Di browser (gunakan Firefox bawaan Kali), atur proxy manual ke `127.0.0.1:8080`.
4. Akses `http://burp` di browser saat proxy aktif untuk mengunduh dan install sertifikat CA Burp — ini diperlukan supaya traffic HTTPS juga bisa diintersep tanpa banyak warning sertifikat.

## 5.3 Menjelajahi Juice Shop Sambil Mengamati Traffic

Dengan proxy aktif, mulai jelajahi Juice Shop seperti pengguna biasa: lihat produk, coba fitur pencarian, coba login/register. Sambil melakukan ini, perhatikan tab **Proxy History** di Burp — kamu akan melihat setiap request yang terjadi "di belakang layar", termasuk panggilan-panggilan API yang nggak kelihatan cuma dari tampilan browser.

Catat di `01-observasi-manual.md`:
- Endpoint API apa saja yang kamu temukan (biasanya berpola `/rest/...` atau `/api/...` di Juice Shop)?
- Apakah ada informasi menarik di response header?

## 5.4 Fingerprinting Teknologi

```bash
whatweb http://localhost:3000
whatweb http://localhost:8082/WebGoat
```

Bandingkan juga dengan hasil ekstensi Wappalyzer di browser (install dulu dari toko ekstensi kalau belum ada).

## 5.5 Content Discovery dengan ffuf

```bash
ffuf -u http://localhost:3000/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o ~/Lab-Notes/Bab-05-Web-Recon/ffuf-juiceshop.json
```

Coba juga wordlist yang lebih spesifik untuk API:

```bash
ffuf -u http://localhost:3000/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -o ~/Lab-Notes/Bab-05-Web-Recon/ffuf-juiceshop-api.json
```

Perhatikan status code dan panjang response untuk membedakan hasil yang valid dari noise (kadang aplikasi mengembalikan status 200 untuk semua path yang tidak ditemukan — perhatikan pola ini dan sesuaikan filter ffuf jika perlu, misalnya dengan `-fs <ukuran-response-yang-mau-diabaikan>`).

## 5.6 Mengecek Spesifikasi API yang Mungkin Bocor

Coba akses beberapa path umum tempat spesifikasi API sering "bocor":

```bash
curl -s http://localhost:3000/api-docs | head -50
curl -s http://localhost:3000/swagger.json | head -50
```

## 5.7 Menyusun Peta Permukaan Serangan

Sekarang gabungkan semua temuan dari langkah 5.3 sampai 5.6 ke dalam `template_peta_serangan.md`. Ini akan jadi dokumen acuan utama kamu di Bab 6 dan Bab 7 nanti — usahakan selengkap mungkin.

## Checkpoint Praktikum

- [ ] Juice Shop dan WebGoat berhasil dijalankan dan diakses.
- [ ] Proxy intersep (Burp/ZAP) berhasil dikonfigurasi, termasuk sertifikat CA untuk HTTPS.
- [ ] Minimal 5 endpoint API tercatat dari hasil observasi manual di Proxy History.
- [ ] Fingerprinting teknologi sudah dilakukan (whatweb + Wappalyzer).
- [ ] Content discovery dengan ffuf sudah dijalankan terhadap Juice Shop, dengan hasil tersimpan.
- [ ] Peta permukaan serangan (`template_peta_serangan.md`) sudah terisi lengkap.

## Sumber Daya Pendukung

Semua file teknis — `docker-compose.yml` (Juice Shop + WebGoat), provisioning Vagrant untuk Kali dengan ffuf/gobuster/feroxbuster/whatweb/SecLists/ZAP, dan template peta serangan — ada di paket resource yang menyertai modul ini.

## Tutorial / Step-by-Step-by-st

https://youtu.be/dwXLDKlnMX8?si=_tZ6oA3gMQJ325UJ


## Template Peta Serangan

## Info Target
- **Aplikasi**: (Juice Shop / WebGoat)
- **URL Dasar**:
- **Tanggal pemetaan**:

## Fingerprinting Teknologi
| Alat | Temuan |
|---|---|
| Wappalyzer | |
| WhatWeb | |
| Header HTTP manual | |

## Daftar Endpoint Ditemukan (Manual via Proxy History)

| Method | Endpoint | Fungsi (dugaan) | Butuh Auth? | Catatan |
|---|---|---|---|---|
| | | | | |
| | | | | |
| | | | | |

## Daftar Endpoint dari Content Discovery (ffuf)

| Path | Status Code | Ukuran Response | Catatan |
|---|---|---|---|
| | | | |
| | | | |

## Spesifikasi API yang Ditemukan
- Swagger/OpenAPI bocor? (Ya/Tidak, path jika ada):
- GraphQL endpoint ditemukan? (Ya/Tidak):
- Introspection GraphQL aktif? (Ya/Tidak, jika relevan):

## Ringkasan
> Dari seluruh peta di atas, area mana yang paling mencurigakan untuk didalami lebih lanjut di Bab 6-7?
