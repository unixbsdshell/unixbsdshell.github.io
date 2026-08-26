---
layout: post
title:  "Installing Proxmox VE 9 on Intel MacBook Pros"
date:   2025-08-29 12:00:00 -0700
tags:   laptops proxmox linux macbook-pro apple
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

Metode yang berbeda memiliki opsi khusus, bergantung pada protokol yang digunakan. Opsi metode ini dapat ditentukan menggunakan opsi -O . Beberapa opsi dipisahkan dengan koma (atau gunakan beberapa -Os pada baris perintah ). Masing-masing metode mungkin mempunyai pilihan khususnya sendiri atau mungkin tidak ada sama sekali.



