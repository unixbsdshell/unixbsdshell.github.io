layout: post
title:  "Cara mengunduh Visual C++ Redistributable 2005-2022"
date:   2024-07-04 19:30:00 -0700
tags:   Windows
---


Paket Microsoft Visual C++ Redistributable menginstal komponen runtime dari pustaka Visual C++ yang diperlukan untuk menjalankan aplikasi yang dikembangkan dengan Visual C++ pada komputer tanpa Visual C++ yang terinstal. Metode integrasinya adalah SVCPACK (T13) dan juga dapat diinstal pada sistem aktif.

Paket Microsoft Visual C++ yang didistribusikan (Visual C++ Redistributable) berisi komponen yang diperlukan untuk menjalankan game dan program yang dikembangkan menggunakan versi Visual Studio yang sesuai dan, sebagai aturan, diperlukan untuk kesalahan seperti "Program tidak dapat diluncurkan" karena sistem tidak mendeteksi file DLL dengan nama yang dimulai dengan msvcr atau msvcp. Paling sering, komponen Visual Studio 2012, 2013, dan 2015 diperlukan, tetapi Anda dapat mengunduh dan menginstal paket Visual C++ redistributable versi 2005-2022.

## Unduh paket Visual C++ yang dapat didistribusikan ulang dari situs web Microsoft

Cara pertama untuk mengunduh komponen Visual C++ adalah cara resmi dan, karenanya, cara yang paling aman. Komponen berikut tersedia untuk diunduh (beberapa di antaranya dapat diunduh dengan berbagai cara).
Visual Studio 2015-2022-Saat menginstal kit ini, semua komponen Visual C++ 2015, 2017, 2019, dan 2022 yang dapat didistribusikan ulang diinstal dalam satu berkas penginstal.
- Visual Studio 2013 (Visual C++ 12.0).
- Visual Studio 2012 (Visual C++ 11.0).
- Visual Studio 2010 SP1.
- Visual Studio 2008 SP1

Urutan pemuatan komponen adalah sebagai berikut:
- Buka [halaman resmi Microsoft](https://learn.microsoft.com/ru-ru/cpp/windows/latest-supported-vc-redist) dan pilih komponen yang ingin Anda unduh.
- Untuk Visual C++ 2015-2022, cukup unduh dan instal langsung file vc_redist.x86.exe dan vc_redist.x64.exe untuk sistem x64, opsi x86 saja untuk sistem 32-bit atau vc_redist.arm64.exe untuk perangkat dengan prosesor ARM.
- Untuk komponen Visual C++ 2013, unduh berkas penginstal dari bagian "Paket Microsoft Visual C++ Redistributable untuk Visual Studio 2013".
- Untuk beberapa komponen (misalnya, untuk versi Visual C++ 2012), Anda akan diminta untuk masuk dengan akun Microsoft Anda. Namun, Anda tidak perlu melakukan ini — nanti di artikel ini, saya akan memberikan tautan untuk mengunduh langsung dari situs Microsoft tanpa harus masuk.

<figure class="figure w-100">
    <a target="_blank" href="https://raw.githubusercontent.com/unixbsdshell/unixbsdshell.github.io/refs/heads/main/img/ags-23/Versi%20terbaru%20Microsoft%20Visual%20C%20plus%20plus%20Redistributable.png">
    <img src="https://raw.githubusercontent.com/unixbsdshell/unixbsdshell.github.io/refs/heads/main/img/ags-23/Versi%20terbaru%20Microsoft%20Visual%20C%20plus%20plus%20Redistributable.png" class="img-fluid border" alt="Versi terbaru Microsoft Visual C++ Redistributable">
    </a>
    <figcaption class="figure-caption text-center">
        Versi terbaru Microsoft Visual C++ Redistributable Yang ada di situs Resmi Microsoft
    </figcaption>
</figure>
<br/>

Halaman terpisah untuk mengunduh paket Microsoft Visual C++ yang dapat didistribusikan ulang juga tersedia di situs web Microsoft:
- [Visual C++ 2013](https://support.microsoft.com/ru-ru/help/3179560/update-for-visual-c-2013-and-visual-c-redistributable-package) (di bagian kedua halaman terdapat tautan langsung untuk mengunduh versi x86 dan x64)
- [Visual C++ 2010](https://www.microsoft.com/ru-ru/download/details.aspx?id=26999)
- [Visual C++ 2008](https://www.microsoft.com/ru-ru/download/details.aspx?id=26368)
- [Visual C++ 2017 (x64)](https://go.microsoft.com/fwlink/?LinkId=746572)
- Visual C++ 2015 - halaman unduhan [pertama](https://www.microsoft.com/ru-ru/download/details.aspx?id=53840) dan [kedua](https://www.microsoft.com/ru-ru/download/details.aspx?id=52685) di situs web resmi
Setelah mengunduh komponen Visual C++ yang diperlukan, jalankan file yang diunduh dan lakukan seluruh proses instalasi.



