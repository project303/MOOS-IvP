# The IvPHelm as a MOOS Application

## 1   Helm IvP sebagai Aplikasi MOOS​ ​
Di bagian ini, helm dibahas dalam konteks identitasnya sebagai aplikasi MOOS - parameter konfigurasi MOOS, loop Iterate() , output ke konsol, dan output dalam konteks publikasi ke aplikasi lain yang berjalan di komunitas MOOS. Status helm dan status all-stop juga diperkenalkan karena keduanya merupakan deskripsi tingkat tertinggi terkait aktivitas helm.


### 1.1   Tinjauan Umum
Helm IvP diimplementasikan sebagai modul MOOS yang disebut pHelmIvP . Di permukaan, helm ini mirip dengan aplikasi MOOS lainnya - helm ini berjalan sebagai proses tunggal yang terhubung ke proses MOOSDB yang sedang berjalan yang berinteraksi hanya dengan antarmuka publikasi-berlangganan, seperti yang digambarkan dalam . Helm ini dikonfigurasi dari file perilaku, atau file .bhv , sebagai tambahan dari file MOOS yang digunakan untuk mengonfigurasi aplikasi MOOS lainnya. Helm terutama menerbitkan aliran informasi yang stabil yang menggerakkan platform, biasanya mengenai arah, kecepatan, atau kedalaman yang diinginkan. Helm juga dapat menerbitkan informasi yang menyampaikan aspek status otonomi yang mungkin berguna untuk memantau, men-debug, atau memicu algoritme lain baik di dalam helm atau dalam proses MOOS lainnya. Helm dapat dikonfigurasi untuk menghasilkan keputusan atas hampir semua ruang keputusan yang ditentukan pengguna.

> **Gambar 1.1: Aplikasi pHelmIvP MOOS**\
> Helm IvP diimplementasikan sebagai aplikasi MOOS pHelmIvP . Helm dikonfigurasikan dengan dua berkas - berkas misi dan berkas perilaku. Setelah diluncurkan, helm terhubung ke MOOSDB bersama dengan aplikasi MOOS lain yang menjalankan fungsi lain. Informasi yang mengalir ke helm mencakup informasi sensor dan masukan perintah dan kontrol. Helm menghasilkan perintah untuk mengendalikan kendaraan bersama dengan informasi status lain yang dihasilkan oleh perilaku aktif.

Helm berlangganan informasi sensor atau informasi lain yang dibutuhkannya untuk membuat keputusan. Informasi ini mencakup informasi navigasi mengenai posisi dan lintasan platform saat ini, informasi mengenai posisi atau keadaan kendaraan lain, atau informasi lingkungan. Informasi yang berlangganan ditentukan oleh perilaku itu sendiri, yang dikonfigurasi dalam berkas .bhv . Selain informasi sensor, helm juga menerima beberapa tingkat informasi perintah dan kontrol. Misalnya, dalam beberapa konfigurasi kendaraan laut, salah satu modul "MOOSApp Lainnya" dalam gambar adalah driver untuk modem akustik yang dapat digunakan untuk menyampaikan informasi perintah dan kontrol.

Kemudi memiliki beberapa deskripsi status tingkat tinggi yang informatif, status kemudi dan status berhenti total , yang dapat dibandingkan dengan pengoperasian mobil. Meluncurkan proses pHelmIvP MOOS dianalogikan dengan menyalakan mesin mobil. Menempatkan kemudi dalam mode DRIVE seperti memindahkan mobil dari "Parkir" ke "Berkendara". Dan status berhenti total mengacu pada apakah mobil mengerem untuk berhenti atau tidak. Analoginya dirangkum di bawah ini.

> **Gambar 1.2: Analogi Helm/Mobil**\
> Helm dan deskripsi status tingkat tingginya, status helm dan status berhenti total , memiliki analogi dengan pengoperasian mobil.


### 1.2   Negara Helm
Antarmuka tingkat tertinggi dengan helm tercermin dalam status helm . Status helm dapat memiliki satu dari dua nilai, parkir atau mengemudi (kecuali saat menggunakan helm siaga yang dijelaskan dalam ). Dalam mode mengemudi , helm kemungkinan sedang dalam proses menjalankan misi. Dalam mode parkir , helm sedang menunggu, kemungkinan karena diminta untuk menunggu, tetapi juga karena helm mungkin telah menyadari sesuatu yang salah (menghasilkan semua berhenti) dan kemudian menempatkan dirinya ke parkir sendiri, menunggu kesempatan untuk kembali ke status mengemudi . Di bagian ini kita membahas (a) bagaimana status helm diubah, (b) apa yang terjadi di helm saat diparkir, dan (c) bagaimana status helm diinisialisasi saat memulai. Pada titik waktu mana pun setelah helm diluncurkan, helm akan memposting variabel MOOS IVPHELM_STATE dengan nilai "DRIVE" , "PARK" atau "MALCONFIG" . Hal ini diposting pada tiap iterasi dan mendaftar untuk email ini adalah cara yang direkomendasikan oleh aplikasi MOOS lainnya untuk memantau detak jantung kemudi.


#### 1.2.1   Transisi Keadaan Helm   
Keadaan kemudi dapat diubah dengan menulis ke variabel MOOS MOOS_MANUAL_OVERIDE . Seperti yang digambarkan, nilai false , yang tidak peka huruf besar/kecil, mengubah keadaan kemudi menjadi DRIVE . Nilai true menempatkannya ke PARK . Ketika kemudi berubah dari DRIVE ke PARK , ia membuat satu publikasi lagi ke variabel keputusan kemudi, masing-masing dengan nilai nol. Ini disebut sebagai produksi posting all-stop , yang dibahas lebih rinci di bagian selanjutnya.

> **Gambar 1.3: Kondisi Helm IvP**\
> Kondisi helm memiliki nilai PARK atau DRIVE , tergantung pada bagaimana helm diinisialisasi dan email yang diterima oleh helm setelah dinyalakan pada variabel MOOS_MANUAL_OVERIDE . Helm juga dapat parkir sendiri jika peristiwa berhenti total telah terdeteksi.

Variabel MOOS_MANUAL_OVERIDE berisi kesalahan ejaan "override". Akan tetapi, variabel ini memiliki beberapa keberadaan lama di aplikasi MOOS lain seperti iRemote . Untuk menghindari situasi saat ada upaya untuk mengganti helm, tetapi permintaan diabaikan karena ejaan (yang benar), helm juga akan mematuhi permintaan transisi pada variabel MOOS_MANUAL_OVERRIDE yang dieja dengan benar . Akan tetapi, ini memiliki kekurangan karena kedua variabel ini mungkin memiliki nilai yang berbeda di MOOSDB. Ini bukan masalah, tetapi dapat membingungkan bagi seseorang yang mencoba menyimpulkan status helm dengan membuka cakupan pada MOOSDB, baik pada variabel yang salah atau dua variabel yang tidak sesuai. Dalam kasus ini, status helm akan disejajarkan dengan variabel dengan cap waktu publikasi terbaru. Bagaimanapun, cara terbaik untuk memantau status helm adalah dengan cakupan pada variabel MOOS IVPHELM_STATE , yang dipublikasikan oleh helm itu sendiri, atau menggunakan alat uHelmScope .

Kemudi juga dapat secara otomatis mengubah dirinya sendiri dari status DRIVE ke status PARK (tetapi tidak sebaliknya), dengan memposting peristiwa berhenti total. Peristiwa berhenti total dan kemudi dibahas secara terpisah di . Peristiwa berhenti total dihasilkan oleh kemudi setelah menemukan bahwa satu atau lebih kemungkinan kondisi kesalahan telah terdeteksi selama pelaksanaan normal iterasi kemudi. Jika kemudi parkir karena berhenti total, kemudi dapat dikembalikan ke status DRIVE oleh klien MOOS lain yang memposting MOOS_MANUAL_OVERIDE=false , tetapi ini tidak menjamin bahwa kemudi tidak akan parkir lagi segera jika kondisi yang sama yang menyebabkan peristiwa berhenti total tetap ada.


#### 1.2.2   Apa yang Terjadi dan Tidak Terjadi saat Helm Diparkir   
Ketika status helm adalah PARK , loop aplikasi MOOS berlanjut. OnNewMail() terus dipanggil dan email baru dibaca dan ditangani persis seperti yang akan terjadi jika status helm adalah DRIVE . Namun, loop Iterate() dipotong menjadi hampir tanpa operasi, dengan satu-satunya tindakan adalah keluaran karakter detak jantung ke konsol jika helm dikonfigurasi untuk melakukannya (). Tidak ada kode perilaku yang dipanggil sama sekali. Penghitung iterasi helm, indeks kunci dalam keluaran uHelmScope , juga ditangguhkan meskipun secara teknis loop Iterate() terus dipanggil.


#### 1.2.3   Inisialisasi Status Helm pada Waktu Peluncuran Proses  
Secara default, kemudi dikonfigurasikan agar awalnya dalam posisi PARK saat dinyalakan. Dengan menyetel parameter start_in_drive=true dalam blok konfigurasi berkas misi, kemudi memang akan berada dalam posisi DRIVE saat dinyalakan. Fitur ini ditemukan memiliki kegunaan praktis dalam operasi UUV untuk memungkinkan komputer otonomi di-boot ulang guna meluncurkan kemudi secara otomatis, dalam posisi DRIVE dan siap menerima perintah lapangan. Fitur ini harus digunakan dengan hati-hati, dan mungkin akan dihapus secara bertahap dalam rilis perangkat lunak berikutnya.


### 1.3   Helm All-Stop Events dan All-Stop Status
Peristiwa berhenti total adalah sesuatu yang membuat kendaraan berhenti total, dengan perintah kecepatan nol dan kedalaman nol. Status berhenti total hanyalah rangkaian yang menjelaskan mengapa kendaraan berhenti total. Terkadang berhenti total menunjukkan adanya masalah, misalnya, informasi sensor penting yang hilang. Terkadang kendaraan mungkin berhenti hanya sebagai bagian dari misi, misalnya, muncul ke permukaan untuk memperbaiki GPS. Pesan status berhenti total dapat digunakan untuk membedakan dua jenis situasi. Dalam analogi mobil, berhenti total setara dengan menginjak rem mobil dengan maksud untuk berhenti total. Peristiwa berhenti total akan menghasilkan hal berikut:

- Nilai nol akan diposting untuk semua variabel keputusan, DESIRED_SPEED=0 , DESIRED_DEPTH=0 , dst.
- Kemudi kemungkinan akan beralih ke PARK . Secara default kemudi dikonfigurasi untuk tetap berada di DRIVE saat berhenti total, tetapi jika dikonfigurasi dengan park_on_allstop = true , kemudi akan benar-benar parkir saat berhenti total.
- Alasan untuk berhenti total akan diposting ke variabel MOOS IVPHELM_ALLSTOP . Nilai variabel ini akan "dihapus" jika tidak ada kejadian berhenti total yang terjadi sejak kemudi memasuki status DRIVE .

Alasan untuk penghentian total mungkin karena:

- Tidak ada perilaku yang aktif. Helm sama sekali tidak memiliki pendapat tentang variabel keputusannya. Dalam kasus ini, berikut ini akan diposting: IVPHELM_ALLSTOP ="NothingToDo" .
- Beberapa perilaku aktif, tetapi keputusan tidak ada pada satu atau beberapa variabel keputusan wajib. Dalam kasus ini, berikut ini akan diposting: IVPHELM_ALLSTOP ="MissingDecVars" .
- Bila kendaraan diparkir karena pengesampingan manual, maka akan diposting hal berikut: IVPHELM_ALLSTOP ="ManualOverride" .
- Salah satu perilaku telah menentukan bahwa penghentian total diperlukan karena beberapa alasan. Misalnya, perilaku titik jalan yang tidak dapat menentukan posisi platform sendiri saat ini akan menyatakan penghentian total. Dalam kasus ini, berikut ini akan diposting: IVPHELM_ALLSTOP ="BehaviorError" .

Untuk mendapatkan wawasan lebih jauh tentang penghentian total yang disebabkan oleh kesalahan perilaku, sifat kesalahan dinyatakan dalam posting terpisah ke variabel MOOS BHV_ERROR . Ada kemungkinan lebih dari satu kesalahan perilaku terjadi pada iterasi kemudi tempat peristiwa penghentian total terjadi, dalam hal ini akan ada beberapa posting ke variabel BHV_ERROR . Saat kendaraan dalam posisi DRIVE dan beroperasi tanpa penghentian total, status penghentian total tercermin oleh IVPHELM_ALLSTOP ="clear" .


### 1.4   Parameter untuk Blok Konfigurasi MOOS pHelmIvP   
Parameter konfigurasi berikut ditetapkan untuk helm. Nama parameter tidak peka huruf besar/kecil.

**Daftar 1.1** - Parameter Konfigurasi untuk pHelmIvP.

| Variable	               | Deskripsi | 
| ------------------------ | ----------| 
| allow_park :|	Jika salah, kemudi tidak dapat ditempatkan di PARK . Parameter ini tidak wajib. Nilai default adalah benar. |
| behaviors :|	Nama dan lokasi berkas konfigurasi perilaku. Parameter ini tidak wajib, tetapi biasanya digunakan. Secara teknis, helm dapat diluncurkan dari baris perintah dan menyediakan berkas perilaku pada baris perintah. |
| ivp_behavior_dir :|	Direktori untuk mencari perilaku yang dimuat secara dinamis. Parameter ini tidak wajib, karena informasi direktori juga dapat ditangani menggunakan variabel lingkungan shell. |
| community :| Parameter MOOS global. Menentukan nama kepemilikan. Parameter ini wajib, tetapi disediakan di luar blok konfigurasi helm dan digunakan oleh aplikasi lain. |
| park_on_allstop :| Jika benar, kemudi akan berhenti sepenuhnya. Parameter ini tidak wajib dan default-nya adalah false. Bagian . |
| domain :|	Ruang keputusan untuk IvP Solver. Parameter ini wajib diisi. |
| hold_on_app :|	Daftar aplikasi MOOS yang harus ditunggu sebelum helm menerbitkan posting dari panggilan fungsi onHelmStart() perilaku . Tersedia setelah Rilis 17.7.x. |
| ok_skew :|	Toleransi usia, dalam detik, dari surat masuk sebelum ditolak karena dianggap terlalu lama. Parameter ini tidak wajib. |
| other_override_var :|	Parameter tersebut menamai variabel MOOS tambahan yang bertindak sebagai MOOS_MANUAL_OVERIDE . Parameter ini tidak wajib. |
| start_in_drive :|	Menentukan apakah kemudi dalam mode override saat start-up atau tidak. Parameter ini tidak wajib. Nilai default adalah false. Secton . |
| helm_prefix :|	Tambahkan awalan ke semua keluaran helm DESIRED_* . Misalnya helm_prefix=FOO_ akan menghasilkan FOO_DESIRED_SPEED dan seterusnya. Diperkenalkan setelah Rilis 19.8.x |
| verbose :|	Menentukan verbositas keluaran terminal - quiet , ringkas , atau verbose . Parameter ini tidak wajib. Defaultnya adalah verbose . |

