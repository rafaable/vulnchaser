# TEXTBOOK CYBERSECURITY BAB 3

## PROGRAM SARJANA

# Reconnaissance Aktif: Penemuan Host, Port, dan Enumerasi Layanan

**Dari candidate asset menuju service register yang terukur dan dapat diaudit**

**Hatma Suryotrisongko**

*Materi latihan hanya untuk lingkungan milik sendiri atau yang memiliki izin tertulis.*

## Isi Bab

Bab ini bergerak dari scan plan, fondasi protokol, host discovery, port scanning, enumerasi layanan, kerangka KENDALI, hingga handoff dan perspektif defender.

* 3.1 Kedudukan Reconnaissance Aktif

* 3.2 Dari Handoff Pasif menuju Scan Plan

* 3.3 Fondasi TCP/IP untuk Membaca Respons

* 3.4 Port, Layanan, dan Banner

* 3.5 Host Discovery

* 3.6 Port Scanning dengan Nmap

* 3.7 Service Detection dan OS Fingerprinting

* 3.8 Enumerasi DNS Aktif

* 3.9 Enumerasi SMB

* 3.10 Enumerasi SMTP

* 3.11 Enumerasi SNMP

* 3.12 UDP Scanning yang Bertanggung Jawab

* 3.13 Nmap Scripting Engine: Kategori dan Risiko

* 3.14 Enumerasi Web dan Virtual Host

* 3.15 Living off the Land dan Validasi Manual

* 3.16 Timing, Deteksi, dan Pengendalian Dampak

* 3.17 Kerangka KENDALI untuk Reconnaissance Aktif

* 3.18 Studi Kasus Terbimbing

* 3.19 Perspektif Defender

* 3.20 Praktik Terbimbing

> *Catatan akademik. Contoh pada bab ini menggunakan aplikasi fiktif atau laboratorium vulnerable-by-design. Pembaca dilarang mempraktikkan teknik pada sistem publik tanpa otorisasi eksplisit.*

## Pendahuluan

Reconnaissance aktif dimulai ketika penguji mengirimkan lalu lintas secara langsung kepada sistem, jaringan, layanan, atau infrastruktur target. Berbeda dengan reconnaissance pasif pada Bab 2, aktivitas ini dapat menghasilkan entri pada firewall, sensor jaringan, DNS, aplikasi, endpoint detection and response, maupun sistem korelasi keamanan. Karena itu, keberhasilan tahap aktif tidak diukur dari banyaknya paket yang dikirim, melainkan dari kemampuan memperoleh informasi yang relevan dengan jejak, dampak, dan ketidakpastian yang terkendali.

MITRE ATT&CK mendefinisikan active scanning sebagai probing terhadap infrastruktur korban melalui lalu lintas jaringan. Subtekniknya mencakup pemindaian blok IP, pemindaian kerentanan, dan pemindaian berbasis wordlist \[4\]. Dalam konteks penetration testing profesional, teknik yang serupa digunakan secara sah karena didasarkan pada otorisasi, ruang lingkup, aturan pelaksanaan, batas laju, serta mekanisme penghentian yang disepakati.

Materi bab ini mengikuti cakupan kompetensi *Active Information Gathering* dalam PEN-200: pemindaian port menggunakan Netcat dan Nmap, serta enumerasi DNS, SMB, SMTP, dan SNMP. Pembahasan diperkaya dengan standar Internet, dokumentasi resmi Nmap, NIST SP 800-115, MITRE ATT&CK, OWASP WSTG, serta dokumentasi vendor dan protokol \[1\]–\[3\].

> **Prinsip keselamatan.** Seluruh perintah pada bab ini hanya boleh dijalankan pada laboratorium milik sendiri atau aset yang secara eksplisit tercantum dalam otorisasi tertulis. Gunakan placeholder seperti `<LAB-IP>` dan `<LAB-DOMAIN>`; jangan menggantinya dengan sistem publik tanpa izin.

### Capaian Pembelajaran Bab

Setelah mempelajari bab ini, mahasiswa diharapkan mampu:

 1. menjelaskan perbedaan reconnaissance pasif, reconnaissance aktif, dan vulnerability scanning;

 2. menerangkan hubungan TCP, UDP, port, layanan, dan respons jaringan dengan hasil pemindaian;

 3. merancang scan plan berdasarkan scope, tujuan informasi, anggaran paket, dan stop conditions;

 4. melakukan host discovery serta pemindaian TCP dan UDP secara bertahap pada laboratorium berizin;

 5. menafsirkan status `open`, `closed`, `filtered`, `unfiltered`, `open|filtered`, dan `closed|filtered` secara tepat;

 6. melakukan enumerasi dasar DNS, SMB, SMTP, SNMP, dan layanan web tanpa langsung beralih ke eksploitasi;

 7. memilih NSE script berdasarkan kategori dan potensi dampaknya;

 8. merekam perintah, waktu, sumber, alasan status, dan artefak keluaran sebagai bukti yang dapat diaudit;

 9. menilai ketidakpastian, kontradiksi, serta kebutuhan validasi ulang; dan

10. menjelaskan bagaimana defender mendeteksi dan mengurangi paparan yang ditemukan pada tahap aktif.

### Peta Konsep

Reconnaissance aktif membentuk alur penalaran berikut:

$$
\text{candidate asset register} \rightarrow \text{konfirmasi scope} \rightarrow \text{host discovery} \rightarrow \text{port state} \rightarrow \text{service identity} \rightarrow \text{enumerasi protokol} \rightarrow \text{hipotesis} \rightarrow \text{validasi} \rightarrow \text{evidence package}
$$

Hasil Bab 2 menjadi masukan, bukan izin otomatis. Sebuah nama host yang memiliki confidence tinggi tetap harus dikonfirmasi status scope-nya sebelum dipindai. Sebaliknya, aset yang sudah *in-scope* belum tentu boleh menerima seluruh jenis probe; *Rules of Engagement* dapat membatasi waktu, laju, port, protokol, sumber pemindaian, atau penggunaan script tertentu.

## 3.1 Kedudukan Reconnaissance Aktif

### 3.1.1 Definisi Operasional

Reconnaissance aktif adalah pengumpulan informasi melalui interaksi langsung dengan target. Aktivitasnya dapat berupa permintaan DNS ke server target, percobaan koneksi pada port tertentu, pengiriman probe identifikasi layanan, permintaan daftar share SMB, atau kueri SNMP. Target dapat merespons, menolak, membatasi, atau sama sekali tidak memberikan respons.

Reconnaissance aktif berbeda dari dua kegiatan yang sering tercampur:

| Kegiatan | Pertanyaan Utama | Contoh | Keluaran | 
| ----- | ----- | ----- | ----- | 
| **Reconnaissance pasif** | Apa yang diamati pihak ketiga? | CT, passive DNS, RDAP, scan database | Kandidat aset dan hipotesis | 
| **Reconnaissance aktif** | Apa yang merespons ketika ditanya langsung? | Host discovery, port scan, banner probe | Status teramati dan identitas layanan | 
| **Vulnerability scanning** | Apakah konfigurasi atau versi selaras dengan pola kelemahan? | Scanner kerentanan dan pemeriksaan khusus | Indikasi kerentanan yang perlu verifikasi | 

Port terbuka bukan kerentanan. Versi lama juga belum otomatis membuktikan kerentanan karena vendor dapat melakukan backport, banner dapat disamarkan, atau produk dapat diproteksi oleh kontrol kompensasi. Reconnaissance menghasilkan fakta dasar dan hipotesis; pembuktian kerentanan memerlukan tahap tersendiri.

### 3.1.2 Otorisasi sebagai Prasyarat Teknis

NIST SP 800-115 menempatkan perencanaan dan *Rules of Engagement* sebagai bagian esensial pengujian keamanan \[3\]. Sebelum mengirim paket, penguji sekurang-kurangnya memastikan:

1. target IP, domain, dan jaringan benar-benar *in-scope*;

2. alamat sumber pemindaian telah disetujui atau di-*allowlist* bila diperlukan;

3. jendela waktu, zona waktu, dan kontak eskalasi telah ditetapkan;

4. batas laju dan maksimum paralelisme sesuai kapasitas target;

5. sistem kritis, perangkat keselamatan, dan pihak ketiga telah dikecualikan jika perlu;

6. data yang boleh dikumpulkan serta masa retensinya telah ditetapkan;

7. teknik yang dilarang, misalnya autentikasi, brute force credential, atau NSE kategori *intrusive*, telah dicatat; dan

8. *stop conditions* dapat dijalankan tanpa menunggu persetujuan berjenjang.

> **Stop condition minimum.** Hentikan pemindaian apabila terjadi degradasi layanan, respons target berubah tidak wajar, volume alarm melebihi kesepakatan, aset ternyata milik pihak ketiga, ditemukan data rahasia, atau tim pemilik sistem meminta penghentian.

## 3.2 Dari Handoff Pasif menuju Scan Plan

Handoff Bab 2 idealnya memuat kandidat aset, provenance, confidence, kontradiksi, serta status scope. Tim aktif mengubahnya menjadi scan plan yang terukur. Scan plan mencegah kebiasaan menjalankan `nmap -A` terhadap seluruh jaringan tanpa pertanyaan yang jelas.

### 3.2.1 Komponen Scan Plan

| Komponen | Pertanyaan Pengendali | Contoh Keputusan Laboratorium | 
| ----- | ----- | ----- | 
| **Tujuan informasi** | Keputusan apa yang membutuhkan data? | Memastikan layanan TCP yang benar-benar mendengarkan | 
| **Target** | Host dan port mana yang diizinkan? | Satu VM `<LAB-IP>`, TCP $1-65535$, UDP top 20 | 
| **Metode** | Probe minimum apa yang cukup? | Discovery $\rightarrow$ SYN scan $\rightarrow$ `-sV` hanya pada port terbuka | 
| **Laju** | Berapa paket atau paralelisme maksimum? | Konservatif; dinaikkan setelah respons stabil | 
| **Waktu** | Kapan scan boleh berjalan? | Jendela praktikum 60 menit | 
| **Bukti** | Apa yang harus direkam? | Perintah, timestamp, versi alat, `-oA`, `--reason` | 
| **Stop condition** | Kapan harus berhenti? | Packet loss meningkat atau service health menurun | 

### 3.2.2 Strategi Bertahap

Strategi bertahap lebih aman dan lebih mudah ditafsirkan:

1. validasi alamat serta rute lokal;

2. lakukan host discovery jika diizinkan;

3. pindai port prioritas dengan laju konservatif;

4. perluas ke seluruh rentang hanya bila diperlukan;

5. jalankan version detection pada port yang relevan;

6. lakukan enumerasi protokol secara selektif;

7. korelasikan dengan hasil pasif; dan

8. dokumentasikan pertanyaan yang belum terjawab.

Urutan tersebut mengurangi noise. Jika kebutuhan informasi sudah terjawab pada langkah ketiga, penguji tidak harus menjalankan semua langkah berikutnya.

## 3.3 Fondasi TCP/IP untuk Membaca Respons

### 3.3.1 TCP Three-Way Handshake

TCP adalah protokol transport berorientasi koneksi. RFC 9293 mendefinisikan mekanisme sinkronisasi nomor urut melalui pertukaran SYN, SYN-ACK, dan ACK \[5\]. Secara ringkas:

1. klien mengirim segmen dengan flag SYN;

2. server yang mendengarkan mengirim SYN-ACK; dan

3. klien mengirim ACK sehingga koneksi masuk ke keadaan *established*.

Jika tidak ada proses yang mendengarkan pada port, sistem umumnya mengirim RST. Jika firewall membuang probe atau respons, pemindai mungkin tidak menerima jawaban. Tiga kemungkinan ini menjadi dasar klasifikasi `open`, `closed`, dan `filtered` pada SYN scan.

### 3.3.2 TCP Connect dan SYN Scan

TCP connect scan (`-sT`) meminta sistem operasi lokal menjalankan `connect()` dan menyelesaikan handshake. Mode ini tersedia tanpa hak raw-packet, tetapi koneksi lebih mungkin mencapai log aplikasi. 

SYN scan (`-sS`) mengirim SYN secara langsung; ketika menerima SYN-ACK, pemindai tidak menyelesaikan koneksi normal dan mengakhiri percobaan dengan RST \[7\], \[9\].

Istilah *stealth scan* untuk `-sS` harus digunakan dengan hati-hati. SYN scan dapat mengurangi jejak pada sebagian log aplikasi lama, tetapi tetap terlihat oleh firewall, IDS/IPS, sensor jaringan, dan modern endpoint telemetry. Perbedaan teknis yang dapat dipertanggungjawabkan adalah handshake penuh versus tidak penuh, bukan "terdeteksi versus tidak terdeteksi".

### 3.3.3 UDP dan Ketidakpastian Berbasis Diam

UDP tidak memiliki handshake setara TCP. Layanan yang menerima datagram tidak dikenal dapat merespons, diam, atau diblokir. Port UDP tertutup sering memicu ICMP *port unreachable*, tetapi pesan ini dapat dibatasi lajunya. Karena itu, ketiadaan respons sering menghasilkan status `open|filtered`, bukan kepastian bahwa port terbuka \[10\].

## 3.4 Port, Layanan, dan Banner

Port adalah bilangan 16-bit yang membantu sistem transport membedakan endpoint komunikasi. IANA membagi ruang port menjadi System Ports $0-1023$, User Ports $1024-49151$, dan Dynamic/Private Ports $49152-65535$ \[6\]. Registrasi port hanya menunjukkan konvensi; lalu lintas pada port 443 belum tentu HTTPS, dan layanan SSH dapat ditempatkan pada port selain 22.

Tiga istilah harus dipisahkan:

1. **port:** endpoint numerik pada TCP atau UDP;

2. **service:** protokol atau fungsi yang dilayani;

3. **product/version:** implementasi spesifik, misalnya OpenSSH atau Apache HTTP Server;

4. **banner:** string atau atribut respons yang mengungkap identitas; dan

5. **CPE:** penamaan terstruktur untuk produk, sistem operasi, dan perangkat yang kadang disediakan Nmap \[27\].

Banner dapat salah karena dikustomisasi, diproksikan, atau tidak diperbarui setelah patch. Oleh sebab itu, laporan yang tepat berbunyi "probe Nmap mengidentifikasi respons yang konsisten dengan produk X versi Y", bukan "target pasti menjalankan X versi Y dan rentan".

## 3.5 Host Discovery

Host discovery bertujuan menjawab apakah alamat tampak aktif dari posisi penguji. Nmap dapat menggunakan ARP pada jaringan lokal serta kombinasi ICMP dan TCP pada jaringan lain. Host yang tidak merespons belum tentu mati; firewall dapat memblokir probe, rute dapat asimetris, atau layanan hanya menerima paket tertentu.

Contoh pada subnet laboratorium:

```bash
sudo nmap -sn 192.0.2.0/29 --reason -oA evidence/01-host-discovery
```

Opsi `-sn` menonaktifkan port scan dan hanya melakukan discovery. `--reason` membantu merekam alasan Nmap menetapkan keadaan host. `-oA` menyimpan keluaran normal, XML, dan grepable sehingga hasil dapat dibaca manusia sekaligus diproses ulang \[24\].

Jika target diketahui aktif tetapi tidak menanggapi mekanisme discovery, `-Pn` membuat Nmap memperlakukan target sebagai aktif dan melanjutkan pemindaian. Opsi ini bukan teknik bypass; konsekuensinya, Nmap akan mengirim probe port kepada setiap alamat yang disebutkan. Gunakan hanya pada ruang alamat kecil dan telah dikonfirmasi.

### 3.5.1 Validasi Posisi Penguji

Sebelum menyimpulkan target tidak aktif, catat:

1. interface dan alamat sumber;

2. tabel rute dan gateway;

3. apakah target satu segmen Layer 2;

4. VPN atau jump host yang digunakan;

5. packet loss dan round-trip time; dan

6. kontrol jaringan yang diketahui.

Kesalahan rute, DNS, atau interface sering lebih menjelaskan "host down" daripada kondisi target.

## 3.6 Port Scanning dengan Nmap

### 3.6.1 Enam Status Port

Nmap menggunakan enam keadaan untuk mengekspresikan hasil dan ketidakpastian \[8\].

| Status | Makna Operasional | Respons Tipikal | Tindakan Berikutnya | 
| ----- | ----- | ----- | ----- | 
| `open` | Aplikasi menerima koneksi atau datagram | SYN-ACK atau respons layanan | Lakukan identifikasi layanan | 
| `closed` | Host dapat dijangkau, tidak ada listener | RST atau ICMP port unreachable | Catat; berguna sebagai pembanding | 
| `filtered` | Filter menghalangi penentuan | Diam atau ICMP prohibited | Periksa jalur dan kebijakan | 
| `unfiltered` | Port terjangkau, status open/closed belum diketahui | Respons ACK scan | Gunakan scan lain bila diizinkan | 
| `open|filtered` | Tidak dapat membedakan terbuka dan difilter | Tidak ada respons pada UDP/FIN/NULL/Xmas | Gunakan probe protokol atau `-sV` | 
| `closed|filtered` | Tidak dapat membedakan tertutup dan difilter | Hasil scan tertentu | Perlakukan sebagai tidak pasti | 

Status adalah kesimpulan berdasarkan metode, posisi jaringan, dan waktu tertentu. Ia bukan properti abadi dari port.

### 3.6.2 Pemindaian TCP Dasar

Tanpa hak raw-packet, gunakan connect scan:

```bash
nmap -sT -p 22,80,443,445 <LAB-IP> --reason -oA evidence/02-tcp-connect
```

Dengan hak yang sesuai pada laboratorium, SYN scan dapat digunakan:

```bash
sudo nmap -sS -p 22,80,443,445 <LAB-IP> --reason -oA evidence/03-tcp-syn
```

Untuk seluruh rentang TCP, gunakan `-p-`. Jalankan setelah port prioritas dan hanya bila scope serta waktu mengizinkan:

```bash
sudo nmap -sS -p- <LAB-IP> --reason --max-rate 200 -oA evidence/04-all-tcp
```

`--max-rate` pada contoh merupakan batas atas konservatif untuk laboratorium, bukan rekomendasi universal. Kapasitas perangkat embedded, layanan lama, atau koneksi WAN dapat memerlukan nilai jauh lebih rendah.

### 3.6.3 Membaca pada Level Paket

Untuk memahami satu port, gunakan `--packet-trace` pada satu target dan satu port. Dokumentasi resmi Nmap merekomendasikan fitur ini untuk melihat perilaku paket secara langsung \[26\].

```bash
sudo nmap -sS -p 80 <LAB-IP> --reason --packet-trace
```

Jangan mengaktifkan packet trace pada pemindaian besar karena keluaran menjadi sangat padat dan dapat mempersulit analisis.

## 3.7 Service Detection dan OS Fingerprinting

### 3.7.1 Service/Version Detection

Opsi `-sV` mengirim probe yang dipahami berbagai protokol, lalu mencocokkan respons dengan basis `nmap-service-probes` \[11\]. Jalankan secara selektif pada port yang sudah ditemukan:

```bash
sudo nmap -sV --version-light -p 22,80,445 <LAB-IP> --reason -oA evidence/05-service-detection
```

`--version-light` menurunkan intensitas probe. Jika identitas tetap belum jelas dan RoE mengizinkan, intensitas dapat dinaikkan secara bertahap. Catat bahwa `-sV` lebih interaktif daripada port scan murni; ia dapat memicu log aplikasi.

Validasi manual membantu memahami respons:

```bash
nc -nv <LAB-IP> 22
```

Koneksi tersebut cocok untuk laboratorium dengan banner SSH. Jangan mengirim string protokol acak ke layanan produksi karena parser lama dapat tidak stabil.

### 3.7.2 OS Fingerprinting

Opsi `-O` mengirim serangkaian probe TCP dan UDP, kemudian membandingkan karakteristik respons dengan `nmap-os-db`. Akurasi meningkat ketika terdapat sekurang-kurangnya satu port TCP terbuka dan satu tertutup \[12\].

```bash
sudo nmap -O --osscan-limit <LAB-IP> -oA evidence/06-os
```

Hasil OS detection adalah estimasi. NAT, load balancer, firewall, proxy, atau perangkat keamanan dapat mengubah ciri paket. Laporkan tingkat kecocokan dan alternatif, bukan hanya satu label sistem operasi.

## 3.8 Enumerasi DNS Aktif

DNS aktif berarti penguji mengirim kueri langsung kepada resolver atau authoritative server. RFC 1034 membedakan layanan rekursif dan nonrekursif melalui bit RD dan RA \[14\]. Sebelum enumerasi, pastikan server yang dituju memang termasuk scope; authoritative DNS dapat dikelola penyedia pihak ketiga.

### 3.8.1 Record Dasar

Kueri awal yang aman dan terarah mencakup SOA, NS, A/AAAA, MX, TXT, CNAME, SRV, serta CAA jika relevan:

```bash
dig @<DNS-IP> <LAB-DOMAIN> SOA +norecurse
dig @<DNS-IP> <LAB-DOMAIN> NS +norecurse
dig @<DNS-IP> <LAB-DOMAIN> MX +norecurse
dig @<DNS-IP> _ldap._tcp.<LAB-DOMAIN> SRV +norecurse
```

`+norecurse` membantu membedakan jawaban authoritative dari layanan rekursif. Rekam flag aa, ra, status, TTL, dan server yang menjawab.

### 3.8.2 Zone Transfer

AXFR merupakan mekanisme transfer zona untuk menjaga konsistensi antar-authoritative name server \[15\]. Pengujian salah konfigurasi dilakukan hanya terhadap server yang diizinkan:

```bash
dig AXFR @<DNS-IP> <LAB-DOMAIN>
```

Keberhasilan AXFR mengungkap isi zona yang dilayani server, tetapi tidak otomatis berarti seluruh nama aktif, internal, atau sensitif. Kegagalannya justru merupakan hasil normal pada konfigurasi yang membatasi transfer kepada secondary server yang sah.

### 3.8.3 Wordlist Enumeration dan Wildcard DNS

Wordlist scanning mencoba label seperti `www`, `mail`, `vpn`, atau `dev` dan memeriksa respons. MITRE ATT&CK mengklasifikasikannya sebagai T1595.003 \[25\]. Teknik ini aktif, menghasilkan volume kueri, dan dapat memicu proteksi DNS.

```bash
dnsrecon -d <LAB-DOMAIN> -D /path/to/authorized-small-wordlist.txt -n <DNS-IP> -t brt
```

Sebelum menafsirkan hasil, uji label acak untuk mendeteksi wildcard DNS. Jika setiap nama acak mengarah ke IP yang sama, daftar "subdomain ditemukan" dapat berisi false positive. Bandingkan record, TTL, pola jawaban, dan perilaku aplikasi pada tahap berikutnya.

## 3.9 Enumerasi SMB

SMB digunakan Windows untuk berbagi file, printer, dan sumber daya jaringan \[16\]. Penemuan port 445 hanya membuktikan adanya listener yang merespons sesuai scan; enumerasi diperlukan untuk mengetahui dialek, signing, nama host/domain, dan share yang dapat dilihat.

### 3.9.1 Urutan Enumerasi Berisiko Rendah

1. identifikasi dialek SMB dan security mode;

2. periksa apakah signing diperlukan;

3. lakukan null/guest listing hanya bila diizinkan;

4. catat share, komentar, dan akses yang diberikan server;

5. jangan membuka atau menyalin isi file tanpa kebutuhan dan izin.

Contoh laboratorium:

```bash
sudo nmap -p 445 --script smb-protocols,smb2-security-mode -oA evidence/07-smb-posture <LAB-IP>
smbclient -L //<LAB-IP>/ -N
```

Opsi `-N` tidak meminta password. Keberhasilan listing tanpa autentikasi menunjukkan paparan yang perlu dianalisis; kegagalan bukan alasan untuk mencoba credential acak. Pada Windows 11 24H2 dan Windows Server 2025, SMB signing diwajibkan secara default untuk koneksi masuk dan keluar, sehingga baseline modern dapat berbeda dari laboratorium lama \[17\].

### 3.9.2 Bukti yang Perlu Dicatat

1. dialek SMB yang dinegosiasikan;

2. status signing required/enabled;

3. konteks autentikasi: anonymous, guest, atau credential uji;

4. share yang terlihat dan hak yang benar-benar diberikan;

5. error code; dan

6. apakah server Windows, Samba, NAS, atau proxy berdasarkan bukti yang tersedia.

## 3.10 Enumerasi SMTP

SMTP mendukung perintah EHLO untuk mengumumkan kapabilitas server. RFC 5321 juga mendefinisikan VRFY dan EXPN, tetapi implementasi boleh membatasi atau menonaktifkannya agar tidak membocorkan informasi \[18\].

Contoh pemeriksaan manual terbatas:

```bash
nc -nv <LAB-IP> 25
EHLO student.lab
VRFY labuser
QUIT
```

Gunakan hanya identitas laboratorium yang disediakan pengajar. Pengujian daftar username dalam jumlah besar mengubah kegiatan menjadi account enumeration dan dapat mendukung serangan credential berikutnya. Pemeriksaan relay, pengiriman email, atau autentikasi tidak termasuk reconnaissance dasar kecuali disebut eksplisit dalam RoE.

Nmap dapat merangkum command yang ditawarkan:

```bash
sudo nmap -p 25 --script smtp-commands -oA evidence/08-smtp-commands <LAB-IP>
```

Interpretasi respons harus membedakan 250, 252, 550, timeout, dan penolakan berbasis kebijakan. Respons berbeda belum tentu membuktikan keberadaan user; server dapat sengaja menyeragamkan atau mengaburkan respons.

## 3.11 Enumerasi SNMP

SNMP digunakan untuk membaca atau mengubah objek manajemen pada perangkat dan sistem. SNMPv1/v2c menggunakan community string, sedangkan kerangka SNMPv3 mendukung model keamanan berbasis pengguna dan kontrol akses yang lebih kuat \[19\].

### 3.11.1 Read-Only Walk pada Laboratorium

Jika pengajar menyediakan community string read-only:

```bash
snmpwalk -v2c -c <AUTHORIZED-COMMUNITY> <LAB-IP> 1.3.6.1.2.1.1
```

Subtree `1.3.6.1.2.1.1` berisi kelompok sistem seperti deskripsi, nama, uptime, dan kontak, bergantung pada konfigurasi. `snmpwalk` menjalankan rangkaian GETNEXT untuk menelusuri subtree \[21\]. Batasi OID agar pengumpulan proporsional.

### 3.11.2 Batas Penting

1. Menebak community string adalah aktivitas credential guessing dan harus memperoleh izin tersendiri.

2. Jangan menjalankan `snmpset` pada reconnaissance; operasi write dapat mengubah konfigurasi atau mengganggu perangkat.

3. Jangan meletakkan password SNMPv3 sensitif pada command line produksi karena dapat muncul pada process listing atau shell history.

4. Nilai sysDescr dapat mengungkap versi, tetapi tetap memerlukan verifikasi dan penanganan sebagai data sensitif.

## 3.12 UDP Scanning yang Bertanggung Jawab

Pemindaian UDP lambat karena banyak port terbuka dan terfilter sama-sama diam, sementara ICMP *unreachable* dapat dibatasi \[10\]. Strategi yang lebih aman adalah memulai dari port relevan atau top ports:

```bash
sudo nmap --top-ports 20 <LAB-IP> --reason -sU --max-rate 50 -oA evidence/09-udp-top20
```

Kemudian, gunakan `-sV` pada kandidat `open|filtered` yang relevan. Version detection dapat mengirim payload protokol dan mengubah status menjadi `open` bila memperoleh respons \[11\].

Jangan mengubah semua `open|filtered` menjadi "terbuka" dalam laporan. Pernyataan yang benar adalah: "Nmap tidak dapat membedakan port terbuka dan difilter pada metode serta posisi pengujian ini."

## 3.13 Nmap Scripting Engine: Kategori dan Risiko

NSE memperluas Nmap melalui script Lua. Script dapat menjalankan discovery sederhana, version detection, autentikasi, brute force, pemeriksaan kerentanan, bahkan tindakan *intrusive*. Nama kategori bukan jaminan mutlak; baca dokumentasi dan kode script sebelum digunakan \[13\].

| Kategori/Jenis | Tujuan Umum | Sikap Penggunaan | 
| ----- | ----- | ----- | 
| `safe` | Dirancang agar tidak mengeksploitasi atau mengganggu | Tetap verifikasi dokumentasi dan target | 
| `discovery` | Memperoleh informasi tambahan | Periksa volume dan data yang dikumpulkan | 
| `version` | Mendukung identifikasi layanan | Jalankan pada port terpilih | 
| `auth` | Berkaitan dengan autentikasi | Memerlukan credential dan izin jelas | 
| `brute` | Mencoba banyak credential/identifier | Di luar recon dasar kecuali eksplisit | 
| `vuln` | Mendeteksi kondisi kerentanan | Tidak selalu aman; verifikasi per script | 
| `intrusive`, `dos`, `exploit` | Berpotensi mengubah atau mengganggu | Jangan digunakan tanpa otorisasi khusus | 

Gunakan `--script-help` sebelum eksekusi:

```bash
nmap --script-help smb2-security-mode
sudo nmap -p 445 --script smb2-security-mode <LAB-IP>
```

Jangan menjalankan `--script vuln` terhadap seluruh jaringan sebagai kebiasaan. Pilih script berdasarkan pertanyaan informasi, versi layanan, serta potensi dampak.

## 3.14 Enumerasi Web dan Virtual Host

OWASP WSTG menempatkan identifikasi attack surface, web server fingerprinting, framework fingerprinting, dan pemetaan arsitektur sebagai bagian information gathering \[22\], \[23\]. Setelah port HTTP(S) ditemukan, penguji dapat memeriksa:

1. kode status, header, redirect, dan cookie;

2. sertifikat TLS dan nama alternatif;

3. perbedaan respons berdasarkan Host header;

4. halaman default, error page, dan path yang telah disetujui;

5. teknologi frontend/backend berdasarkan beberapa indikator; dan

6. kemungkinan reverse proxy, load balancer, CDN, atau WAF.

Permintaan header langsung bersifat aktif:

```bash
curl -k -sS -D evidence/web-headers.txt -o /dev/null https://<LAB-IP>/
```

`-k` hanya sesuai untuk laboratorium dengan sertifikat tidak tepercaya; pada assessment nyata, kegagalan validasi sertifikat justru merupakan bukti dan tidak seharusnya disembunyikan. Virtual-host wordlist scanning dapat menghasilkan banyak request. Gunakan wordlist kecil yang diturunkan dari reconnaissance pasif, tetapkan baseline wildcard, serta batasi laju.

## 3.15 Living off the Land dan Validasi Manual

PEN-200 memasukkan *Living off the Land* pada cakupan active information gathering \[1\]. Dalam konteks ini, istilah tersebut berarti memakai utilitas yang sudah tersedia ketika instalasi alat tambahan tidak mungkin atau tidak diizinkan. Tujuannya bukan menyembunyikan aktivitas, melainkan memperoleh bukti minimum dengan alat bawaan.

Contoh pada Windows:

```powershell
Test-NetConnection -ComputerName <LAB-IP> -Port 443 -InformationLevel Detailed
Resolve-DnsName <LAB-DOMAIN> -Type A -Server <DNS-IP>
```

Contoh pada sistem Unix-like:

```bash
nc -zv <LAB-IP> 443
dig @<DNS-IP> <LAB-DOMAIN> A +norecurse
```

Keterbatasannya harus dicatat: output mungkin tidak menyediakan status yang sama dengan Nmap, tidak memiliki retry adaptif, atau hanya menguji satu port. Tool availability juga bergantung pada versi sistem. Validasi manual berfungsi menjelaskan hasil, bukan menggandakan seluruh scan.

## 3.16 Timing, Deteksi, dan Pengendalian Dampak

Nmap menyediakan template `-T0` hingga `-T5`, dari sangat lambat hingga sangat agresif \[28\]. Template memengaruhi timeout, paralelisme, dan penundaan. `-T0` atau `-T1` dapat mengurangi laju, tetapi bukan jaminan menghindari IDS; dokumentasi Nmap sendiri menekankan waktu pemindaian yang sangat panjang \[29\].

Untuk pengujian profesional, tujuan utama bukan "tidak terdeteksi", melainkan dapat diprediksi dan dapat dibedakan dari serangan nyata. Praktik yang lebih baik meliputi:

1. koordinasikan source IP dan jendela waktu dengan SOC;

2. gunakan `--max-rate`, host group kecil, dan target terpilih;

3. mulai dari probe minimum lalu naikkan bertahap;

4. pantau packet loss, latency, error aplikasi, dan health dashboard;

5. jangan memakai fragmentasi, decoy, spoofing, atau teknik evasion kecuali terdapat tujuan uji dan otorisasi khusus;

6. kirim pemberitahuan bila scan dihentikan; dan

7. simpan scan ID agar SOC dapat mengorelasikan alarm.

### Telemetri yang Mungkin Terpicu

| Sumber Deteksi | Pola yang Dapat Terlihat | 
| ----- | ----- | 
| **Firewall / NetFlow** | Satu sumber mengakses banyak port atau host | 
| **IDS / IPS** | Urutan SYN, signature Nmap, probe protokol anomali | 
| **DNS log** | Label tidak ada atau kueri AXFR | 
| **Web / WAF** | Variasi Host, user-agent, path, atau kode status banyak | 
| **Endpoint / service log** | Koneksi singkat, handshake gagal, banner probe | 
| **SIEM** | Korelasi lintas sensor dan pengulangan berbasis waktu | 

Perspektif ini penting: scan yang "berhasil" bagi pentester sekaligus menjadi latihan validasi monitoring bagi defender.

## 3.17 Kerangka KENDALI untuk Reconnaissance Aktif

Bab ini menggunakan kerangka orisinal KENDALI sebagai checklist operasional:

1. **K**onfirmasi kewenangan – cocokkan target, sumber, waktu, dan teknik dengan RoE.

2. **E**ksekusi bertahap – mulai dari probe minimum, lalu perluas berdasarkan kebutuhan.

3. **N**ilai respons – tafsirkan paket, error, dan ketiadaan respons bersama metode yang digunakan.

4. **D**okumentasikan provenance – rekam perintah, versi alat, waktu, posisi, dan output asli.

5. **A**tur laju dan dampak – gunakan rate limit, pemantauan kesehatan, dan stop conditions.

6. **L**akukan validasi silang – bandingkan Nmap, alat protokol, dan sumber pasif.

7. **I**ntegrasikan hasil – ubah data menjadi service register, hipotesis, dan rencana tahap berikutnya.

KENDALI melanjutkan SARING pada Bab 2. SARING menghasilkan kandidat beralasan; KENDALI menguji kandidat tersebut secara langsung tanpa kehilangan batas kewenangan dan provenance.

### 3.17.1 Evidence Record Minimum

Setiap observasi sebaiknya memuat:

| Field | Contoh | 
| ----- | ----- | 
| `target` | `<LAB-IP>:445/tcp` | 
| `scope_status` | *In-scope* | 
| `source_ip` | Alamat workstation atau scanner | 
| `command` | Perintah lengkap yang dijalankan | 
| `tool_version` | Nmap dan OS penguji | 
| `started_at / ended_at` | Timestamp dengan zona waktu | 
| `observation` | `open`, respons SYN-ACK | 
| `reason` | Output `--reason` dan artefak terkait | 
| `inference` | Listener SMB kemungkinan tersedia | 
| `confidence` | Tinggi untuk port state; sedang untuk produk | 
| `limitations` | NAT/WAF/proxy mungkin memengaruhi respons | 
| `next_step` | Enumerasi dialek/signing yang diizinkan | 

Output mentah tidak boleh diedit. Analisis, normalisasi, dan ringkasan disimpan terpisah agar integritas bukti tetap terjaga.

## 3.18 Studi Kasus Terbimbing: Laboratorium Kampus Fiktif

### 3.18.1 Skenario

Tim penguji menerima satu kandidat dari Bab 2: `portal.lab.example` mengarah ke `<LAB-IP>`. Pemilik sistem mengonfirmasi target *in-scope*. RoE mengizinkan TCP full-port, UDP top 20, DNS read-only, SMB anonymous listing, SMTP command discovery, dan SNMP read-only dengan community khusus. Brute force, vulnerability script, eksploitasi, dan operasi write dilarang.

### 3.18.2 Urutan Pengujian

1. Host discovery menunjukkan target aktif berdasarkan TCP response.

2. SYN scan port prioritas menemukan 22, 25, 53, 80, dan $445/\text{tcp}$.

3. Full-port scan menemukan tambahan $8443/\text{tcp}$.

4. Service detection mengidentifikasi respons yang konsisten dengan SSH, SMTP, DNS, HTTP, dan SMB.

5. UDP top 20 menghasilkan $53/\text{udp}$ `open` dan $161/\text{udp}$ `open|filtered`.

6. Probe SNMP dengan community yang disediakan menghasilkan sysDescr; status $161/\text{udp}$ dinaikkan menjadi `open`.

7. AXFR ditolak, yang merupakan konfigurasi yang diharapkan.

8. SMB menunjukkan *signing enabled* tetapi tidak *required* serta satu share *guest-readable*.

9. SMTP EHLO mengungkap kapabilitas; VRFY `labuser` mengembalikan 252, sehingga keberadaan user tidak dapat dipastikan.

### 3.18.3 Mengubah Observasi menjadi Hipotesis

| Observasi | Interpretasi yang Tepat | Yang Belum Terbukti | 
| ----- | ----- | ----- | 
| `$445/\text{tcp}$ open` | Terdapat listener yang merespons SYN | Share dapat diakses atau SMB rentan | 
| *Signing enabled, not required* | Server dapat menegosiasikan koneksi tanpa signing pada kondisi tertentu | Relay pasti dapat dieksploitasi | 
| *Share guest-readable* | Metadata share dapat diakses sebagai guest | Seluruh file sensitif atau writable | 
| `$161/\text{udp}$ merespons SNMP` | Agent SNMP aktif dengan credential uji | Community default atau write access | 
| VRFY memberi 252 | Server tidak mengonfirmasi user | User valid atau tidak valid | 
| AXFR ditolak | Transfer zona tidak diberikan kepada sumber uji | Seluruh DNS aman dari salah konfigurasi lain | 

### 3.18.4 Ringkasan Handoff

Target merespons pada enam layanan TCP dan dua layanan UDP terkonfirmasi. Bukti terkuat untuk tindak lanjut adalah share SMB *guest-readable* dan SMB signing yang tidak diwajibkan; keduanya memerlukan validasi dampak terpisah sesuai RoE. SNMP hanya diuji read-only dengan community yang disediakan. Tidak ada brute force, write operation, vulnerability script, atau eksploitasi. Seluruh output mentah, perintah, timestamp, dan alasan status tersedia pada evidence package.

## 3.19 Perspektif Defender

Reconnaissance aktif memberikan daftar kontrol yang dapat diuji tim pertahanan:

1. **Kelola inventaris eksternal.** Bandingkan hasil scan resmi dengan CMDB, firewall policy, dan cloud inventory.

2. **Kurangi service exposure.** Tutup listener yang tidak diperlukan dan batasi management protocol pada jaringan administrasi.

3. **Perkuat DNS.** Batasi AXFR, nonaktifkan *open recursion*, serta pantau NXDOMAIN burst dan kueri anomali.

4. **Perkuat SMB.** Terapkan signing sesuai baseline, hapus guest access, minimalkan SMB lintas segmen, dan blokir 445 dari Internet \[17\].

5. **Perkuat SMTP.** Seragamkan respons account enumeration, batasi VRFY/EXPN, dan monitor pola command yang tidak lazim.

6. **Modernisasi SNMP.** Prioritaskan SNMPv3, batasi source, gunakan read-only, rotasi secret, serta pantau walk besar.

7. **Korelasikan sensor.** Hubungkan firewall, DNS, IDS, WAF, endpoint, dan service log.

8. **Uji deteksi.** Cocokkan scan ID dengan alarm yang muncul dan perbaiki aturan yang melewatkan pola relevan.

Reconnaissance resmi dapat menjadi *purple-team micro-exercise*: pentester menghasilkan stimulus yang terkontrol, sementara SOC membuktikan visibilitas dan waktu deteksinya.

## 3.20 Praktik Terbimbing

### 3.20.1 Lingkungan

Gunakan jaringan host-only yang tidak dirutekan ke Internet dengan dua VM:

1. satu workstation Kali Linux; dan

2. satu target vulnerable-by-design, misalnya Metasploitable2, atau image laboratorium lain yang disetujui pengajar.

Ambil snapshot sebelum praktikum. Pastikan tidak ada bridge ke jaringan kampus atau produksi. Catat IP, interface, rute, versi Nmap, waktu mulai, dan identitas mahasiswa.

### 3.20.2 Tugas

 1. Susun scan plan satu halaman menggunakan KENDALI.

 2. Jalankan host discovery dan jelaskan alasan status host.

 3. Bandingkan connect scan dengan SYN scan pada empat port.

 4. Gunakan `--packet-trace` pada satu port terbuka dan satu tertutup.

 5. Lakukan full TCP scan dengan rate limit yang disetujui.

 6. Jalankan service detection hanya pada port terbuka.

 7. Lakukan UDP top 20 dan validasi dua kandidat dengan probe protokol.

 8. Lakukan enumerasi DNS, SMB, SMTP, atau SNMP sesuai layanan yang tersedia.

 9. Susun service register dengan fakta, inferensi, confidence, dan keterbatasan.

10. Buat ringkasan defender: log apa yang seharusnya muncul dan kontrol apa yang perlu diperiksa.

### 3.20.3 Rubrik Penilaian

| Aspek | Bobot | Kriteria | 
| ----- | ----- | ----- | 
| **Scope dan keselamatan** | 20% | Target benar, rate limit, stop conditions, tanpa teknik terlarang | 
| **Penalaran protokol** | 20% | Mampu menjelaskan respons TCP/UDP dan status Nmap | 
| **Metode bertahap** | 15% | Probe minimum, eskalasi beralasan, tidak menjalankan opsi agresif membabi buta | 
| **Bukti dan provenance** | 20% | Command, waktu, versi, output asli, reason, dan struktur direktori | 
| **Analisis** | 15% | Fakta dipisahkan dari inferensi dan ketidakpastian dinyatakan | 
| **Handoff dan defender view** | 10% | Prioritas validasi serta kontrol deteksi/remediasi jelas | 

### 3.20.4 Quality Gate

Sebelum dikumpulkan, pastikan:

 1. tidak ada IP publik atau aset produksi;

 2. tidak ada password, token, atau community string pada laporan;

 3. output mentah tidak diubah;

 4. setiap kesimpulan menunjuk bukti;

 5. `open|filtered` tidak ditulis sebagai `open` tanpa validasi;

 6. nama produk/version tidak dianggap sebagai bukti kerentanan;

 7. perbedaan hasil antarscan dijelaskan; dan

 8. seluruh VM dikembalikan ke snapshot bila praktikum mengubah keadaan.

## Rangkuman Bab

 1. Reconnaissance aktif berinteraksi langsung dengan target dan dapat dideteksi.

 2. Hasil pasif harus dikonfirmasi status scope-nya sebelum menjadi target aktif.

 3. TCP connect menyelesaikan handshake, sedangkan SYN scan tidak; keduanya tetap dapat terlihat oleh sensor.

 4. Port terdaftar hanya konvensi. Identitas layanan harus ditentukan dari respons dan divalidasi.

 5. Status Nmap menyatakan kesimpulan berdasarkan metode dan posisi; `filtered` atau `open|filtered` mengandung ketidakpastian.

 6. UDP membutuhkan kesabaran, probe protokol, dan bahasa laporan yang hati-hati.

 7. DNS, SMB, SMTP, dan SNMP memiliki jalur enumerasi serta batas dampak yang berbeda.

 8. NSE dipilih per script; kategori `vuln` atau `intrusive` tidak boleh dijalankan tanpa telaah dan izin.

 9. Timing lambat bukan jaminan tidak terdeteksi. Tujuan profesional adalah scan yang terkendali, dapat dikorelasikan, dan dapat dihentikan.

10. KENDALI menghubungkan kewenangan, eksekusi bertahap, interpretasi, bukti, dampak, validasi silang, dan handoff.

## Uji Pemahaman

### Pilihan Ganda

 1. Pernyataan paling tepat tentang SYN scan adalah:

    * A. Sama sekali tidak dapat dideteksi

    * **B. Tidak menyelesaikan handshake normal, tetapi tetap menghasilkan lalu lintas yang dapat dicatat**

    * C. Selalu lebih akurat daripada semua scan lain

    * D. Tidak membutuhkan otorisasi

 2. Respons SYN-ACK terhadap SYN scan umumnya menunjukkan:

    * **A. Port TCP terbuka**

    * B. Port UDP terbuka

    * C. Host pasti rentan

    * D. Firewall tidak tersedia

 3. Status `filtered` berarti:

    * A. Aplikasi pasti tidak berjalan

    * **B. Filter menghalangi Nmap menentukan port secara langsung**

    * C. Port terbuka dan sudah dieksploitasi

    * D. Target tidak pernah menerima paket

 4. Mengapa port 443 tidak membuktikan layanan HTTPS?

    * A. Port tidak memiliki nomor

    * **B. Registrasi port adalah konvensi dan aplikasi dapat menggunakan port lain**

    * C. HTTPS hanya menggunakan UDP

    * D. Nmap selalu mengubah nomor port

 5. Fungsi `--reason` adalah:

    * A. Menghapus log target

    * **B. Menampilkan alasan paket yang mendasari status host atau port**

    * C. Mengeksploitasi layanan

    * D. Mengubah hasil menjadi kerentanan

 6. Status `open|filtered` pada UDP sebaiknya ditulis sebagai:

    * A. Port pasti terbuka

    * B. Port pasti tertutup

    * **C. Belum dapat dibedakan antara terbuka dan difilter**

    * D. Host pasti mati

 7. Tindakan yang paling tepat setelah menemukan port terbuka adalah:

    * A. Langsung menjalankan semua NSE kategori vuln

    * **B. Melakukan service detection terarah sesuai scope**

    * C. Menganggap layanan rentan

    * D. Mencoba seluruh password default

 8. Keberhasilan AXFR membuktikan:

    * **A. Server memberikan data zona kepada sumber penguji**

    * B. Seluruh host dalam zona aktif

    * C. Domain pasti dapat diambil alih

    * D. Semua record bersifat internal

 9. Community string SNMP yang diberikan untuk praktikum seharusnya:

    * A. Ditulis pada laporan publik

    * **B. Digunakan untuk read-only scope yang disetujui dan disamarkan dalam laporan**

    * C. Dicoba pada seluruh Internet

    * D. Digunakan dengan snmpset

10. Mengapa timing `-T0` bukan jaminan *stealth*?

    * **A. Paket lambat tetap dapat dicatat dan dikorelasikan**

    * B. `-T0` selalu mengirim lebih banyak exploit

    * C. IDS hanya melihat bandwidth

    * D. `-T0` menonaktifkan jaringan

11. Dalam KENDALI, huruf D berarti:

    * A. Decoy scanning

    * **B. Dokumentasikan provenance**

    * C. Disable firewall

    * D. Download data

12. Kesimpulan yang tepat dari banner produk versi lama adalah:

    * A. Kerentanan otomatis terbukti

    * **B. Banner merupakan observasi yang perlu dikorelasikan dan diverifikasi**

    * C. Eksploitasi boleh dilakukan tanpa izin

    * D. Patch tidak mungkin diterapkan

### Kunci Jawaban Pilihan Ganda

1. B | 2. A | 3. B | 4. B | 5. B | 6. C | 7. B | 8. A | 9. B | 10. A | 11. B | 12. B

## Pertanyaan Esai

1. Jelaskan perbedaan teknis dan implikasi logging antara TCP connect scan dan SYN scan.

2. Rancang scan plan untuk lima server laboratorium dengan batas maksimum 100 paket per detik.

3. Jelaskan bagaimana membedakan fakta, inferensi, dan ketidakpastian dari output Nmap.

4. Susun prosedur enumerasi DNS aktif yang memeriksa wildcard dan mengendalikan volume kueri.

5. Jelaskan bagaimana hasil active reconnaissance dapat digunakan SOC untuk menguji kemampuan deteksi.

## Glosarium

* **Active reconnaissance:** Pengumpulan informasi melalui interaksi langsung dengan target.

* **AXFR:** Mekanisme transfer seluruh data zona DNS antara server yang berwenang.

* **Banner:** Informasi identitas atau karakteristik yang diberikan layanan melalui respons protokol.

* **Closed:** Keadaan Nmap ketika port dapat dijangkau tetapi tidak terdapat aplikasi yang mendengarkan.

* **Community string:** Nilai autentikasi sederhana pada SNMPv1/v2c yang juga menentukan konteks akses.

* **Connect scan:** Pemindaian TCP yang menggunakan system call `connect()` dan menyelesaikan handshake.

* **Filtered:** Keadaan ketika filter mencegah pemindai menentukan apakah port terbuka atau tertutup.

* **Host discovery:** Proses menentukan host yang tampak aktif dari posisi penguji.

* **KENDALI:** Kerangka reconnaissance aktif: Konfirmasi, Eksekusi, Nilai, Dokumentasikan, Atur, Lakukan validasi silang, dan Integrasikan.

* **NSE:** Nmap Scripting Engine, sistem script Lua untuk memperluas fungsi Nmap.

* **Open:** Keadaan ketika aplikasi menerima koneksi atau datagram pada port yang diuji.

* **Open|filtered:** Keadaan ambigu ketika pemindai tidak dapat membedakan port terbuka dan difilter.

* **Port:** Bilangan 16-bit untuk membedakan endpoint komunikasi transport.

* **Provenance:** Asal-usul bukti, termasuk target, sumber, perintah, alat, waktu, dan kondisi akuisisi.

* **Service detection:** Pengiriman probe protokol untuk mengidentifikasi layanan, produk, dan kemungkinan versi.

* **SYN scan:** Pemindaian TCP yang menyimpulkan keadaan port tanpa menyelesaikan handshake normal.

* **Version detection:** Proses pencocokan respons layanan dengan signature produk atau versi.

* **Wildcard DNS:** Konfigurasi yang memberikan respons untuk banyak atau semua nama yang sebenarnya tidak didefinisikan secara eksplisit.

## Referensi

\[1\] OffSec. "OSCP+ Body of Knowledge - Active Information Gathering." 2026. `https://help.offsec.com/hc/en-us/articles/38543335188756-OSCP-Body-of-knowledge`

\[2\] OffSec. "PEN-200: Penetration Testing with Kali Linux Course Syllabus." 2026. `https://manage.offsec.com/app/uploads/2026/03/PEN-200_Syllabus.pdf`

\[3\] National Institute of Standards and Technology. "Technical Guide to Information Security Testing and Assessment." NIST SP 800-115. 2008. `https://csrc.nist.gov/pubs/sp/800/115/final`

\[4\] MITRE ATT&CK. "Active Scanning, T1595." `https://attack.mitre.org/techniques/T1595/`

\[5\] Eddy, W., ed. "Transmission Control Protocol (TCP)." RFC 9293. 2022. `https://www.rfc-editor.org/info/rfc9293/`

\[6\] Internet Assigned Numbers Authority. "Service Name and Transport Protocol Port Number Registry." `https://www.iana.org/assignments/service-names-port-numbers`

\[7\] Nmap Project. "Port Scanning Techniques." `https://nmap.org/book/man-port-scanning-techniques.html`

\[8\] Nmap Project. "Port Scanning Basics." `https://nmap.org/book/man-port-scanning-basics.html`

\[9\] Nmap Project. "TCP SYN (Stealth) Scan." `https://nmap.org/book/synscan.html`

\[10\] Nmap Project. "UDP Scan (-sU)." `https://nmap.org/book/scan-methods-udp-scan.html`

\[11\] Nmap Project. "Service and Version Detection." `https://nmap.org/book/man-version-detection.html`

\[12\] Nmap Project. "OS Detection." `https://nmap.org/book/man-os-detection.html`

\[13\] Nmap Project. "Nmap Scripting Engine." `https://nmap.org/book/nse.html`

\[14\] Mockapetris, P. "Domain Names Concepts and Facilities." RFC 1034. 1987. `https://www.rfc-editor.org/info/rfc1034/`

\[15\] Lewis, E. dan A. Hoenes, ed. "DNS Zone Transfer Protocol (AXFR)." RFC 5936. 2010. `https://www.rfc-editor.org/info/rfc5936/`

\[16\] Microsoft. "SMB Features in Windows and Windows Server." 2025. `https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-feature-descriptions`

\[17\] Microsoft. "SMB Security Hardening in Windows Server and Windows." 2024. `https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security-hardening`

\[18\] Klensin, J. "Simple Mail Transfer Protocol." RFC 5321. 2008. `https://www.rfc-editor.org/info/rfc5321/`

\[19\] Harrington, D., Presuhn, R., dan B. Wijnen. "An Architecture for Describing SNMP Management Frameworks." RFC 3411. 2002. `https://www.rfc-editor.org/info/rfc3411/`

\[20\] Net-SNMP Project. "SNMPv3 Options." `https://www.net-snmp.org/tutorial/tutorial-5/commands/snmpv3.html`

\[21\] Net-SNMP Project. "snmpwalk." `https://www.net-snmp.org/wiki/index.php/TUT%3Asnmpwalk`

\[22\] OWASP Foundation. "Attack Surface Identification." Web Security Testing Guide. `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/04-Attack_Surface_Identification`

\[23\] OWASP Foundation. "Fingerprint Web Application Framework." Web Security Testing Guide. `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/08-Fingerprint_Web_Application_Framework`

\[24\] Nmap Project. "Nmap Output Formats." `https://nmap.org/book/output.html`

\[25\] MITRE ATT&CK. "Wordlist Scanning, T1595.003." `https://attack.mitre.org/techniques/T1595/003/`

\[26\] Nmap Project. "TCP SYN Scan - Packet Trace." `https://nmap.org/book/synscan.html`

\[27\] Nmap Project. "Common Platform Enumeration." `https://nmap.org/book/output-formats-cpe.html`

\[28\] Nmap Project. "Command-line Flags - Timing." `https://nmap.org/book/port-scanning-options.html`

\[29\] Nmap Project. "Timing Templates." `https://nmap.org/book/performance-timing-templates.html`

\[30\] Republik Indonesia. Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi. `https://peraturan.bpk.go.id/Details/229798/uu-no-27-tahun-2022`

\[31\] Republik Indonesia. Undang-Undang Nomor 1 Tahun 2024 tentang Perubahan Kedua atas Undang-Undang Nomor 11 Tahun 2008 tentang Informasi dan Transaksi Elektronik. `https://peraturan.bpk.go.id/Details/274494/`

*Seluruh sumber daring diperiksa pada 31 Agustus 2026. Materi PEN-200 digunakan sebagai rujukan cakupan kompetensi; narasi, kerangka KENDALI, studi kasus, struktur latihan, dan contoh pada bab ini disusun secara mandiri.*