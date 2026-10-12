--- Recon pasif

akbar gapernah pake nmap karena dia agresif, kadang dan auto filter waf cloudflare. buat platformnya kamu bisa belajar di githubnya juga, alternatif lain pakai naabu, karena ditulis pakai golang dan lebih cepet.

kalau misal masih stuck dan bingung mungkin kamu cari di goverment lokal atau internasional cari cve yang relevan, tapi makesure buat curl kamu bisa deteksi version webnya, biasanya muncul via curl aja kok. jangan terpaku juga sama cve tapi ini penting juga tergantung library atau plugin yang dipakai sama webnya.

buat recon atau yang lain,oiya coba juga hack the box/tryhackme juga

--- tools GLM

ini temen kamu pakai agent ai wkwk jadi dia main di copy api glm entah 5.3 atau tergantung modelnya juga itu pengaruh. ai model china sering guardrails/filternya lengah jadi bisa pakai metode jailbreak atau prompt injection dia bakal nurut, tapi jujur masih banyak yang perlu didalemin soal ini wkwk.

ai red team gitu kadang suka berisik di sisi server kalau ga dikasih tau soal user agent acak misalnya kayak mozilla, chrome, atau safari yang bener-bener kayak bunglon, harus pinter-pinter disini mantaunya.

ada saatnya laporan pernah tembus ke internasional, tudeft.nl itu belanda aku iseng nyoba pakai, cuman settingannya lebih kompleks pakai opencode + model deepseek. disini intinya manusia harus bener-bener validasi impactnya bukan langsung percaya sama outputnya ya, kadang ai mesti halu juga.

https://tudeft.nl/

--- step testing yang selalu sama

aku paling simple aja, pakai tools otomatis paling cuman subfinder buat tau subdomain apa aja yang ke list dan httpx buat ngasih tau ini websitenya hidup apa enggak.

lanjut pelajarin cara analisis javascript, ini penting banget buat ngalahin logikanya. biasanya aku dump semua main.js atau source code via terminal atau cuma curl biasa gitu. kamu bisa pakai bantuan ai juga kok soal ini, cuman makesure kamu kasih tau soal tugas kamu jadi dia bisa strict buat itu wkwk. tapi kadang yang dia larang aku malah lakuin sih.

biasanya kalau bingung aku tanya di grup discord secara personal, ini lebih ngebantu ketimbang ikutan bootcamp yang cuma ngajarin hal dasar soal pentest btw wkwkwk.

biasanya web indo lebih rentan ke dorking, pelajarin itu lumayan penting. butuh jam terbang di burp suite ketimbang tools otomatis. kamu kalau ngandelin tools otomatis yang agresif dan ip kamu terblokir akhirnya di darklist, kena rate limiting dan nunggu lama banget buat ke whitelist. bisa kamu akalin pakai switch ip kok misal ganti hotspot wifi ke hp jadinya aman.

--- tambahan

makesure juga cek portswigger atau juiceshop itu ngebantu juga buat awal-awal, misal business logic flaw. kebanyakan orang yang dapet uang di platform legal via hackerone, yeswehack, atau intigriti, bugcrowd itu makai burp suite semua, jadi make sure kamu perdalamin ini ketimbang materi dalem kampus ya.

discord ngebantu banget sih. channel yang bener-bener bahas misalnya meta4sec atau channelnya 0xriki kamu search di tiktok ada kok link dc nya. aku juga salah satu member disana, jadinya dapet insight khusus dari para sepuh yang dapet uang beneran via platform.

buat awal sih biasanya kamu targetin aja goverment lokal atau kampus, itu kadang rentan banget. lanjut ke internasional, belanda bener-bener welcome banget soal itu. jangan terlalu takut soal izin dan apapun, pokonya ada csirt atau misalnya pelaporan laporin aja. masalah dibales mestinya lama goverment atau kampus indo, makanya sekarang aku ke international atau bug bounty yang bener-bener bagus buat porto sama impact nyatanya.

ada tambahan soal github
https://github.com/swisskyrepo/PayloadsAlltheThings

aplikasiin web programming juga penting, eh openvas waktu itu kenya buat scanning network ya, tapi concern aku tetep yang paling penting di burp suite ya,event versinya community itu berguna banget buat pantau log setiap kamu klik sesuatu di browser burp sendiri,jadi buat endpoint kamu ga harus scanning brutal dan ngasih banyak traffic kesitu,enaknya tinggal manipulasi request,intercept buat cegat traffic masuk terus send dan liat response nya 

buat hack the box sama tryhackme lumayan disana challenge nya lebih banyak variasinya paling sulit mungkin reverse engineering sih bener" bedah from stratch,kalau suatu saat kamu suka sama aplikasi mobile yang mau dibedah

buat di discord kadang juga cuma ikut nimbrung aja wkwk tapi dapet insight lumayan penting buat source misalnya write up itu ada sendiri

luas banget soalnya di cysec ra haha,tapi suatu saat kamu mau lompat ke blue team juga bisa dari jalur red team

oiya buat soal dorking tadi disclaimer aku lupa satu hal,google ada issue soal dorking dan nge ban beberapa kata sensitif via inurl atau .sql,alternatifnya coba pakai search bar nya duck duck go dia lebih powerfull,cysec emang nguras jam tidur banget sih malem begadang besok kelas,tapi gimana lagi buat cari sebuah bug juga harus tau dan ngalahin developer itu sendiri jadi saran aku kuatin mental kamu disitu,sisi gelapnya masih banyak juga,ada yang silent fix juga,sering terjadi di indo btw,jadi kalau nanti udah puas sama bug di indo langsung aja gas cari vdp banyak kok 

buat ai pentesting tadi last option aja gapapa, but misal chit chat via web kayak claude atau deepseek masih enak kok,deepsek juga bagus tapi make sure buat teliti karena dia kadang ngehalu dan coba coba command gajelas sih, struggling diawal aku dulu gitu, tapi pasti ada spark disana kok
