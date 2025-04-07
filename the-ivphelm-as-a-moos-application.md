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

#### 1.4.1   Parameter allow_park 
Opsional. Dengan menyetel parameter ini ke false , kemudi tidak dapat ditimpa (diparkir) secara manual setelah dimasukkan ke DRIVE . Ini dapat berbahaya dan harus dipertimbangkan dengan saksama, sehingga defaultnya adalah true . Opsi ini diimplementasikan berdasarkan pengalaman dalam meluncurkan misi otonomi UUV dan mencegah parkir yang tidak disengaja karena login jarak jauh ke kendaraan. Ada kecenderungan bagi beberapa pengguna untuk menggunakan iRemote saat login jarak jauh untuk berinteraksi dengan MOOS, dan iRemote memposting MOOS_MANUAL_OVERIDE =true saat peluncuran dan koneksi ke MOOSDB.


#### 1.4.2   Parameter behaviors    
Opsional (semacam). Parameter ini menamai file behaviors, yaitu, berkas *.bhv , pada sistem file lokal tempat helm behaviors dibaca. Lebih dari satu file dapat ditetapkan pada baris terpisah, dan helm akan membaca semua berkas hampir seolah-olah berkas tersebut merupakan satu berkas tunggal. Secara teknis ini merupakan parameter opsional karena berkas perilaku dapat disediakan pada baris perintah. Berkas perilaku harus ditetapkan melalui satu cara atau yang lain. Jika berkas perilaku ditetapkan pada baris perintah dan blok konfigurasi pHelmIvP dengan parameter ini, keduanya akan digunakan untuk mengonfigurasi perilaku helm.


#### 1.4.3   Parameter behavior_dir   
Opsional. Parameter ini menamai direktori dalam sistem file lokal tempat helm mencari behaviors yang dapat dimuat secara dinamis (berbeda dengan set default yang dibangun secara statis pada helm). Penulis yang melengkapi helm dengan perilaku mereka sendiri perlu menentukan lokasi perilaku tersebut dengan parameter ini. Lebih dari satu baris dapat diberikan, masing-masing menentukan lokasi direktori yang berbeda.


#### 1.4.4   Parameter community    
Parameter ini ditetapkan pada level "global" di luar blok konfigurasi proses MOOS mana pun. Helm membaca parameter ini dan menggunakan nilainya sebagai nama yang dikaitkan dengan "kepemilikan". Ini adalah parameter wajib.


#### 1.4.5   Parameter park_on_allstop   
Opsional. Dengan menyetel parameter ini ke true , kemudi akan berhenti saat terjadi peristiwa berhenti total. Setelan default adalah false .


#### 1.4.6   Parameter domain   
Wajib. Parameter ini menentukan ruang keputusan helm. Terdiri dari satu baris per variabel keputusan. Setiap baris berisi daftar empat bidang yang dipisahkan titik dua. Bidang pertama adalah nama variabel domain, bidang kedua adalah nilai batas bawah, bidang ketiga adalah nilai batas atas, dan bidang keempat adalah jumlah titik dalam domain. Misalnya domain = speed:0:3:16 menunjukkan variabel domain yang disebut "speed", dengan batas bawah dan atas masing-masing 0 dan 3 meter/detik. Karena ada 16 titik, pilihan kecepatannya adalah 0, 0,2, 0,4, ..., 2,8, 3,0. Helm mengharuskan keputusan dibuat pada semua variabel yang tercantum pada setiap iterasi loop kontrol. Jika suatu variabel digunakan oleh beberapa perilaku tetapi tidak selalu terlibat dalam semua keputusan, variabel tersebut dapat dideklarasikan sebagai opsional. Misalnya domain=speed:0:3:16:optional .


#### 1.4.7   Parameter ok_skew   
Opsional. Parameter ini menetapkan kemiringan yang diizinkan yang ditoleransi oleh helm untuk menerima pesan email masuk. Jika kemiringan jam terdeteksi lebih besar dari nilai ini, pesan akan diabaikan. Pemeriksaan kemiringan dapat dinonaktifkan dengan menetapkan ok_skew=any . Nilai default adalah 60 detik.


#### 1.4.8   Parameter other_override_var   
Opsional. Parameter ini menamai variabel MOOS yang akan dianggap oleh helm sebagai sinonim dengan dua variabel default yang diterima untuk penggantian manual, MOOS_MANUAL_OVERRIDE , dan kesalahan ejaan lama dari variabel ini, MOOS_MANUAL_OVERIDE .


#### 1.4.9   Parameter start_in_drive   
Opsional. Parameter ini disetel ke true atau false . Nilai default adalah false karena kemudi biasanya dimulai dalam PARK dan perlu menerima email MOOS pada variabel MOOS_MANUAL_OVERIDE dengan nilai variabel ini disetel ke false . Saat start_in_drive disetel ke true , kemudi berada dalam status DRIVE saat dinyalakan. Masalah status kemudi dibahas lebih rinci di .


#### 1.4.10   Parameter hold_on_app   
Opsional. Tersedia setelah Rilis 17.7.x. Parameter ini menamai satu atau beberapa aplikasi MOOS yang akan dipantau helm dalam daftar DB_CLIENTS . Setelah melihat setiap aplikasi yang diberi nama setidaknya satu kali, helm akan merilis semua email yang dihasilkan oleh perilaku melalui fungsi onHelmStart() mereka .

Ini mungkin berguna untuk perilaku yang menghasilkan informasi konfigurasi ke aplikasi pendukung lainnya. Contohnya termasuk aplikasi pBasicContactMgr dan pTaskManager . Aplikasi ini perlu diberi tahu oleh perilaku tentang sifat peringatan yang mungkin dihasilkan, yang berpotensi menghasilkan perilaku yang dihasilkan. Jika perilaku menghasilkan beberapa posting dari variabel MOOS yang sama, dan posting tersebut terjadi sebelum aplikasi penerima yang dimaksud online (terhubung ke MOOSDB), maka mereka mungkin hanya menerima bagian terakhir dari email. Dengan utilitas ini, helm akan menunggu hingga semua aplikasi yang disebutkan terhubung sebelum memposting email onHelmStart() .

Parameter dapat mengambil daftar aplikasi yang dipisahkan koma, atau dapat menyediakan beberapa baris. Dua gaya berikut ini setara:

> hold_on_app = pBasicContactMgr, pTaskManager

Dan

> hold_on_app = pBasicContactMgr
> hold_on_app = pTaskManager


#### 1.4.11   Parameter verbose    
Opsional. Parameter ini memengaruhi seberapa banyak informasi yang ditulis ke terminal pada setiap iterasi helm. Nilai yang mungkin adalah verbose , terse , atau quiet . Pengaturan verbose akan menulis laporan helm singkat ke terminal pada setiap iterasi. Dengan pengaturan terse, output minimal akan dihasilkan, karakter '*' saat tidak menghasilkan perintah helm, dan karakter '$' saat aktif dan sehat. Dengan pengaturan quiet , tidak ada output sama sekali yang akan ditulis ke terminal. Nilai default adalah terse . Pengaturan ini dapat diubah setelah helm dimulai dengan mengubah nilai \mvar{HELM_VERBOSE} di MOOSDB.


#### 1.4.12   Contoh Blok Konfigurasi pHelmIvP MOOS 
Di bawah ini adalah contoh blok konfigurasi untuk helm.

**Daftar 1.2** - Contoh blok konfigurasi pHelmIvP .

```
   1 //-------- pHelmIvP configuration block  -------------
   2 ProcessConfig = pHelmIvP
   3 {
   4   AppTick    = 4   // Defined for all MOOS processes
   5   CommsTick  = 4   // Defined for all MOOS processes
   6
   7   domain     = course:0:359:360
   8   domain     = speed:0:3:16
   9   domain     = depth:0:500:101
  10   	
  11   behaviors  = foobar.bhv  
  12   verbose    = terse
  13   ok_skew    = ANY
  14
  15   start_in_drive  = false
  16   allow_park      = true
  17   park_on_allstop = false
  18 }
```

Parameter **AppTick** dan **CommsTick** ditetapkan untuk semua proses MOOS dan menentukan frekuensi proses helm beriterasi dan berkomunikasi dengan MOOSDB. Parameter komunitas tidak disertakan dalam blok konfigurasi karena ditetapkan pada tingkat global dalam berkas misi.



### 1.5   Meluncurkan Helm dan Output ke Jendela Terminal 
Helm dapat diluncurkan baik langsung dari baris perintah, atau dari dalam Antler. Pada baris perintah, penggunaannya adalah sebagai berikut:

```
   Usage: pHelmIvP file.moos [file.bhv]...[file.bhv]     
          [--help|-h] [--version|-v]                     

   [file.moos] Nama file untuk mendapatkan parameter konfigurasi MOOS.   
   [file.bhv] Nama file untuk mendapatkan parameter konfigurasi Helm IvP.
   [-v] Keluarkan nomor versi dan keluar.           
   [-h] Keluarkan informasi penggunaan ini dan keluar.   
```

Jika tidak ada berkas perilaku yang ditentukan dalam berkas .moos maka berkas perilaku harus diberikan pada baris perintah. Beberapa berkas perilaku dapat diberikan. Urutan argumen tidak menjadi masalah - argumen baris perintah yang diakhiri dengan .bhv akan dibaca sebagai berkas perilaku, dan yang diakhiri dengan .moos sebagai berkas MOOS. Spesifikasi berkas perilaku juga dapat dibagi antara referensi dalam berkas .moos dan baris perintah. Spesifikasi duplikat dari satu berkas akan diabaikan begitu saja. Output awal yang umum untuk terminal ditunjukkan di bawah ini.


**Daftar 1.3** - Contoh keluaran awal yang dihasilkan oleh proses pHelmIvP.
```
  0  ****************************************************
  1  *                                                  *
  2  *       This is MOOS Client                        *
  3  *       c. P Newman 2001                           *
  4  *                                                  *
  5  ****************************************************
  6  
  7  ---------------MOOS CONNECT-----------------------
  8    contacting a MOOS server localhost:9000 -  try 00001 
  9    Contact Made
 10    Handshaking as "pHelmIvP"
 11    Handshaking Complete
 12    Invoking User OnConnect() callback...ok
 13  --------------------------------------------------
 14  
 15  The IvP Helm (pHelmIvP) is starting....
 16  Loading behavior dynamic libraries....
 17      Loading directory: /Users/mikerb/project-colregs/src/lib_behaviors-colregs
 18  Loading behavior dynamic libraries - FINISHED.
 19  Number of behavior files: 1
 20  Processing Behavior File: bravo.bhv  START
 21      Successfully found file: bravo.bhv
 22      InitializeBehavior: found static behavior BHV_Loiter
 23      InitializeBehavior: found static behavior BHV_Loiter
 24      InitializeBehavior: found static behavior BHV_Waypoint
 25      InitializeBehavior: found static behavior BHV_Timer
 26  Processing Behavior File: bravo.bhv  END
 27  pHelmIvP is Running:
 28  	 AppTick   @ 4.0 Hz
 29  	 CommsTick @ 4 Hz
 30  	 Time Warp @ 1.0 
 31  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
```

Output pada baris 0-13 adalah output standar yang dihasilkan oleh proses MOOS yang diluncurkan dan berhasil dihubungkan ke MOOSDB yang sedang berjalan. Baris 15-30 adalah output awal yang dihasilkan khusus untuk helm dan penggunaan pengguna tertentu. Perilaku yang digunakan oleh helm bersifat statis atau dinamis. Perilaku statis dikompilasi ke dalam pHelmIvP yang dapat dieksekusi. Perilaku dinamis dibawa masuk pada waktu proses melalui pustaka bersama yang dikompilasi secara terpisah. Helm mencari variabel lingkungan IVP_BEHAVIOR_DIRS untuk daftar direktori yang dipisahkan titik dua untuk mencari pustaka bersama. Jika variabel ini tidak disetel, atau jika satu atau beberapa direktori bukan direktori yang sah, pesan kesalahan akan menunjukkannya di antara baris 16 dan 18 di . Jenis kesalahan ini mungkin tidak benar-benar menjadi masalah jika perilaku yang ditetapkan dalam berkas perilaku semuanya dapat ditemukan dengan sukses.

Untuk setiap berkas perilaku yang ditentukan, informasi yang ditampilkan pada baris 20-26 dihasilkan ke terminal. Untuk setiap konfigurasi perilaku dalam berkas .bhv tertentu , satu baris dikeluarkan seperti pada baris 22-25 yang menunjukkan bahwa jenis perilaku dikenali dan dikonfigurasi dengan benar. Satu perilaku yang tidak dikenali atau konfigurasi yang tidak tepat akan menghasilkan (a) pesan kesalahan yang menunjukkan nomor baris dan nama berkas yang bermasalah, (b) keluaran baris yang bermasalah sebenarnya, dan (c) pemutusan segera proses dari MOOSDB dan keluar. (Kiat: Jika helm diluncurkan dengan Antler, kesalahan selama memulai akan mengakibatkan penutupan jendela konsol pHelmIvP yang membuatnya sulit untuk menangkap keluaran kesalahan yang berguna untuk debugging. Dalam kasus ini, helm harus diluncurkan di luar Antler di jendela terminalnya sendiri.)

Output pada baris 31 dari , serangkaian karakter tanda dolar, menunjukkan untuk setiap karakter, penyelesaian satu iterasi helm - output detak jantung. Ini adalah output ketika parameter verbose ditetapkan ke pengaturan default terse . Ketika ditetapkan ke quiet tidak ada output yang dihasilkan sama sekali. Ketika ditetapkan ke verbose , laporan multi-baris pendek dihasilkan untuk setiap iterasi. Contoh ditunjukkan di bawah ini dalam :

**Daftar 1.4** - Contoh laporan iterasi helm yang dihasilkan oleh helm aktif.
```
  1  Iteration: 161  ******************************************
  2  Helm Summary  ---------------------------
  3  loiter_a did NOT produce an obj-function
  4  loiter_b produces obj-function - time:0.00 pcs: 9.00000 pwt: 100.00000
  5  waypt_return did NOT produce an obj-function
  6  loiter_timer did NOT produce an obj-function
  7  Number of Objective Functions: 1
  8  DESIRED_SPEED:  2.10
  9  DESIRED_COURSE:  145.00
 10  (End) Iteration: 161  ******************************************
```

Pada setiap iterasi, Ringkasan Helm menunjukkan perilaku mana yang menghasilkan fungsi objektif (baris 2-5), dan untuk perilaku yang menghasilkan fungsi objektif, ringkasan menunjukkan waktu CPU yang dibutuhkan untuk menghasilkan fungsi, jumlah bagian dalam fungsi IvP linier bagian demi bagian, dan bobot prioritasnya. Setelah ini, keputusan yang diberikan untuk iterasi saat ini dikeluarkan dengan satu baris per variabel keputusan (baris 7-8). Ini adalah ringkasan yang sangat tipis tentang apa yang terjadi di dalam helm dan perlu dicatat bahwa alat uHelmScope jauh lebih cocok untuk memantau aktivitas helm dan melakukan debugging.


### 1.6   Publikasi dan Langganan untuk IvP Helm
IvP Helm, seperti proses MOOS lainnya, dapat ditentukan dalam hal antarmukanya ke MOOSDB, yaitu, variabel apa yang dipublikasikan dan variabel apa yang dilangganinya. Tidak mungkin untuk memberikan spesifikasi lengkap di sini karena helm terdiri dari perilaku, dan sarana untuk menyertakan sejumlah perilaku pihak ketiga. Setiap perilaku dapat memposting pasangan variabel-nilai, yang dipublikasikan ke MOOSDB oleh helm atas nama perilaku di akhir iterasi. Demikian pula, setiap perilaku dapat mendeklarasikan ke helm sejumlah variabel MOOS yang ingin didaftarkan helm atas namanya. Kecuali variabel-variabel ini, yang dipublikasikan dan dilanggani oleh helm atas nama perilaku individual, bagian ini membahas bagian yang tersisa dari antarmuka publikasi - langganan helm.


#### 1.6.1   Variabel yang diterbitkan oleh IvP Helm   
Variabel yang diterbitkan oleh IvP Helm dirangkum di bawah ini.

- **IVPHELM_SUMMARY** : Diproduksi pada setiap iterasi helm untuk dikonsumsi oleh aplikasi uHelmScope . Ini berisi informasi tentang iterasi helm saat ini mengenai jumlah fungsi IvP yang dibuat, waktu pembuatan, waktu penyelesaian, perilaku mana yang aktif, berjalan, diam, dan keputusan yang akhirnya dihasilkan selama iterasi. Ringkasan tidak menyertakan setiap komponen dalam setiap ringkasan. Komponen yang nilainya tidak berubah sejak ringkasan sebelumnya dihilangkan dari ringkasan saat ini. Hal ini dimotivasi oleh tujuan untuk mengurangi jejak berkas log untuk helm.
- **IVPHELM_STATEVARS** : Diproduksi secara berkala oleh helm untuk dikonsumsi oleh aplikasi uHelmScope . Berisi daftar variabel MOOS yang dipisahkan koma yang terlibat dalam prasyarat perilaku apa pun, yaitu, variabel yang memengaruhi status perilaku yang dijalankan.
- **IVPHELM_DOMAIN** : Diproduksi sekali oleh kemudi saat start-up untuk dikonsumsi oleh aplikasi uHelmScope . Berisi spesifikasi Domain IvP yang digunakan oleh kemudi.
- **IVPHELM_MODESET** : Diproduksi sekali oleh kemudi saat start-up untuk dikonsumsi oleh aplikasi uHelmScope . Berisi spesifikasi Deklarasi Mode Hirarkis, jika ada, yang digunakan oleh kemudi.
- **IVPHELM_STATE** : Ditulis oleh kemudi pada setiap iterasi aplikasi pHelmIvP MOOS, terlepas dari apakah kemudi dalam status DRIVE atau tidak. (lihat ). Baik "DRIVE" atau "PARK" . Ini adalah variabel MOOS yang direkomendasikan untuk dianggap sebagai indikator "detak jantung" kemudi.
- **HELM_IPF_COUNT** : Diproduksi pada setiap iterasi helm. Berisi jumlah fungsi IvP yang terlibat dalam penyelesai pada iterasi saat ini.
- **CREATE_CPU** : Waktu CPU dalam detik yang digunakan secara total oleh semua perilaku pada iterasi saat ini untuk membangun fungsi IvP.
- **LOOP_CPU** : Waktu CPU dalam detik yang digunakan oleh pemecah IvP dalam iterasi helm saat ini.
- **BHV_IPF** : Helm akan menerbitkan variabel ini untuk setiap perilaku aktif dalam iterasi saat ini. Variabel ini berisi representasi string dari fungsi IvP yang dihasilkan oleh perilaku tersebut. Variabel ini digunakan untuk visualisasi oleh aplikasi uFunctionVis , dan untuk pencatatan dan pemutaran serta analisis selanjutnya.
- **PLOGGER_CMD** : Variabel ini diterbitkan dengan nilai di bawah ini untuk memastikan bahwa aplikasi pLogger mencatat file .bhv bersama dengan file log data lainnya dan file .moos .

>   "COPY_FILE_REQUEST = nama_file.bhv"

- **DESIRED_*** : Setiap variabel keputusan dalam IvPDomain yang disediakan dalam konfigurasi helm akan memiliki posting terpisah yang diawali oleh DESIRED_ seperti dalam DESIRED_SPEED . Satu pengecualian adalah bahwa variabel course akan diubah menjadi heading karena alasan lama.
- **BHV_WARNING** : Meskipun variabel ini mungkin tidak pernah diposting, variabel ini adalah variabel MOOS default yang digunakan saat suatu perilaku memposting peringatan. Peringatan mungkin tidak berbahaya tetapi perlu dipertimbangkan.
- **BHV_ERROR** : Meskipun variabel ini mungkin tidak pernah diposting, ini adalah variabel MOOS default yang digunakan saat suatu perilaku memposting apa yang dianggapnya sebagai kesalahan fatal - kesalahan yang akan ditafsirkan oleh helm sebagai permintaan untuk menghasilkan padanan dari ALL-STOP.

Selain variabel-variabel di atas, helm akan mengeposkan setiap pasangan variabel-nilai atas nama perilaku yang membuat permintaan. Ini termasuk endflags, runflags, idleflag, activeflags, dan inactiveflags.


#### 1.6.2   Variabel yang Dilanggan oleh IvP Helm   
Variabel yang berlangganan oleh IvP Helm dirangkum di bawah ini:

- **MOOS_MANUAL_OVERIDE** : Bila diatur ke true , biasanya oleh aplikasi pihak ketiga seperti iRemote , atau dari komunikasi perintah dan kontrol, kemudi dapat melepaskan kendali. Jika kemudi dikonfigurasi dengan active_start = true , kemudi tidak akan melepaskan kendali (ini dapat diubah).
- **HELM_VERBOSE** : Mempengaruhi keluaran konsol yang dihasilkan oleh helm. Nilai yang sah adalah verbose , singkat , atau quiet . Lihat Section 1.5.
- **HELM_MAP_CLEAR** : Saat diterima, helm akan menghapus peta internal yang digunakan untuk menekan posting duplikat yang berulang. Lihat Section 1.8.

Selain variabel-variabel di atas, helm akan berlangganan untuk setiap pasangan variabel-nilai atas nama perilaku yang membuat permintaan. Ini termasuk, tetapi tidak terbatas pada, variabel yang terlibat dalam kondisi dan memperbarui parameter yang tersedia secara umum untuk semua perilaku.


### 1.7   Menggunakan Standby Helm  
Kemudi siaga mengacu pada peluncuran kemudi kedua yang berjalan di samping kemudi utama lain yang dikonfigurasi secara normal. Hal ini dilakukan untuk mengurangi risiko yang terkait dengan kemungkinan kegagalan kemudi utama. Kedua kemudi tersebut adalah contoh aplikasi pHelmIvP MOOS yang dikonfigurasi dengan misi dan/atau perilaku yang berbeda. Agaknya kemudi siaga dikonfigurasi dengan serangkaian perilaku yang lebih sederhana dan lebih konservatif yang difokuskan pada pemulihan kendaraan yang aman. Meskipun ketentuan umumnya dibuat selama pengoperasian kendaraan untuk mendeteksi detak jantung kemudi yang hilang, dalam situasi seperti pengoperasian UUV di bawah es, tidak dapat diterima untuk hanya menghentikan kendaraan dan muncul ke permukaan. Upaya harus dilakukan untuk melaksanakan misi yang lebih sederhana untuk mengembalikan kendaraan ke lokasi yang aman untuk pemulihan.

Penggunaan helm siaga semudah menambahkan blok konfigurasi kedua dalam berkas konfigurasi .moos . Contoh misi Kilo menunjukkan misi yang menggunakan helm siaga. Mendeklarasikan helm kedua sebagai helm siaga memerlukan satu baris tambahan dalam bentuk

>   SIAGA = N

di mana **N** adalah jumlah detik yang dibutuhkan oleh kemudi siaga untuk menoleransi detak jantung kemudi utama yang tidak ada sebelum mengambil alih kendali dari kemudi utama. Setelah pengambilalihan kemudi siaga dipicu, pengambilalihan tersebut tidak dapat dibatalkan.


#### 1.7.1   Dua Jenis Kegagalan Helm , Penyebab , dan Deteksi
Apa yang dimaksud dengan helm crash , dan bagaimana itu bisa terjadi? Crash adalah hasil dari kode yang menyebabkan proses berhenti tiba-tiba dan tanpa peringatan. Baris sesederhana assert(0); dalam kode akan cukup untuk mereplikasi crash, tetapi penyebab crash bisa jadi jauh lebih halus karena merujuk pada memori yang tidak dialokasikan dengan benar dan sebagainya. Jenis kegagalan helm lainnya adalah helm yang hang . Ini merujuk pada skenario saat helm memasukkan sepotong kode yang membutuhkan waktu terlalu lama atau tidak pernah menyelesaikan eksekusi. Ini bisa jadi sepele seperti mencapai baris while(1); di suatu tempat dalam kode. Kedua jenis kegagalan ini serius. Terlepas dari kenyataan bahwa kami belum pernah mengalami kegagalan ini dalam latihan lapangan apa pun pada kendaraan apa pun, hilangnya proses helm secara tiba-tiba harus dipertimbangkan dan ditangani seanggun mungkin.

Bagaimana kegagalan helm dideteksi? Helm menghasilkan pesan detak jantung pada setiap iterasi dengan memposting ke variabel IVPHELM_STATE . Jika helm dikonfigurasi untuk beriterasi empat kali per detik, kecurigaan kegagalan dapat dimulai kapan saja setelah seperempat detik berlalu tanpa posting ke variabel ini. Di bawah beban CPU helm yang khas dengan perilaku standar umum, interval seperempat detik seharusnya cukup untuk menyelesaikan iterasi. Namun, ada faktor tambahan yang perlu dipertimbangkan. Mungkin ada proses lain pada mesin yang mendominasi CPU, sehingga menantang helm untuk melakukan pekerjaannya dalam waktu yang diharapkan. Mungkin ada kalkulasi perilaku periodik, seperti menghitung ulang jalur titik jalan yang panjang, yang dapat menyebabkan lonjakan siklus CPU yang dibutuhkan oleh perilaku tersebut. Sebagai aturan praktis, interval waktu tanpa adanya detak jantung, bisa dibilang harus dua detik atau lebih sebelum mendeklarasikan kegagalan helm. Interval ini dapat dikonfigurasi secara langsung, sebagai parameter N , di baris konfigurasi standby =N .


#### 1.7.2   Penanganan Helm Crash dengan Menggunakan Standby Helm   
Kecelakaan kemudi (helm crash) merupakan kasus kegagalan yang lebih mudah ditangani dari kedua kasus tersebut. Proses tersebut hilang begitu saja dan tidak lagi dipublikasikan ke MOOSDB. Tentu saja proses MOOS lainnya, termasuk kemudi siaga, tidak memiliki cara untuk mengetahui melalui kotak surat MOOS mereka apakah kemudi utama hilang atau hanya tertunda. Dalam kedua kasus tersebut, urutan kejadiannya adalah sebagai berikut:

1. Helm siaga mendeteksi tidak adanya detak jantung selama lebih dari N detik.
2. Pada iterasi yang sama, kemudi siaga mem-posting IVPHELM_STATE = DRIVE+ alih-alih IVPHELM_STATE = STANDBY yang telah di-posting pada setiap iterasi hingga saat ini. Ia juga mulai mem-posting keputusan kemudi yang diinginkan pada iterasi ini.

Standby Helm memiliki semua fungsi yang sama dengan helm utama, modulo perilaku dan konfigurasi misi. Helm akan berada dalam status DRIVE hanya jika tidak diganti secara manual. Jika MOOS_MANUAL_OVERRIDE=true saat helm siaga mengambil alih, awalnya helm akan berada dalam status PARK , bukan DRIVE . Helm akan mengirim IVPHELM_STATE =PARK+ . Perhatikan bahwa helm siaga akan menambahkan karakter '+' ke string status helm untuk membantu aplikasi lain dan pengguna memahami bahwa status helm yang dikirim berasal dari helm siaga yang telah mengambil alih kendali.

Misi contoh Kilo berjalan melalui simulasi helm crash.


#### 1.7.3   Menangani Hung Helm dengan Menggunakan Standby Helm   
Menangani kemudi yang *hang* memerlukan sedikit pertimbangan. Bagian yang sulit adalah bahwa sangat mungkin kemudi yang tergantung itu hanya tergantung sementara , dan pada suatu saat kemudi itu akan terlepas dan dapat beroperasi untuk satu iterasi seolah-olah kemudi itu masih memegang kendali. Dua kemudi, dengan misi yang berbeda, keduanya mengira mereka yang memegang kendali! Urutan kejadiannya dirangkum di bawah ini:

1. Standby Helm mendeteksi tidak adanya heartbeat selama lebih dari N detik.
2. Pada iterasi yang sama, kemudi siaga mem-posting IVPHELM_STATE = DRIVE+ alih-alih IVPHELM_STATE = STANDBY yang telah di-posting pada setiap iterasi hingga saat ini. Ia juga mulai mem-posting keputusan kemudi yang diinginkan pada iterasi ini.
3. Beberapa waktu kemudian, helm primer asli menyelesaikan iterasinya dan memposting keputusan helm, misalnya, serangkaian posting ke variabel keputusan helm, DESIRED_HEADING , dsb.
4. Pada iterasi berikutnya dari helm utama asli, ia memperhatikan bahwa helm lain telah memposting detak jantung non-siaga, misalnya, posting ke IVPHELM_STATE yang tidak sama dengan "STANDBY" .
5. Helm utama asli mencatatkan all-stop dan IVPHELM_STATE = DISABLED . Ini adalah saat terakhir helm akan mencatatkan detak jantung atau keputusan helm.
5. Standby helm tetap memegang kendali dengan mengeluarkan keputusan pimpinan, tanpa menyadari pencerahan dan pengaruh sementara dari pimpinan utama sebelumnya.

Masalah utama dalam skenario ini adalah bahwa helm utama asli benar-benar memposting keputusan helm saat terlepas dari suspensinya, meskipun helm siaga mungkin telah lama mengambil alih dan mungkin memposting serangkaian keputusan helm yang sepenuhnya bertentangan dengan keputusan yang diposting oleh helm utama yang terlepas dari suspensi yang baru saja terbangun.

Tidak ada asumsi yang dapat dibuat tentang proses MOOS yang mendengarkan urutan keputusan kemudi. Proses tersebut dapat berupa proses antarmuka muatan, atau pengontrol PID asli, atau beberapa proses lain yang bertanggung jawab untuk mengubah keputusan kemudi menjadi perintah aktuator tingkat rendah. Asumsi yang dibuat di sini adalah bahwa penyimpangan satu kali dalam urutan keputusan kemudi dapat ditoleransi. Penyimpangan ini tidak akan mengakibatkan aktuator rusak karena perubahan mendadak dalam urutan perintah seperti kecepatan tinggi, kecepatan nol, kecepatan tinggi.

Perlu dicatat juga bahwa kemudi utama asli, setelah bangun dan mengetahui bahwa ia tidak lagi memegang kendali, memang akan mengeluarkan satu keputusan kemudi lagi, yaitu keputusan berhenti total ( DESIRED_SPEED = 0 ). Hal ini dilakukan untuk memastikan bahwa keputusan berhenti total, yang mungkin telah diberlakukan oleh kemudi siaga, tidak digantikan oleh keluaran keputusan kemudi akhir kemudi utama asli yang dibuat setelah bangun.


#### 1.7.4   Aktivitas Standby Helm Saat Berdiri​   
Dalam status siaga, helm tidak akan melakukan apa pun dalam loop iterasinya selain mengeposkan karakter detak jantung ke terminal jika dikonfigurasi untuk melakukannya. Helm tidak akan memanggil perilaku apa pun untuk melakukan apa pun. Namun, helm akan membaca dan memproses semua email yang telah dilanggannya. Secara khusus, helm akan memantau pengeposan ke IVPHELM_STATE , dan akan memulai pengambilalihan saat tidak menerima email tersebut selama N detik. Perhatikan bahwa helm siaga juga menerbitkan ke variabel ini, IVPHELM_STATE = STANDBY , tetapi mengabaikan email yang berasal dari dirinya sendiri.

Helm juga akan membaca dan memproses semua email lain di kotak masuknya, memperbarui buffer informasi helm. Perhatikan bahwa buffer informasi juga menyimpan riwayat posting ke variabel tertentu sehingga biasanya perilaku dapat memproses beberapa posting jika beberapa posting dibuat ke MOOSDB di antara iterasi helm. Riwayat ini dihapus oleh helm di akhir setiap iterasi helm. Ketika helm dalam status siaga, ia juga akan menghapus riwayat setelah setiap iterasi meskipun perilaku belum diberi kesempatan untuk mengakses riwayat ini. Ini untuk memastikan bahwa riwayat buffer informasi tidak tumbuh tanpa batas saat dalam mode siaga. Misalnya, jika helm siaga dan primer keduanya dikonfigurasi dengan perilaku titik jalan dan helm primer mengunjungi setengah titik pada titik tabrakan helm, perilaku titik jalan helm siaga akan dimulai dengan titik jalan pertama.


#### 1.7.5   Aktivitas Helm Primer Setelah Take-Over   
Helm utama yang telah diambil alih disebut sebagai helm didsabled . Helm ini hanya akan memposting sekali IVPHELM_STATE = DISABLED . Proses helm tetap berjalan, tetapi tidak melakukan apa pun. Helm ini tidak membaca emailnya, dan loop iterate hanya memposting karakter detak jantung ( '!' ) ke terminal jika dikonfigurasi untuk melakukannya, dengan parameter konfigurasi helm verbose=terse .


### 1.8   Penyaringan Otomatis Publikasi Helm Duplikat Berturut-turut   
Helm tersebut menerapkan "filter duplikasi" untuk mengurangi secara drastis jumlah surat yang dikirim oleh helm atas nama perilaku. Filter ini telah diketahui mengurangi ukuran berkas log keseluruhan yang terlihat selama latihan di dalam air hingga 60-80%. Pengurangan pada level ini secara nyata memudahkan penggunaan alat analisis pasca-misi dan pengarsipan data. Sebagian besar filter ini beroperasi di balik layar untuk pengguna helm pada umumnya. Namun, pengetahuan tentang hal itu memang relevan bagi pengguna yang ingin menerapkan perilaku mereka sendiri, dan kami membahasnya di sini untuk menjelaskan sedikit apa yang ada di balik variabel HELM_MAP_CLEAR yang berlangganan helm, dan tercantum di atas dalam .


#### 1.8.1   Motivasi untuk Filter Duplikasi  
Motivasi utama penerapan filter duplikasi adalah untuk mengurangi jumlah surat yang tidak perlu yang dikirim oleh pimpinan atas nama perilaku, dan dengan demikian mengurangi ukuran berkas log dan memfasilitasi penanganan data pasca-misi. Yang kami maksud dengan tidak perlu adalah pasangan variabel-nilai yang berurutan yang sama persis di kedua bidang. Tentunya ada kasus ketika pengembang perilaku mungkin tidak menginginkan filter ini, dan ada cara sederhana untuk melewati filter untuk setiap kiriman. Namun dalam kebanyakan kasus, kiriman duplikat yang berurutan hanya berlebihan dan tidak perlu.


#### 1.8.2   Implementasi dan Penggunaan Filter Duplikasi   
Helm tersebut menyimpan dua map (STL map dalam C++), satu untuk data string dan satu untuk data numerik:

>    KEY --> StringValue
>    KEY --> DoubleValue

Kedua map tersebut sesuai dengan tipe pesan double dan string dalam MOOS. KEY biasanya adalah nama variabel MOOS. Di dalam implementasi perilaku, empat fungsi berikut tersedia:

```
     void postMessage(string nama variabel, string nilai, string kunci="");
     void postMessage(string nama variabel, nilai ganda, string kunci="");
     void postBoolMessage(string nama variabel, bool nilai, string kunci="");
     void postIntMessage(string nama variabel, nilai ganda, string kunci="");
```

Fungsi-fungsi ini tersedia dalam semua implementasi perilaku karena fungsi-fungsi ini didefinisikan dalam superkelas IvPBehavior, yang merupakan subkelas dari semua perilaku. Sebelum helm mengeposkan pesan ke MOOSDB, filter diterapkan dengan pemeriksaan sederhana pada petanya untuk menentukan apakah ada kecocokan nilai pada kunci yang diberikan. Jika kecocokan terjadi, pengeposan tidak akan dilakukan ke MOOSDB atas nama perilaku tersebut. Fungsi postIntMessage() hanyalah versi praktis dari fungsi postMessage() yang membulatkan nilai variabel ke bilangan bulat terdekat untuk lebih mengurangi pengeposan saat digabungkan dengan filter. postBoolMessage() akhirnya mengeposkan nilai string "true" atau "false" .

Nilai default dari parameter key adalah string kosong, dan dalam kebanyakan kasus parameter ini dapat dihilangkan tanpa menonaktifkan filter duplikasi. Ini karena KEY yang digunakan oleh pemanggil hanya bagian dari kunci yang sebenarnya digunakan oleh filter duplikasi. Kunci sebenarnya adalah gabungan dari (a) nama perilaku, (b) nama variabel, dan (c) kunci yang diteruskan oleh pemanggil. Jadi nilai default, string kosong, masih menghasilkan kunci yang layak digunakan oleh filter. Kunci tersebut ditambah dengan nama perilaku karena sering kali ada lebih dari satu perilaku yang mengeposkan pesan pada variabel yang sama. Parameter key opsional digunakan karena dua alasan. Pertama, parameter ini dapat digunakan untuk lebih membedakan pos dalam perilaku pada nama variabel yang sama. Kedua, ketika nilai key memiliki nilai khusus "repeatable" , maka tidak ada kunci yang digunakan dan filter duplikasi dinonaktifkan untuk pengeposan variabel tersebut.


Dua fungsi kenyamanan tambahan tersedia:

```
     void postRepeatableMessage(string nama variabel, string nilai);
     void postRepeatableMessage(string nama variabel, nilai ganda);
```

Posting postRepeatableMessage("FOO", 100) setara dengan postMessage("FOO", 100, "repeatable") .


#### 1.8.3   Menghapus Filter Duplikasi   
Kadang-kadang pengguna, atau aplikasi MOOS lain dalam komunitas yang sama dengan helm, mungkin ingin "menghapus" peta yang digunakan oleh helm untuk menerapkan filter duplikasinya. Ini dapat dilakukan dengan menulis ke variabel HELM_MAP_CLEAR , dengan nilai apa pun. Ini mungkin diperlukan karena alasan berikut. Misalkan aplikasi GUI berlangganan variabel VIEW_SEGLIST yang berisi daftar segmen garis untuk dirender. Jika aplikasi penampil diluncurkan setelah variabel dipublikasikan, aplikasi hanya akan menerima email terbaru pada variabel VIEW_SEGLIST . Mungkin ada publikasi untuk variabel ini, yang dibuat sebelum publikasi terbaru, yang relevan dengan aplikasi GUI pada waktu peluncuran. Publikasi untuk variabel VIEW_SEGLIST tersebut mungkin bukan yang terbaru dari perspektif MOOSDB, tetapi mungkin yang terbaru dari perspektif perilaku tertentu di helm. Dengan menghapus filter, setiap perilaku memiliki kesempatan untuk sekali lagi memiliki semua posting nilai variabelnya yang dibuat ke MOOSDB. Dalam aplikasi pMarineViewer , publikasi ke HELM_MAP_CLEAR dilakukan saat memulai. Menghapus filter hanya akan membuka jalan bagi posting berikutnya untuk variabel tertentu. Ini tidak akan mengakibatkan penerbitan konten peta yang digunakan oleh filter ke MOOSDB.


