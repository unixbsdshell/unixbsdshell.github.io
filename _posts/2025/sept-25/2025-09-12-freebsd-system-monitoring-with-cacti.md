---
layout: post
title:  "Cara Melakukan Pelacakan Rute Jaringan Dengan Baris Perintah"
date:   2025-08-29 12:00:00 -0700
tags:   Shell
---

Jejak rute paket ke host jaringan menunjukkan semua node perantara yang dilalui paket sebelum mencapai tujuan yang ditentukan. Artinya, dengan menggunakan tracing, Anda dapat mengetahui node mana, dengan alamat IP mana, suatu paket dikirimkan sebelum dikirimkan ke tujuannya.

Penelusuran dapat digunakan untuk mengidentifikasi masalah yang berkaitan dengan pengoperasian jaringan komputer, serta untuk penelitian jaringan (menentukan struktur jaringan, mencari node jaringan perantara, dan lain-lain).

## 1. Cara kerja penelusuran jaringan

Paket jaringan yang diteruskan terdiri dari dua area, yaitu header dan data. Header berisi berbagai informasi, misalnya alamat IP titik pengiriman dan tujuan, port pengiriman dan tujuan, jenis paket, checksum paket, dll. Di antara bidang header, protokol IP memiliki bidang seperti time to live (TTL) atau masa pakai paket. Ini adalah counter dengan nomor yang berkurang satu setiap kali sebuah paket melewati node baru. Counter ini dibuat untuk memastikan bahwa paket yang bermasalah (misalnya jika terjadi kesalahan yang mengakibatkan rute loopback) tidak berjalan melalui jaringan tanpa henti. Artinya, setiap paket, setelah melewati sejumlah node tertentu, pada akhirnya akan mencapai tujuannya atau akan dibuang oleh salah satu node jaringan ketika “masa pakainya” berakhir.

Ketika penghitung TTL menjadi nol, gateway berikutnya tidak meneruskan paket ini lagi. Namun pada saat yang sama, gateway mengirimkan respons **TIME_EXCEEDED** melalui protokol ICMP ke alamat IP asal paket dengan masa berlaku habis (masa pakai telah habis). Dan respons ini berisi alamat IP gateway tempat paket tersebut berakhir.

Jadi, inti dari pelacakan adalah bahwa satu paket dikirim dengan waktu hidup (TTL) diatur ke satu gateway pertama yang akan mengurangi nilainya satu, melihat bahwa penghitung menjadi nol, tidak mengirim paket ini ke mana pun, tetapi mengirimi kami respons bahwa paket "mati".

Kita sudah mengetahui bahwa paket tersebut mati, dari kasus ini kita dapat mencari jawabannya pada alamat IP gateway tempat kemalangan ini terjadi pada paket tersebut. Kemudian sebuah paket dikirim dengan counter disetel ke 2 tetapi paket tetap akan melewati gateway pertama (kita sudah mengetahui IP-nya), tetapi kesialan (counter mencapai nol) sudah terjadi di gateway kedua. Pada kasus ini kita akan menerima respons ICMP dari IP gerbang ini. Kemudian paket berikutnya dikirim, dan seterusnya, hingga semua node dan host jaringan yang kita perlukan teridentifikasi.


## 2. Jenis penelusuran jaringan

Ada beberapa jenis penelusuran. Perbedaan utamanya terletak pada paket yang dikirim - dapat berupa paket protokol transport TCP atau UDP, atau paket Protokol Pesan Kontrol Internet ICMP, atau paket IP mentah.
Terkadang, karena firewall atau konfigurasi node jaringan, alamat IP node tidak dapat diperoleh. Dalam hal ini, Anda dapat mencoba menggunakan metode lain yang mungkin membuahkan hasil.

Hal ini dapat diilustrasikan dengan dua contoh traceroute berikut ke host yang sama:


```bash
root@ns1:~# traceroute unixwinbsd.site
traceroute to unixwinbsd.site (64.190.63.222), 30 hops max, 60 byte packets
 1  192.168.17.1 (192.168.17.1)  0.180 ms  0.176 ms  0.133 ms
 2  192.168.1.1 (192.168.1.1)  0.651 ms  0.760 ms  0.881 ms
 3  36.70.96.1 (36.70.96.1)  2.701 ms  3.155 ms  3.241 ms
 4  180.252.2.141 (180.252.2.141)  3.526 ms  3.787 ms  3.926 ms
 5  180.240.190.77 (180.240.190.77)  96.032 ms  16.635 ms  18.080 ms
 6  180.240.190.77 (180.240.190.77)  17.020 ms  14.468 ms  15.354 ms
 7  180.240.192.233 (180.240.192.233)  170.584 ms  187.078 ms  170.548 ms
 8  180.240.196.1 (180.240.196.1)  169.914 ms  166.764 ms  170.556 ms
 9  193.239.116.37 (193.239.116.37)  190.583 ms  211.423 ms  196.257 ms
10  lo-0-0.bb-a.ess.muc.de.net.ionos.com (212.227.117.142)  189.415 ms  206.294 ms  193.498 ms
11  lo-0-0.rc-a.ess.muc.de.net.ionos.com (212.227.117.118)  189.902 ms  206.117 ms  207.581 ms
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
root@ns1:~#
```

Tanda bintang pada contoh di atas menunjukkan bahwa kami tidak mengenali beberapa node. Sekarang kita coba dengan menambahkan flags `-I`. Lihat hasilnya.

```bash
root@ns1:~# traceroute -I unixwinbsd.site
traceroute to unixwinbsd.site (64.190.63.222), 30 hops max, 60 byte packets
 1  192.168.17.1 (192.168.17.1)  0.356 ms  57.473 ms *
 2  192.168.1.1 (192.168.1.1)  57.536 ms  0.777 ms  0.933 ms
 3  36.70.96.1 (36.70.96.1)  2.946 ms * *
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * 180.240.196.1 (180.240.196.1)  176.946 ms  176.771 ms
 9  193.239.116.37 (193.239.116.37)  190.257 ms * *
10  lo-0-0.bb-a.ess.muc.de.net.ionos.com (212.227.117.142)  187.481 ms  187.789 ms  188.386 ms
11  lo-0-0.rc-a.ess.muc.de.net.ionos.com (212.227.117.118)  187.869 ms  188.004 ms  188.464 ms
12  ve-1460.bb-1-slx-sedo.ess.muc.de.net.ionos.com (212.227.158.119)  197.632 ms * *
13  91.195.241.102 (91.195.241.102)  190.068 ms  190.381 ms *
14  * * *
15  * * *
16  * * *
17  * * *
18  64.190.63.222 (64.190.63.222)  186.675 ms  187.011 ms  186.996 ms
root@ns1:~#
```

Dengan menambahkan flags `-I` beberapa node mulai dapat ditemukan.

Berkat perubahan metode penelusuran, semua node perantara dapat ditemukan. Metode lain mungkin memberikan hasil yang berbeda dari yang ditunjukkan.

Beberapa program memungkinkan Anda memilih metode penelusuran, mengubah nomor port, dan juga mengatur nilai beberapa bidang di header paket.



## 3. Software untuk penelusuran jaringan

Ada banyak utilitas penelusuran berbeda yang tersedia, beberapa di antaranya mendukung metode penelusuran berbeda. Contoh program tersebut:

- traceroute
- tracepath
- mtr and mtr-gtk (console and graphical versions, respectively)
- lft
- tcptraceroute

Anda juga dapat menentukan node rute paket menggunakan Nmap (beberapa cara) dan bahkan menggunakan ping !

Pada artikel ini saya akan mengulas semua program yang tercantum di atas. Mari kita mulai dengan traceroute , karena ia mengimplementasikan sebagian besar metode pemindaian.

### A. Cara menggunakan traceroute
Untuk mulai melacak, cukup tentukan IP atau situs yang ingin Anda lacak rutenya:

```bash
$ traceroute 36.90.8.64
```

Jika Anda tertarik pada node terdekat (jaringan lokal, misalnya), maka Anda dapat memilih situs mana pun sebagai tujuan akhir.

#### a. Metode penelusuran di traceroute

Dalam lingkungan jaringan saat ini, metode penelusuran tradisional tidak selalu dapat diterapkan karena meluasnya penggunaan firewall. Firewall semacam itu memfilter port UDP yang “tidak mungkin” atau bahkan paket gema ICMP. Untuk mengatasi masalah ini, beberapa metode penelusuran jaringan tambahan (termasuk tcp) telah diterapkan. Metode ini mencoba menggunakan protokol dan port sumber/tujuan yang berbeda untuk melewati firewall (sehingga firewall menganggapnya hanya sebagai permulaan sesi jaringan dari jenis yang diizinkan).

Metode yang berbeda memiliki opsi khusus, bergantung pada protokol yang digunakan. Opsi metode ini dapat ditentukan menggunakan opsi `-O`. Beberapa opsi dipisahkan dengan koma (atau gunakan beberapa `-Os` pada baris perintah ). Masing-masing metode mungkin mempunyai pilihan khususnya sendiri atau mungkin tidak ada sama sekali.

##### a.1. default
Metode default - digunakan jika tidak ada metode lain yang ditentukan, atau dapat ditentukan secara eksplisit dengan  opsi -M default . Ini adalah metode penelusuran rute yang tradisional dan kuno.

Paket probe adalah datagram UDP dengan port tujuan yang disebut "tidak mungkin". Port probe "tidak mungkin" pertama adalah 33434, kemudian untuk setiap probe berikutnya bertambah satu. Karena port tersebut diperkirakan tidak digunakan, host tujuan biasanya akan mengembalikan respons akhir "icmp unreach port". Nomor port dapat diubah (lebih lanjut tentang ini di bawah).

Metode ini dapat dilakukan oleh pengguna yang tidak memiliki hak istimewa.

##### a.2. icmp
Ini adalah metode yang paling umum, menggunakan paket icmp echo sebagai probe. Jika Anda dapat melakukan ping ke host tujuan, maka penelusuran icmp juga dapat diterapkan. Untuk memilih metode ini, gunakan opsi -M icmp atau versi singkatnya -I .

Metode ini diperbolehkan untuk pengguna yang tidak memiliki hak istimewa.

Metode ini memiliki dua opsi spesifik:

##### a.2.1. raw
Gunakan hanya soket raw (metode tradisional). Secara default, metode ini dicoba terlebih dahulu (untuk alasan kompatibilitas), kemudian soket dgram icmp baru dicoba sebagai cadangan.

##### a.2.2. dgram
Gunakan hanya soket dgram icmp.

##### a.3. TCP
Metode modern terkenal yang dirancang untuk melewati firewall. Untuk menggunakannya, tentukan opsi -M tcp atau opsi singkat -T . Menggunakan port tujuan tetap (defaultnya adalah 80, http).

Jika ada filter pada jalur pelacakan jaringan, kemungkinan besar port udp yang "tidak biasa" (seperti yang digunakan oleh metode default) atau bahkan icmp echo (seperti untuk icmp) difilter, dan seluruh proses penelusuran akan difilter. berhenti di firewall seperti itu. Untuk melewati filter jaringan, kita hanya perlu menggunakan kombinasi pasangan protokol/port yang diizinkan. Jika kita melakukan penelusuran ke, katakanlah, server email, kemungkinan besar kita dapat menjangkaunya dengan -T -p 25 , meskipun kita tidak dapat menjangkaunya dengan -I .

Metode ini menggunakan “teknik koneksi setengah terbuka” yang terkenal, yang menyebabkan aplikasi di komputer tujuan tidak melihat paket probe kami sama sekali. Biasanya dikirim melalui tcp syn. Untuk port yang tidak mendengarkan, kami mendapat respons reset tcp dan selesai. Untuk port yang mendengarkan secara aktif, kita menerima tcp syn+ack, namun menanggapinya dengan tcp reset (bukannya tcp ack yang diharapkan), sehingga sesi tcp jarak jauh direset, dan aplikasi yang mendengarkan pada port tersebut bahkan tidak menerima pemberitahuan.

Ada beberapa opsi untuk metode tcp:

##### a.3.1. syn,ack,fin,rst,psh,urg,ece,cwr
Menetapkan tanda TCP tertentu untuk paket probing, kombinasi apa pun darinya dapat digunakan.

##### a.3.2. flags=NUMBER
Menyetel bidang flags di header tcp ke nomor yang tepat.

##### a.3.3. ecn
Mengirimkan paket syn dengan flag tcp ECE dan CWR (untuk Pemberitahuan Kemacetan Eksplisit, rfc3168).

a.3.4. sack,timestamps,window_scaling
Menggunakan opsi header tcp yang sesuai dalam paket probing keluar.

##### a.3.5. sysctl
Menggunakan pengaturan sysctl saat ini (/proc/sys/net/*) untuk opsi header tcp untuk opsi di atas dan ecn. Selalu disetel ke default jika tidak ada yang ditentukan.

##### a.3.6. mss=NUMBER
Gunakan NUM untuk opsi header maxseg tcp (saat sinkronisasi).

##### a.3.7. info
Cetak tanda tcp dari respons tcp akhir ketika host target tercapai. Membantu menentukan apakah suatu aplikasi mendengarkan pada suatu port dan hal berguna lainnya. 

Opsi defaultnya adalah syn,sysctl.

##### a.4. tcpconn
Implementasi awal metode tcp hanya menggunakan panggilan connect(2), yang membuka sesi tcp penuh. Tidak disarankan untuk penggunaan normal karena selalu mempengaruhi aplikasi yang mendengarkan pada port pada host tujuan.

Untuk mengaktifkan metode ini, gunakan opsi -M tcpconn.

##### a.5. udp
Menggunakan datagram udp dengan port tujuan tetap (default 53, dns). Juga dirancang untuk melewati firewall. Untuk menggunakan metode penelusuran ini, tentukan opsi -M udp atau pintasan -U .

Perhatikan bahwa tidak seperti metode tcp, aplikasi terkait pada host tujuan selalu menerima probe kami (dengan data acak) yang dapat membingungkannya. Dalam kebanyakan kasus, ia tidak akan merespons paket kami, jadi kami tidak akan pernah melihat hop (node) terakhir di jalur traceroute. (Untungnya, tampaknya setidaknya server DNS mengirimkan semacam respons).

Cara ini tidak memerlukan hak yang lebih tinggi.

##### a.6. udplite
Menggunakan probe datagram udplite (dengan port tujuan tetap, default 53). Untuk mengaktifkan metode ini, tentukan opsi -M udplite atau -UL .

Cara ini tidak memerlukan hak yang lebih tinggi.

Pilihan:

coverage=NUMBER
Menyetel cakupan pengiriman udplite ke NUM .

##### a.7. dccp
Menggunakan paket Permintaan DCCP (rfc4340) untuk probe. Metode ini dapat diaktifkan dengan opsi -M dccp atau -D .

Metode ini menggunakan "teknik koneksi setengah terbuka" yang sama yang digunakan untuk TCP. Port tujuan defaultnya adalah 33434.

Pilihan:

service=NUMBER
Menyetel kode layanan DCCP ke NUMBER (defaultnya adalah 1885957735).

##### a.8. raw
Metode ini mengirimkan paket mentah dari protokol yang ditentukan. Untuk memanggil metode ini, gunakan opsi -M raw atau -P PROTOCOL .

Tidak ada header khusus protokol transport yang digunakan, hanya header protokol IP.

Menyiratkan `-N 1 -w 5`.

Pilihan:

protocol=PROTOCOL

Gunakan PROTOKOL IP (default 253).

## 4. Cara mempercepat penelusuran. Cara menonaktifkan resolusi IP terbalik ke nama host saat menelusuri
Prinsip cara kerja penelusuran dijelaskan tepat di atas - mengirimkan paket dengan masa pakai yang terus meningkat. Faktanya, semua paket (dengan TTL 1, dengan TTL 2, dengan TTL 3, dst) dapat dikirim secara bersamaan. Dan inilah yang terjadi - secara default, 16 paket dikirim sekaligus (jumlahnya dapat diubah dengan opsi `-N` ). Hal ini dilakukan untuk mempercepat penelusuran.

Oleh karena itu, tracing sebenarnya sangat cepat. 1-2 detik yang menurut kami untuk menentukan node jaringan sebenarnya dihabiskan untuk mendapatkan nama host untuk IP. Ini dapat dinonaktifkan menggunakan opsi `-n`.

Dengan menggunakan program waktu , Anda dapat mengukur waktu eksekusi suatu program dengan dan tanpa opsi `-n`:

```bash
root@ns1:~# time traceroute -n google.com
traceroute to google.com (2404:6800:4003:c11::8a), 30 hops max, 80 byte packets
 1  2001:470:36:61c:4000::1  0.692 ms  0.651 ms  0.615 ms
 2  2001:470:35:61c::1  20.170 ms  22.195 ms  24.166 ms
 3  * * *
 4  2404:6800:834a:340::1  38.125 ms 2404:6800:8341::1  38.159 ms  38.806 ms
 5  2404:6800:8341:80::1  39.740 ms 2404:6800:834a:280::1  39.272 ms 2404:6800:8341::1  39.286 ms
 6  2001:4860:0:1::6964  39.194 ms 2001:4860:0:1::322  40.784 ms 2001:4860:0:1::45a  39.725 ms
 7  2001:4860:0:1::9b3a  39.998 ms 2001:4860:0:1::560c  37.542 ms 2001:4860:0:1::7e96  48.404 ms
 8  2001:4860::c:4003:1cb9  36.134 ms 2001:4860::c:4003:1c92  31.446 ms 2001:4860::c:4003:1caf  31.860 ms
 9  2001:4860:0:1::534a  32.283 ms 2001:4860::cc:4004:b25d  31.315 ms  31.096 ms
10  2001:4860:0:1::5377  30.867 ms * 2001:4860:0:1::53b1  30.858 ms
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  2404:6800:4003:c11::8a  19.158 ms *  19.138 ms

real    0m5.781s
user    0m0.000s
sys     0m0.011s
root@ns1:~#
```
<br/>

```bash
root@ns1:~# time traceroute google.com
traceroute to google.com (2404:6800:4003:c11::64), 30 hops max, 80 byte packets
 1  dns1.unixbsdshell.site (2001:470:36:61c:4000::1)  0.860 ms  0.809 ms  0.768 ms
 2  tunnel1029870.tunnel.tserv25.sin1.ipv6.he.net (2001:470:35:61c::1)  17.985 ms  20.867 ms  22.559 ms
 3  * * *
 4  * 2404:6800:8341:40::1 (2404:6800:8341:40::1)  29.399 ms 2404:6800:8341::1 (2404:6800:8341::1)  29.671 ms
 5  2404:6800:834a:280::1 (2404:6800:834a:280::1)  29.000 ms 2404:6800:834a:340::1 (2404:6800:834a:340::1)  29.427 ms  29.424 ms
 6  2001:4860:0:1::77bc (2001:4860:0:1::77bc)  30.273 ms 2001:4860:0:1::2aa2 (2001:4860:0:1::2aa2)  23.257 ms 2001:4860:0:1::322 (2001:4860:0:1::322)  21.937 ms
 7  2001:4860:0:1::f6 (2001:4860:0:1::f6)  22.299 ms  72.963 ms 2001:4860:0:1::7e96 (2001:4860:0:1::7e96)  71.720 ms
 8  2001:4860::c:4003:1cb8 (2001:4860::c:4003:1cb8)  73.311 ms  71.914 ms 2001:4860::c:4003:1cb9 (2001:4860::c:4003:1cb9)  72.881 ms
 9  2001:4860::cc:4004:b25a (2001:4860::cc:4004:b25a)  73.294 ms 2001:4860:0:1::53b0 (2001:4860:0:1::53b0)  73.416 ms 2001:4860:0:1::5376 (2001:4860:0:1::5376)  73.841 ms
10  2001:4860:0:1::534b (2001:4860:0:1::534b)  73.650 ms 2001:4860:0:1::53b5 (2001:4860:0:1::53b5)  53.295 ms  53.500 ms
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  se-in-f100.1e100.net (2404:6800:4003:c11::64)  17.208 ms  17.251 ms  19.628 ms

real    0m10.296s
user    0m0.000s
sys     0m0.017s
root@ns1:~#
```

Waktu eksekusi tracingnya sendiri adalah 0,206 detik, dan waktu tracing + penentuan nama host ternyata 2.360 detik, yaitu 10 kali lebih lama.

## 5. Pelacakan IPv6

Secara default, program menerima IP untuk host yang ditentukan untuk penelusuran dan, berdasarkan alamat IP yang diterima, secara otomatis menentukan protokol mana yang digunakan: IPv4 atau IPv6. Jika IP untuk kedua protokol diterima, maka IPv4 digunakan secara default.

Menggunakan opsi -4 dan -6 Anda dapat secara eksplisit menentukan protokol yang diinginkan.

Misalnya:

```bash
root@ns1:~# traceroute -6 -n google.com
traceroute to google.com (2404:6800:4003:c11::71), 30 hops max, 80 byte packets
 1  2001:470:36:61c:4000::1  0.223 ms  0.215 ms  0.171 ms
 2  2001:470:35:61c::1  18.107 ms  19.887 ms  22.710 ms
 3  * * *
 4  2404:6800:8341:c0::1  29.826 ms * 2404:6800:8341:40::1  31.148 ms
 5  2404:6800:8341:40::1  31.104 ms  31.118 ms 2404:6800:834a:300::1  30.678 ms
 6  2001:4860:0:1::6962  31.554 ms 2001:4860:0:1::77bc  33.965 ms 2001:4860:0:1::1d96  33.822 ms
 7  2001:4860:0:1::9b36  31.399 ms 2001:4860:0:1::9b38  29.537 ms 2001:4860:0:1::560c  28.808 ms
 8  2001:4860::c:4003:1cb0  29.241 ms 2001:4860::c:4003:1cba  28.781 ms 2001:4860::c:4003:1c92  26.546 ms
 9  2001:4860:0:1::534a  26.719 ms 2001:4860:0:1::53b2  27.404 ms 2001:4860::cc:4004:b25a  26.741 ms
10  2001:4860:0:1::5377  25.985 ms 2001:4860:0:1::534b  25.898 ms *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * 2404:6800:4003:c11::71  41.536 ms  41.601 ms
root@ns1:~#
```

Jaringan tempat pelacakan dibuat dengan opsi -6 harus mendukung IPv6, jika tidak, tidak akan ada yang berhasil.

## 6. Mengubah port

Anda dapat mengubah port tujuan menggunakan opsi:

```bash
-p PORT, --port=port
```

Contohnya penggunaaan scriptnya dapat anda lihat di bawah ini

```bash
root@ns1:~# traceroute -p 443 unixbsdshell.site
traceroute to unixbsdshell.site (2001:470:36:61c:4000::2005), 30 hops max, 80 byte packets
 1  * ns1.unixbsdshell.site (2001:470:36:61c:4000::2005)  0.032 ms  0.021 ms
root@ns1:~#
```

Untuk penelusuran UDP, port yang ditentukan akan digunakan sebagai port dasar (nomor port tujuan akan bertambah untuk setiap probe). Untuk jejak ICMP, nomor yang ditentukan akan digunakan sebagai nilai rangkaian ICMP awal (juga bertambah untuk setiap probe).

Untuk TCP dan lainnya, port yang ditentukan akan digunakan sebagai port tujuan permanen untuk terhubung. Biasanya ini tidak diperlukan, tetapi Anda juga dapat menentukan port sumber, ini dilakukan dengan opsi:

```bash
$ --sport=port
```

Opsi ini menyiratkan `-N 1 -w 5`. Biasanya, port sumber (jika berlaku untuk metode penelusuran yang dipilih) dipilih oleh sistem.

## 7. Bagaimana memulai penelusuran dari node tertentu. Cara mengurangi atau menambah jumlah node yang akan dilacak

Dengan menggunakan opsi `-f`, Anda dapat menentukan nomor simpul untuk memulai penelusuran. Nilai defaultnya adalah 1.

Dengan menggunakan opsi `-m`, Anda dapat menentukan jumlah hop maksimum untuk penelusuran; secara default diatur ke 30.

## 8. Memilih antarmuka untuk penelusuran

Secara default, paket dikirim dari antarmuka yang rute defaultnya dikonfigurasi. Namun dengan opsi berikut Anda dapat membuat traceroute mengirim paket dari antarmuka yang ditentukan pada baris perintah:

```bash
$ -i INTERFACE, --interface=INTERFACE
```

Contoh penggunaan scriptnya dapat anda lihat pada contoh di bawah ini.

```bash
root@ns1:~# traceroute -i enp3s0 google.com
traceroute to google.com (2404:6800:4003:c11::71), 30 hops max, 80 byte packets
 1  dns1.unixbsdshell.site (2001:470:36:61c:4000::1)  0.965 ms  0.864 ms  0.778 ms
 2  tunnel1029870.tunnel.tserv25.sin1.ipv6.he.net (2001:470:35:61c::1)  30.538 ms  33.096 ms  34.696 ms
 3  * * *
 4  2404:6800:8341:c0::1 (2404:6800:8341:c0::1)  53.959 ms 2404:6800:834a:340::1 (2404:6800:834a:340::1)  54.294 ms 2404:6800:8341:40::1 (2404:6800:8341:40::1)  54.591 ms
 5  2404:6800:834a:300::1 (2404:6800:834a:300::1)  54.663 ms 2404:6800:834a:280::1 (2404:6800:834a:280::1)  55.094 ms 2404:6800:8341:40::1 (2404:6800:8341:40::1)  55.089 ms
 6  2001:4860:0:1::2260 (2001:4860:0:1::2260)  56.940 ms 2001:4860:0:1::1f90 (2001:4860:0:1::1f90)  39.379 ms 2001:4860:0:1::77ba (2001:4860:0:1::77ba)  39.053 ms
 7  2001:4860:0:1::9b38 (2001:4860:0:1::9b38)  39.099 ms 2001:4860:0:1::560c (2001:4860:0:1::560c)  61.137 ms  61.144 ms
 8  2001:4860::c:4003:1cb9 (2001:4860::c:4003:1cb9)  61.315 ms 2001:4860::c:4003:1cba (2001:4860::c:4003:1cba)  61.800 ms 2001:4860::c:4003:1cb0 (2001:4860::c:4003:1cb0)  61.309 ms
 9  2001:4860:0:1::5350 (2001:4860:0:1::5350)  72.309 ms 2001:4860::cc:4004:b25d (2001:4860::cc:4004:b25d)  61.930 ms 2001:4860:0:1::53b2 (2001:4860:0:1::53b2)  62.674 ms
10  * 2001:4860:0:1::53ad (2001:4860:0:1::53ad)  61.432 ms *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  se-in-f113.1e100.net (2404:6800:4003:c11::71)  26.105 ms  26.471 ms  26.538 ms
root@ns1:~#
```
Dimana `enp3s0` adalah interface atau Lancard yang ada dikomputer kami.

## 9. Cara menunjukkan sistem otonom mana yang dimiliki suatu node saat menelusuri

Setiap alamat IP dikaitkan dengan Sistem Otonom (AS) . Dengan opsi -A Anda dapat mengaktifkan permintaan nomor AS untuk setiap node di sepanjang jalur pelacakan, misalnya:

```bash
root@ns1:~# traceroute -A -n unixwinbsd.site
traceroute to unixwinbsd.site (64.190.63.222), 30 hops max, 60 byte packets
 1  192.168.17.1 [*]  0.647 ms  0.609 ms  0.581 ms
 2  192.168.1.1 [*]  1132.205 ms  1132.185 ms  1132.161 ms
 3  36.70.96.1 [AS7713]  2.903 ms  3.167 ms  4.151 ms
 4  180.252.2.141 [AS7713]  3.638 ms  4.218 ms  4.178 ms
 5  180.240.190.77 [AS7713]  16.841 ms  17.099 ms  27.614 ms
 6  180.240.190.77 [AS7713]  17.344 ms  14.573 ms  14.650 ms
 7  180.240.192.233 [AS7713]  187.672 ms  170.452 ms  170.711 ms
 8  180.240.196.1 [AS7713]  187.014 ms  167.353 ms  183.628 ms
 9  193.239.116.37 [AS0]  195.771 ms  205.464 ms  212.379 ms
10  212.227.117.142 [AS8560]  190.671 ms  190.680 ms  206.768 ms
11  212.227.117.118 [AS8560]  193.994 ms  209.868 ms  209.051 ms
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
root@ns1:~#
```
Seperti yang Anda lihat, sistem otonom tidak ditentukan untuk IP lokal 10.*.*.* - yang cukup logis, karena alamat ini tidak diberikan kepada siapa pun. Mengenai alamat 192.168.1.1 dan unit otonom AS198949, ini adalah semacam kesalahan.

Seperti yang Anda lihat dari output perintah sebelumnya, empat node pertama memiliki alamat IP lokal. Node 5 hingga 9 milik sistem otonom AS38082/AS7470 yang sama. Dua node kedua dari belakang milik satu sistem otonom AS12389 dan dua node terakhir juga milik satu sistem otonom AS48666 - penyedia Internet yang menghosting situs unixwinbsd.site

#### traceroute Options
Perintah traceroute memiliki opsi lain yang mungkin berguna bagi Anda. Anda dapat menemukannya di halaman bantuan program ini:

```bash
$ man traceroute
```

## 10. Cara menggunakan tracepath

Program tracepath mirip dengan traceroute , tetapi hanya menggunakan satu teknik penelusuran: UDP, yang untuknya Anda dapat menentukan port khusus. Karena teknik yang dipilih, program ini tidak memerlukan hak istimewa yang lebih tinggi.

Contoh penggunaan:

```bash
root@ns1:~# tracepath unixwinbsd.site
1?: [LOCALHOST]                      pmtu 1500
 1:  _gateway                                              1.247ms 
 1:  _gateway                                              1.031ms 
 2:  10.20.48.1                                            9.097ms 
 3:  10.246.245.241                                       14.034ms 
 4:  10.185.252.194                                       14.379ms 
 5:  10.185.252.29                                        11.530ms asymm  4 
 6:  58-97-121-237.static.asianet.co.th                   13.849ms asymm  5 
 7:  171-102-247-184.static.asianet.co.th                 15.737ms asymm  6 
 8:  171-102-250-1.static.asianet.co.th                   64.185ms asymm  7 
 9:  171-102-254-232.static.asianet.co.th                 14.962ms asymm  8 
10:  171-102-250-156.static.asianet.co.th                 13.509ms asymm  9 
11:  122.155.226.89                                       18.793ms 
12:  61.19.9.66                                           58.829ms 
13:  no reply
14:  87.226.181.87                                       399.972ms asymm 23 
15:  81.177.108.86                                       263.969ms asymm 14 
16:  j37-ae9-3001.marosnet.net                           307.140ms 
17:  suip.biz                                            304.644ms reached
     Resume: pmtu 1500 hops 17 back 17
```

Di kolom pertama, mungkin ada tanda tanya di sebelah nomor node - ini berarti nomor TTL tidak ada dalam respons yang dikirim dan program mencoba menebaknya. Alih-alih tanda bintang, jika IP tidak dikenali, tidak ada balasan yang ditulis .

Kolom terakhir mungkin berisi angka dan kata asymm . Kata asymm berarti rutenya asimetris - yaitu, dari kita ke node ini paket melewati satu jalur, dan dari node ini ke kita paket melewati jalur yang berbeda. Angka tersebut berarti kemungkinan jumlah lompatan dari node ini ke kita - namun informasinya tidak dapat diandalkan.

tracepath tidak memiliki banyak pilihan:

- -4 : Gunakan IPv4 saja
- -6 : Gunakan IPv6 saja
- -N : Jangan cetak nama host, tetapi cetak nilai IP numerik.
- -B : Cetak nama host dan alamat IP dalam bentuk digital.
- -l : Tetapkan panjang paket awal, bukan 65535 untuk tracepath atau 128000 untuk tracepath6.
- -M : Tetapkan jumlah hop maksimum (atau TTL maksimum) - yaitu, jumlah node "yang disadap" maksimum. Standarnya adalah 30.
- -P : Tetapkan port tujuan awal.

## 11. Cara menggunakan mtr dan mtr-gtk (versi konsol dan grafis)

Program mtr menggabungkan fungsionalitas program traceroute dan ping ke dalam satu alat diagnostik jaringan. Artinya, program ini menunjukkan rute ke node yang ditentukan dan terus melakukan ping ke setiap hop dan pada saat yang sama mengumpulkan statistik kerugian umum - berdasarkan data ini, Anda dapat menentukan node bermasalah di mana paket hilang.

Contoh penggunaan:

```bash
root@mail:~ # mtr unixbsdshell.site
                                                   My traceroute  [v0.96]
mail.unixbsdshell.site (2001:470:36:61c:4000::2000) -> unixbsdshell.site (2001:470:36:61c:4000::20052026-08-26T13:40:47+0700
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                                                    Packets               Pings
 Host                                                                             Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. www.unixbsdshell.site                                                          0.0%    23    0.1   0.2   0.1   0.7   0.1

```

Program ini mendukung beberapa metode penelusuran dan juga mendukung format keluaran yang berbeda untuk menyimpan hasil, misalnya opsi `-C, --csv` untuk menyimpan hasil dalam format CSV (perhatikan bahwa pemisah sebenarnya bukanlah koma, melainkan titik koma), serta opsi `-j, --json` untuk menyimpan dalam format keluaran JSON.

Menggunakan opsi `-n, --no-dns` Anda dapat menonaktifkan resolusi IP ke nama host. Dengan opsi `-b, --show-ips` Anda dapat mengaktifkan tampilan nama host dan alamat IP.

Dengan opsi `-yn, --ipinfo` n Anda dapat mengkonfigurasi tampilan informasi tambahan tentang setiap IP hop. Untuk nilai `n` Anda perlu menentukan angka yang artinya:
- 0 Show autonomous system (AS) number (equivalent to -z)
- 1 Show IP prefix
- 2 Show AS-based country code
- 3 Show RIR (ripencc, arin, ...)
- 4 Show IP prefix allocation date

Bagi saya, untuk nilai `-y` apa pun, hanya nomor sistem otonom yang selalu ditampilkan. Untungnya, Anda dapat menelusuri berbagai tampilan menggunakan tombol `"y"`.

- Opsi `-z, --aslookup` menampilkan nomor Sistem Otonom (AS) untuk setiap hop.
- Opsi `-f` NUM digunakan untuk mengatur nomor TTL pertama. Defaultnya adalah 1.
- Opsi `-m `menentukan jumlah maksimum hop (nilai waktu hidup maksimum) yang akan diproses selama penelusuran. Standarnya adalah 30.
- Opsi `-U` NUM menetapkan jumlah maksimum host yang tidak diketahui. Standarnya adalah 5 . Rupanya, setelah mencapai nilai tersebut, penelusuran lebih lanjut akan dihentikan.
- Dengan opsi `-u, --udp` adalah program akan menggunakan datagram UDP, bukan ICMP ECHO.
- Dan dengan opsi `-T, --tcp` Anda dapat mengatur penggunaan paket TCP SYN alih-alih ICMP ECHO. PACKETSIZE diabaikan karena paket SYN tidak dapat memuat data.

Dengan perintah mtr Anda bahkan dapat menggunakan protokol SCTP untuk penelusuran; untuk melakukan ini, tentukan opsi -S , --sctp dan paket Protokol Transmisi Kontrol Aliran akan digunakan sebagai pengganti ICMP ECHO.

```bash
$ -P PORT, --port PORT
Destination port number for TCP/SCTP/UDP traces.

$ -L LOCAL_PORT, --localport LOCAL_PORT
Source port number for UDP traces.
```

Perintah interaktif tersedia saat program sedang berjalan. Jika Anda menekan d , Anda dapat beralih di antara tampilan yang berbeda.

Menggunakan tombol r Anda dapat mengatur ulang statistik.

Anda dapat menggunakan tombol o untuk mengubah urutan kolom. Omong-omong, dengan opsi -o Anda dapat mengatur bidang mana yang ingin Anda tampilkan dan urutannya. Untuk detailnya lihat:

```bash
$ man mtr
```

## 12. Cara menggunakan lft

Program lft memiliki banyak opsi penelusuran dan dari uraiannya berikut ini bahwa program mencoba beberapa kombinasi dan secara otomatis memilih solusi terbaik. Sejujurnya, saya tidak memperhatikan hal ini: hasil dari mencoba berbagai metode penelusuran secara manual memungkinkan Anda memilih opsi terbaik.

Program ini didokumentasikan dengan baik dan dapat digunakan sebagai alternatif traceroute jika diinginkan .

## 13. Cara menggunakan tcptraceroute

Program tcptraceroute hanya menggunakan paket dari satu protokol TCP. Anda dapat menyetel tanda berbeda di header paket ini dengan opsi. Program traceroute juga memungkinkan Anda menyetel flag protokol TCP, dan terdapat lebih banyak opsi konfigurasi.


## 14. Pelacakan Jaringan di Nmap
Nmap mempunyai pilihan --traceroute untuk tracing , contoh tracing ke website unixwinbsd.site.

```bash
$ sudo nmap --traceroute unixwinbsd.site
```

Jika Anda tidak ingin memindai port, tetapi hanya ingin melakukan penelusuran, tambahkan opsi -sn.

Nmap mempunyai pilihan --traceroute untuk tracing , contoh tracing ke website unixwinbsd.site.

```bash
$ sudo nmap --traceroute -sn unixwinbsd.site
```

Ini akan mengurangi waktu hingga hasilnya ditampilkan secara signifikan. Kebetulan data yang dikeluarkan oleh nmap saat tracing belum selesai. Dalam hal ini, coba tambahkan opsi -PE tambahan.

```bash
$ sudo nmap --traceroute -sn -PE unixwinbsd.site
```

Di Nmap Anda dapat mengatur opsi di header paket protokol IP. Di antara pilihan tersebut ada satu yang menyimpan rute yang dilalui di header paket. Namun opsi ini memiliki sejumlah keterbatasan:

- jumlah 9 slot
- beberapa perangkat mengabaikan opsi ini
- beberapa perangkat tidak mengizinkan paket melewatinya sama sekali dengan opsi ini diinstal

Namun terkadang berhasil, contoh perintah:

```bash
$ sudo nmap -sn --ip-options "R" -n --packet-trace unixwinbsd.site
```

Hasil keluarannya akan tampak seperti ini.

```bash
Binary ip options to be send:
\x01\x07\x27\x04\x00\x00\x00\x00 \x00\x00\x00\x00\x00\x00\x00\x00
\x00\x00\x00\x00\x00\x00\x00\x00 \x00\x00\x00\x00\x00\x00\x00\x00
\x00\x00\x00\x00\x00\x00\x00\x00 
Parsed ip options to be send:
 NOP RR{#0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0}
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-12 18:02 MSK
SENT (0.8154s) ICMP [192.168.1.57 > 185.117.153.79 Echo request (type=8/code=0) id=64674 seq=0] IP [ttl=42 id=53218 iplen=68 ipopts={ NOP RR{#0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0}}]
SENT (0.8154s) TCP 192.168.1.57:36579 > 185.117.153.79:443 S ttl=43 id=9871 iplen=84 ipopts={ NOP RR{#0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0}} seq=1013479372 win=1024 <mss 1460>
SENT (0.8154s) TCP 192.168.1.57:36579 > 185.117.153.79:80 A ttl=41 id=45814 iplen=80 ipopts={ NOP RR{#0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0}} seq=0 win=1024 
SENT (0.8155s) ICMP [192.168.1.57 > 185.117.153.79 Timestamp request (type=13/code=0) id=32210 seq=0 orig=0 recv=0 trans=0] IP [ttl=55 id=35087 iplen=80 ipopts={ NOP RR{#0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0 0.0.0.0}}]
RCVD (1.1526s) ICMP [185.117.153.79 > 192.168.1.57 Echo reply (type=0/code=0) id=64674 seq=0] IP [ttl=48 id=44985 iplen=68 ipopts={ RR{ 10.246.245.242 10.185.252.193 10.185.252.29 10.185.0.12 171.102.250.3 171.102.250.128 203.144.128.48 203.144.144.8 103.3.177.50#} EOL}]
Nmap scan report for suip.biz (185.117.153.79)
Host is up (0.34s latency).
Other addresses for suip.biz (not scanned): 2a02:f680:1:1100::3d5f
```

Anda perhatikan Perhatikan barisnya.

```bash
RCVD (1.1526s) ICMP [185.117.153.79 > 192.168.1.57 Echo reply (type=0/code=0) id=64674 seq=0] IP [ttl=48 id=44985 iplen=68 ipopts={ RR{ 10.246.245.242 10.185.252.193 10.185.252.29 10.185.0.12 171.102.250.3 171.102.250.128 203.144.128.48 203.144.144.8 103.3.177.50#} EOL}]
```

## 15. Menelusuri jaringan dengan perintah ping

Program ping juga dapat merekam rute; untuk melakukan ini, Anda perlu menjalankan program dengan opsi -R . Ia menambahkan opsi RECORD_ROUTE ke paket ECHO_REQUEST dan menampilkan buffer rute dari paket yang dikembalikan. Ini adalah opsi yang sama yang digunakan Nmap. Batasannya sama: maksimal 9 slot, banyak host mengabaikan atau membuang opsi ini.

Contoh penggunaan:

```bash
$ ping -R unixwinbsd.site
```

Rute IP ditampilkan dengan setiap ping. Jika rute tidak berubah, maka akan ditampilkan pesan bahwa rute tidak berubah.

Terlepas dari semua keterbatasan opsi RECORD_ROUTE, terkadang ini adalah satu-satunya pilihan untuk mendapatkan setidaknya beberapa informasi tentang rute, karena perintah ping hampir selalu ada dan tidak memerlukan hak istimewa yang lebih tinggi untuk menjalankannya.

## 16. Menelusuri jaringan melalui Windows tracert
Windows memiliki perintah tracert bawaan untuk penelusuran jaringan . Ia hampir tidak punya pilihan. Untuk menjalankan perintah, cukup tentukan nama host jarak jauh:

```bash
$ tracert unixwinbsd.site
```

Jika fungsi ini tidak cukup bagi Anda, Anda dapat menginstal Nmap di Windows.

Penelusuran dapat berguna untuk memahami struktur jaringan (misalnya, jaringan penyedia layanan Internet Anda), dan juga untuk memecahkan masalah transmisi data (misalnya, mengidentifikasi simpul yang tidak dilewati paket).

Fungsionalitas terkaya untuk penelusuran jaringan adalah program traceroute . Program lain juga berisi opsi menarik atau dapat digunakan sebagai alternatif jika tidak ada program lain yang tersedia atau jika menjalankan traceroute sebagai root.
