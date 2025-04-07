# Gambaran Umum Proyek Otonomi MOOS-IvP

## 1   Tinjauan Umum Proyek Otonomi MOOS - IvP

### 1.1   Latar Belakang Singkat MOOS -IvP   
MOOS ditulis oleh Paul Newman pada tahun 2001 untuk mendukung operasi dengan kendaraan laut otonom dalam program MIT Ocean Engineering dan MIT Sea Grant. Pada saat itu Newman adalah post-doc yang bekerja dengan John Leonard dan sejak itu bergabung dengan fakultas Mobile Robotics Group di Universitas Oxford. MOOS terus dikembangkan dan dipelihara oleh Newman di Oxford dan versi terbaru dapat ditemukan di themoos.org . Perangkat lunak MOOS yang tersedia dalam proyek MOOS-IvP mencakup snapshot kode MOOS yang didistribusikan dari Oxford. IvP Helm dikembangkan pada tahun 2004 untuk kontrol otonom pada kapal permukaan laut tak berawak, dan kemudian platform bawah air. Itu ditulis oleh Mike Benjamin sebagai post-doc yang bekerja dengan John Leonard, dan sebagai ilmuwan peneliti untuk Naval Undersea Warfare Center di Newport Rhode Island. IvP Helm adalah proses MOOS tunggal yang menggunakan optimasi multi-objektif untuk mengimplementasikan koordinasi perilaku.

#### Akronim
MOOS adalah singkatan dari "Mission Oriented Operating Suite" dan penggunaan awalnya adalah untuk kendaraan Bluefin Odyssey III milik MIT. IvP adalah singkatan dari "Interval Programming" yang merupakan model pemrograman matematika untuk optimasi multi-objektif. Dalam model IvP, setiap fungsi objektif adalah konstruksi linier sepotong-sepotong di mana setiap bagian adalah interval dalam N-Space. Model dan algoritme IvP disertakan dalam perangkat lunak IvP Helm sebagai metode untuk merepresentasikan dan merekonsiliasi keluaran perilaku helm. Istilah pemrograman interval terinspirasi oleh model pemrograman matematika dari pemrograman linier (LP) dan pemrograman integer (IP). Akronim semu IvP dipilih hanya dalam semangat ini dan untuk menghindari bentrokan akronim.


### 1.2   Sponsor MOOS -IvP
Pengembangan awal MOOS dan IvP kurang lebih merupakan produk sampingan infrastruktur dari penelitian lain yang disponsori dalam robotika (kebanyakan kelautan). Sponsor tersebut terutama adalah The Office of Naval Research (ONR), serta National Oceanic and Atmospheric Administration (NOAA). MOOS dan IvP saat ini didanai oleh Code 311 di ONR, Dr. Don Wagner dan Dr. Behzad Kamgar-Parsi. Pengujian dan pengembangan tugas kuliah di MIT selanjutnya didukung oleh Battelle, Mr. Mike Mellott. Sponsor Battelle sangat berperan dalam pengembangan dokumentasi dan materi kuliah daring. MOOS juga didukung di Inggris oleh EPSRC. Pengembangan awal IvP mendapat manfaat dari dukungan program In-house Laboratory Independent Research (ILIR) di Naval Undersea Warfare Center di Newport Inggris. Program ILIR didanai oleh ONR.


### 1.3   Tempat Mendapatkan Informasi Lebih Lanjut


#### 1.3.1   Situs Web dan Daftar Email
Ada dua situs web - situs web MOOS yang dikelola oleh Universitas Oxford, dan situs web MOOS-IvP yang dikelola oleh MIT. Pada saat artikel ini ditulis, situs-situs tersebut berada di URL berikut:

- situs web resmi themoos.org
- situs web resmi moos-ivp.org

Apa perbedaan konten antara kedua situs web tersebut? Seperti yang dibahas sebelumnya, MOOS-IvP, sebagai satu set perangkat lunak, merujuk pada perangkat lunak yang dikelola dan didistribusikan dari Oxford ditambah aplikasi MOOS tambahan termasuk IvP Helm dan pustaka perilaku. Bundel perangkat lunak yang dirilis di moos-ivp.org memang menyertakan perangkat lunak MOOS dari Oxford - biasanya versi rilis tertentu. Untuk perangkat lunak inti MOOS dan dokumentasi terkini tentang modul MOOS Oxford, situs web Oxford adalah sumbernya. Untuk informasi terkini tentang inti IvP Helm, perilaku, dan alat MOOS yang didistribusikan oleh MIT, situs web moos-ivp.org adalah sumbernya.

Ada dua milis yang terbuka untuk umum. Milis pertama diperuntukkan bagi pengguna MOOS, dan milis kedua diperuntukkan bagi pengguna MOOS-IvP. Jika topiknya terkait dengan salah satu modul MOOS yang didistribusikan dari situs web Oxford, milis yang tepat adalah milis "moosusers". Anda dapat bergabung dengan milis "moosusers" di URL berikut:

> https://lists.csail.mit.edu/mailman/listinfo/moosusers

Untuk topik yang terkait dengan IvP Helm atau modul yang didistribusikan di situs web moos-ivp.org yang bukan bagian dari distribusi Oxford MOOS (lihat halaman perangkat lunak di moos-ivp.org untuk bantuan dalam membedakannya), milis "moosivp" adalah pilihan yang tepat. Anda dapat bergabung dengan milis "moosivp" di URL berikut:

> https://lists.csail.mit.edu/mailman/listinfo/moosivp


#### 1.3.2   Dokumentasi
Dokumentasi tentang MOOS dapat ditemukan di situs web MOOS:

> situs web resmi themoos.org

Ini mencakup dokumentasi tentang arsitektur MOOS, pemrograman aplikasi MOOS baru serta dokumentasi tentang beberapa aplikasi penting seperti pAntler , pLogger , uMS , pShare , iRemote , iMatlab , pScheduler, dan banyak lagi. Dokumentasi tentang IvP Helm, behaviors, dan aplikasi MOOS terkait otonomi yang bukan dari Oxford dapat ditemukan di situs web www.moos-ivp.org di bawah tautan Dokumentasi.
