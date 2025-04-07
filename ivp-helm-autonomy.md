# Otonomi Helm IvP

## 1   IvP Helm Otonomi


### 1.1   Tinjauan Umum
Helm otonom pada dasarnya adalah mesin untuk pengambilan keputusan. IvP Helm menggunakan arsitektur berbasis perilaku untuk mengatur pengambilan keputusannya dan unik dalam cara mengatasi persaingan antara perilaku yang saling bersaing - helm ini melakukan optimasi multi-objektif pada output kolektif mereka menggunakan model pemrograman matematika yang disebut pemrograman interval.Di sini arsitektur IvP Helm dijelaskan dan cara untuk mengonfigurasinya dengan memberikan seperangkat perilaku dan seperangkat tujuan misi.


#### 1.1.1   Pengaruh Brooks , Stallman dan Dantzig pada IvP Helm    
Gagasan tentang arsitektur berbasis perilaku (behavior-based) untuk mengimplementasikan otonomi pada robot atau kendaraan tak berawak paling sering dikaitkan dengan Arsitektur Subsumsi karya Rodney Brooks, \cite{?} . Prinsip utama di jantung arsitektur Brooks dan bisa dibilang alasan utama daya tariknya bertahan, adalah gagasan bahwa sistem otonomi dapat dibangun secara bertahap . Khususnya, publikasi asli Brooks mendahului kedatangan perangkat lunak Open Source dan Free Software Foundation yang didirikan oleh Richard Stallman. Perangkat lunak Open Source bukanlah prasyarat untuk membangun sistem otonomi secara bertahap, tetapi memiliki kemampuan untuk mempercepat tujuan itu. Pengembangan sistem otonomi yang kompleks akan sangat diuntungkan jika kumpulan pengembang di meja itu besar dan beragam. Terlebih lagi jika mereka dapat berasal dari organisasi yang berbeda dengan mungkin bahkan yang paling longgar tumpang tindih dalam minat mengenai cara menggunakan produk akhir kolektif.

Seperti yang dibahas dalam Bagian ??? , isu kunci dalam otonomi berbasis perilaku adalah isu pemilihan tindakan, dan IvP Helm berbeda dalam hal ini dengan penggunaan optimasi multi-objektif dan pemrograman interval. Algoritme di balik pemrograman interval, serta istilah itu sendiri, dimotivasi oleh model pemrograman matematika, pemrograman linier, yang dikembangkan oleh George Dantzig, \cite{?} . Ide kunci dalam pemrograman linier adalah pilihan konstruksi matematika tertentu yang terdiri dari contoh masalah pemrograman linier - ia memiliki fleksibilitas ekspresif yang cukup untuk mewakili kelas besar masalah praktis, dan konstruksi dapat secara efektif dieksploitasi oleh metode simpleks untuk konvergen dengan cepat bahkan pada contoh masalah yang sangat besar. Konstruksi yang digunakan dalam pemrograman interval untuk mewakili keluaran perilaku (fungsi linier sepotong-sepotong) juga dipilih untuk memiliki fleksibilitas ekspresif yang cukup untuk menangani perilaku saat ini dan masa depan, dan karena peluang untuk mengembangkan algoritma solusi yang mengeksploitasi konstruksi linier sepotong-sepotong.


#### 1.1.2   Aspek Tradisional dan Non-tradisional dari Helm Berbasis Perilaku IvP   
IvP Helm memang mengambil motivasinya dari gagasan awal arsitektur berbasis perilaku, tetapi juga sangat berbeda dalam banyak hal. Gagasan tentang independensi perilaku untuk meredam pertumbuhan kompleksitas dalam sistem yang semakin besar masih merupakan prinsip yang diikuti dengan saksama dalam IvP Helm. Perilaku tentu saja dapat memengaruhi satu sama lain dari satu iterasi ke iterasi berikutnya, seperti yang akan kita lihat dalam diskusi di bagian ini. Ini juga terbukti dalam misi contoh Alpha di Bagian ??? di mana penyelesaian perilaku Survei memicu perilaku Pengembalian. Namun dalam satu iterasi, keluaran yang dihasilkan oleh satu perilaku tidak terpengaruh sama sekali oleh apa yang dihasilkan oleh perilaku lain dalam iterasi yang sama. Satu-satunya "komunikasi" antar-perilaku yang terwujud dalam satu iterasi terjadi ketika pemecah IvP merekonsiliasi keluaran dari beberapa perilaku. Independensi perilaku tidak hanya membantu satu pengembang mengelola pertumbuhan kompleksitas, tetapi juga membatasi ketergantungan antara pengembang. Seorang penulis perilaku tidak perlu khawatir bahwa perubahan dalam implementasi perilaku lain oleh penulis lain memerlukan pengodean ulang berikutnya dari perilakunya sendiri.

Aspek tertentu dari perilaku di IvP Helm juga mungkin menyimpang dari beberapa gagasan yang secara tradisional dikaitkan (cukup atau tidak) dengan arsitektur berbasis perilaku:

- Perilaku memiliki status. Perilaku IvP adalah contoh kelas dengan antarmuka yang cukup sederhana untuk mengendalikannya. Di dalamnya, perilaku tersebut mungkin sangat rumit, menyimpan riwayat data sensor yang diamati, dan mungkin berisi algoritme yang dapat dianggap "reaktif" atau "berbasis rencana".
- Perilaku saling memengaruhi di antara iterasi. Keluaran utama perilaku adalah fungsi objektifnya, yang memberi peringkat pada utilitas tindakan kandidat. Perilaku IvP juga dapat menghasilkan posting bernilai variabel ke MOOSDB yang dapat diamati oleh perilaku pada iterasi helm berikutnya. Dengan cara ini, perilaku tersebut dapat secara eksplisit memengaruhi perilaku lain dengan memicu atau menekan aktivasinya atau bahkan memengaruhi konfigurasi parameter perilaku lain.
- Perilaku dapat menerima rencana yang dihasilkan secara eksternal. Input ke suatu perilaku dapat berupa apa pun yang direpresentasikan oleh variabel MOOS, dan mungkin dihasilkan oleh proses MOOS lain di luar kemudi. Diperbolehkan untuk memiliki satu atau lebih mesin perencanaan yang berjalan pada kendaraan yang menghasilkan output yang dikonsumsi oleh satu atau lebih perilaku.
- Beberapa contoh perilaku yang sama. Perilaku umumnya menerima serangkaian parameter konfigurasi yang memungkinkannya dikonfigurasi untuk tugas atau peran yang sangat berbeda dalam kemudi dan misi yang sama. Perilaku titik arah yang berbeda, misalnya, dapat dikonfigurasi untuk komponen yang berbeda dari misi transit. Atau perilaku penghindaran tabrakan yang berbeda dapat diwujudkan untuk kontak yang berbeda.
- Perilaku dapat dijalankan dalam urutan yang dapat dikonfigurasi. Karena parameter kondisi dan endflag yang ditetapkan untuk semua perilaku, urutan perilaku dapat dengan mudah dikonfigurasi menjadi rencana misi yang lebih besar.
- Perilaku menilai tindakan melalui ruang keputusan yang digabungkan. Fungsi IvP yang dihasilkan oleh perilaku didefinisikan melalui produk Cartesian dari serangkaian variabel keputusan kendaraan. Hal ini berbeda dari gaya pengambilan keputusan yang dipisahkan yang diusulkan dalam \cite{?} dan \cite{?} - pendukung awal optimasi multi-objektif dalam pemilihan tindakan berbasis perilaku.


#### 1.1.3   Dua Lapisan Otonomi Bangunan di Helm IvP   
Otonomi yang berlaku pada kendaraan selama misi tertentu merupakan hasil dari dua upaya berbeda - (1) pengembangan perilaku kendaraan dan algoritmanya, dan (2) perencanaan misi melalui konfigurasi perilaku dan deklarasi moda. Yang pertama melibatkan penulisan kode sumber baru, dan yang kedua melibatkan penyuntingan berkas perilaku misi, seperti contoh sederhana untuk misi contoh Alpha dalam Listing ??? .


### 1.2   Di Dalam Helm - Melihat Loop Iterasi Helm​   
Seperti aplikasi MOOS lainnya, IvP Helm mengimplementasikan loop Iterate() yang di dalamnya fungsi dasar helm dijalankan. Komponen loop Iterate() , berkenaan dengan arsitektur berbasis perilaku, dijelaskan dalam bagian ini. Alur dasar, dalam lima langkah, digambarkan dalam . Deskripsi kelima komponen berikut.

> **Gambar 1.1: Loop Iterasi pHelmIvP**
> (1) Email dibaca dari MOOSDB. Email diurai dan disimpan dalam buffer lokal agar dapat diakses oleh perilaku, (2) Jika ada deklarasi mode dalam berkas perilaku misi, deklarasi tersebut dievaluasi pada langkah ini. (3) Setiap perilaku dikueri kontribusinya dan dapat menghasilkan fungsi IvP dan daftar pasangan variabel-nilai yang akan diposting ke MOOSDB di akhir iterasi, (4) fungsi objektif diselesaikan untuk menghasilkan tindakan, yang dapat dinyatakan sebagai sekumpulan pasangan variabel-nilai, (5) semua pasangan variabel-nilai dipublikasikan ke MOOSDB untuk dikonsumsi oleh proses MOOS lainnya.


#### 1.2.1   Langkah 1 - Membaca Email dan Mengisi Info Buffer   
Langkah pertama dari iterasi helm terjadi di luar loop Iterate() . Seperti yang digambarkan dalam Gambar ??? , aplikasi MOOS akan membaca emailnya dengan menjalankan fungsi OnNewMail() sebelum menjalankan loop Iterate() jika ada email di kotak masuknya. Helm mengurai email untuk mempertahankan buffer informasinya sendiri yang juga merupakan pemetaan variabel ke nilai. Ini dilakukan terutama untuk kesederhanaan - untuk memastikan bahwa setiap perilaku bertindak pada status dunia yang sama seperti yang diwakili oleh buffer info. Setiap perilaku memiliki pointer ke buffer dan dapat menanyakan nilai saat ini dari variabel apa pun di buffer, atau mendapatkan daftar perubahan nilai variabel sejak iterasi sebelumnya.


#### 1.2.2   Langkah 2 - Evaluasi Deklarasi Mode   
Setelah buffer informasi diperbarui dengan semua surat masuk, helm mengevaluasi setiap deklarasi mode yang ditetapkan dalam berkas perilaku. Deklarasi mode dibahas dalam . Singkatnya, mode direpresentasikan oleh variabel string yang disetel ulang pada setiap iterasi berdasarkan evaluasi serangkaian ekspresi logika yang melibatkan variabel lain dalam buffer. Variabel yang merepresentasikan deklarasi mode kemudian tersedia untuk perilaku pada iterasi saat ini ketika, misalnya, mengevaluasi parameter kondisinya . Oleh karena itu, kondisi untuk perilaku yang berpartisipasi dalam iterasi saat ini dapat dibaca seperti kondisi = (MODE==SURVEYING) . Nilai pasti dari variabel MODE ditetapkan selama langkah loop Iterate() ini .


#### 1.2.3   Langkah 3 - Partisipasi Perilaku   
Pada langkah ketiga, sebagian besar pekerjaan helm diwujudkan dengan memberikan setiap perilaku kesempatan untuk berpartisipasi. Setiap perilaku dikueri secara berurutan - helm tidak berisi utas terpisah dalam hal ini. Urutan perilaku yang dikueri tidak memengaruhi output. Langkah ini berisi dua bagian berbeda untuk setiap perilaku - (1) Penentuan apakah perilaku akan berpartisipasi, dan (2) produksi output jika memang berpartisipasi pada iterasi ini. Setiap perilaku dapat menghasilkan dua jenis informasi seperti yang ditunjukkan. Yang pertama adalah fungsi objektif (atau fungsi "utilitas") dalam bentuk fungsi IvP. Jenis output perilaku kedua adalah daftar pasangan variabel-nilai yang akan diposting oleh helm ke MOOSDB di akhir loop Iterate() . Suatu perilaku dapat menghasilkan kedua jenis informasi, tidak satu pun, atau salah satu atau yang lain, pada setiap iterasi yang diberikan.


#### 1.2.4   Langkah 4 - Rekonsiliasi Perilaku   
Pada langkah keempat yang digambarkan dalam , fungsi-fungsi IvP dikumpulkan oleh penyelesai IvP untuk menghasilkan satu keputusan atas ruang keputusan kemudi. Setiap fungsi adalah fungsi IvP - fungsi objektif yang memetakan setiap elemen ruang keputusan kemudi ke nilai utilitas. Dalam kasus ini, fungsi-fungsi tersebut memiliki bentuk tertentu - didefinisikan secara linier sepotong-sepotong. Artinya, setiap bagian adalah interval ruang keputusan dengan fungsi linier terkait. Setiap fungsi juga memiliki bobot terkait dan penyelesai melakukan optimasi multi-objektif atas jumlah fungsi yang terbobot (yang pada dasarnya merupakan optimasi objektif tunggal pada titik tersebut). Outputnya adalah satu titik optimal tunggal dalam ruang keputusan. Untuk setiap variabel keputusan, kemudi menghasilkan pasangan variabel-nilai lain, seperti DESIRED_SPEED = 2,4 untuk dipublikasikan ke MOOSDB.


#### 1.2.5   Langkah 5 - Menerbitkan Hasil ke MOOSDB   
Pada langkah terakhir, helm hanya menerbitkan semua pasangan variabel-nilai ke MOOSDB, beberapa di antaranya diproduksi secara langsung oleh perilaku, dan beberapa di antaranya dihasilkan sebagai output dari IvP Solver. Helm menggunakan filter duplikasi yang dijelaskan di Bagian ??? , hanya pada pasangan variabel-nilai yang dihasilkan secara langsung dari perilaku, dan bukan pasangan variabel-nilai yang dihasilkan oleh IvP solver yang mewakili keputusan dalam domain helm. Misalnya, bahkan jika keputusan tentang kedalaman kendaraan, yang diwakili oleh variabel DESIRED_DEPTH yang dihasilkan oleh helm tidak berubah selama 5 menit pengoperasian, keputusan tersebut akan dipublikasikan pada setiap iterasi helm. Melakukan hal sebaliknya dapat memberikan kesan kepada konsumen variabel bahwa variabel tersebut "basi", yang dapat memicu penggantian helm yang tidak diinginkan karena khawatir akan keselamatan.


### 1.3   File Perilaku Misi    
Helm dikonfigurasikan untuk misi tertentu terutama melalui satu atau beberapa file perilaku misi, biasanya dengan sufiks *.bhv . Berkas perilaku memiliki tiga jenis entri, biasanya tetapi tidak harus disimpan dalam tiga bagian yang berbeda - (1) inisialisasi variabel, (2) konfigurasi perilaku, dan (3) deklarasi mode hierarkis. Ketiga bagian ini dibahas di bawah ini. Contoh berkas alpha.bhv dalam Listing ??? tidak berisi deklarasi mode hierarkis, tetapi berisi contoh inisialisasi variabel dan konfigurasi perilaku.


#### 1.3.1   Sintaks Inisialisasi Variabel   
Sintaks untuk inisialisasi variabel cukup mudah dipahami:

```
    initialize  <variable> = <value>
    ...
    initialize  <variable> = <value>
```

Beberapa inisialisasi dapat dideklarasikan pada satu baris dengan memisahkan setiap pasangan variabel-nilai dengan koma. Kata kunci initialize tidak peka huruf besar/kecil. <variabel> memang peka huruf besar/kecil karena akan dipublikasikan ke MOOSDB dan variabel MOOS peka huruf besar/kecil saat didaftarkan oleh klien. <nilai> mungkin peka huruf besar/kecil atau tidak, tergantung pada apakah klien yang mendaftar untuk variabel tersebut memperhatikan huruf besar/kecil atau tidak. Jika kita perhatikan lagi loop helm Iterate() yang digambarkan dalam , inisialisasi variabel diterapkan ke buffer informasi helm sebelum iterasi helm pertama, tetapi diposting ke MOOSDB di akhir iterasi helm pertama.


Secara default, inisialisasi akan menimpa nilai sebelumnya yang diposting ke MOOSDB. Namun, mungkin ada situasi di mana efek yang diinginkan pengguna adalah inisialisasi hanya diterapkan jika belum ada nilai lain yang ditulis ke variabel MOOS yang diberikan. Sintaks dalam kasus ini adalah:

```
    initialize_  <variable> = <value>    // Deferring to prior posts if any
```

Dengan menggunakan versi "garis bawah" dari deklarasi inisialisasi , helm akan terlebih dahulu mendaftar dengan MOOSDB untuk variabel yang diberikan, menunggu satu iterasi hingga ia berkesempatan menerima email dari MOOSDB pada variabel tersebut, dan hanya menginisialisasi variabel tersebut jika tidak ada yang diketahui tentang variabel tersebut. (Catatan untuk pembaca yang sangat jeli: Inisialisasi tersebut juga mencakup pembaruan pada buffer informasi helm dan posting ke MOOSDB. Posting ke MOOSDB oleh helm, sebagai bagian dari inisialisasi variabel, memang akan muncul di kotak surat masuk helm pada iterasi berikutnya, tetapi posting tersebut ditandai sedemikian rupa sehingga diabaikan oleh helm. Hal ini untuk memastikan bahwa posting tersebut tidak "bertabrakan" dengan posting yang dibuat oleh proses lain.)


#### 1.3.2   Sintaks Konfigurasi Behavior   
Sebagian besar konfigurasi helm dilakukan dengan blok parameter perilaku individual yang memiliki bentuk sebagai berikut:

```
  Behavior = <behavior-type>
  {
    <parameter> = <value>
    ...
    <parameter> = <value>
  }
```

Baris pertama adalah deklarasi tipe perilaku. Kata kunci Behavior tidak peka huruf besar/kecil, tetapi <behavior-type> peka huruf besar/kecil. Ini diikuti oleh kurung buka pada baris terpisah. Setiap baris berikutnya menetapkan parameter perilaku tertentu ke nilai tertentu. Konfigurasi perilaku diakhiri dengan kurung tutup pada baris terpisah. Masalah kepekaan huruf besar/kecil untuk entri <parameter> dan <value> adalah masalah yang ditentukan oleh implementasi perilaku masing-masing.

Sebagai sebuah konvensi (tidak ditegakkan dengan cara apa pun) parameter perilaku umum, yang didefinisikan pada level superkelas Perilaku IvP, dikelompokkan bersama dan dicantumkan sebelum parameter yang berlaku untuk perilaku tertentu. Misalnya, dalam contoh Alfa dalam Daftar ??? , parameter perilaku umum dicantumkan pada baris 8-12 dan 22-25, tetapi parameter khusus untuk perilaku titik jalan, speed , radius , dan points , mengikuti dalam blok terpisah. Umumnya tidak wajib untuk menyediakan pasangan parameter-nilai untuk setiap parameter yang didefinisikan untuk suatu perilaku, mengingat bahwa default yang bermakna ada dalam implementasi perilaku. Namun, beberapa parameter memang wajib. Dokumentasi untuk perilaku individual harus dikonsultasikan. Beberapa contoh jenis perilaku diperbolehkan, seperti dalam contoh Alfa di mana terdapat dua perilaku titik jalan - satu untuk melintasi serangkaian titik, dan satu untuk kembali ke titik pemulihan kendaraan. Setiap perilaku harus memiliki nilai uniknya sendiri yang disediakan dalam parameter name .


#### 1.3.3   Sintaks Deklarasi Mode Hirarkis   
Deklarasi Mode Hirarkis dibahas secara mendalam di , tetapi sintaksnya dibahas secara singkat di sini. File perilaku berisi sekumpulan blok deklarasi dalam bentuk:

```
  Set <mode-variable-name> = <mode-value>
  {
    <mode-variable-name> = <parent-value>
    <condition>
    . . .
    <condition>
  } <else-value>
```

Pohon akan terbentuk di mana setiap simpul di pohon dijelaskan dari jenis deklarasi di atas. Kata kunci Set tidak peka huruf besar/kecil. <nama-variabel-mode> , <nilai-induk> dan <nilai-lain> peka huruf besar/kecil. Entri <kondisi> diperlakukan sama persis dengan parameter kondisi untuk perilaku, lihat Bagian ??? .

Seperti yang ditunjukkan dalam , nilai setiap variabel mode disetel ulang di awal loop Iterate() , setelah buffer informasi diperbarui dengan email masuk. Variabel mode ditetapkan dengan menelusuri setiap blok deklarasi, dan menentukan apakah kondisinya terpenuhi. Jadi, urutan blok deklarasi penting - spesifikasi induk harus dibuat sebelum spesifikasi anak. Contoh pembahasan lebih lanjut dapat ditemukan di bawah ini dalam .


### 1.4   Deklarasi Mode Hirarkis  
Deklarasi mode hierarkis (HMD) merupakan fitur opsional Helm IvP untuk mengatur aktivasi perilaku sesuai dengan mode misi yang dideklarasikan. Mode dan sub-mode dapat dideklarasikan, sejalan dengan konsep evolusi misi perencana misi, dan perilaku dapat dikaitkan dengan mode yang dideklarasikan. Dalam misi yang lebih kompleks, fitur ini dapat memfasilitasi perencanaan misi (dalam hal waktu yang lebih sedikit dan deteksi kesalahan manusia yang lebih baik), dan dapat memfasilitasi pemahaman tentang apa yang sebenarnya terjadi di helm - selama pelaksanaan misi dan pasca-analisis.


#### 1.4.1   Latar Belakang
Tren penggunaan kendaraan tak berawak dapat dicirikan sebagai semakin berkurangnya jenis misi yang lebih pendek dan terprogram menjadi semakin banyak jenis misi yang lebih panjang dan adaptif. Misi umum di lab kami sendiri lima tahun lalu akan berisi serangkaian tugas tertentu, biasanya titik jalan dan akhirnya titik pertemuan untuk memulihkan kendaraan. Data yang diperoleh selama penyebaran diturunkan dan dianalisis kemudian di laboratorium. Apa yang berubah? Pematangan komunikasi akustik, pemrosesan sensor di dalam pesawat, dan masa pakai baterai kendaraan yang lebih lama secara bersamaan telah mengubah sifat konfigurasi misi secara dramatis. Kendaraan diharapkan beradaptasi dengan fenomena yang diindera dan diproses di dalam pesawat, serta menyesuaikan operasinya dengan perintah kontrol lapangan yang diterima melalui komunikasi akustik, radio, atau satelit. Misi kolaboratif multi-kendaraan juga semakin layak karena biaya kendaraan yang lebih rendah dan kemampuan komunikasi yang matang. Dalam kasus seperti itu, kendaraan tidak hanya beradaptasi dengan fenomena yang diindera dan perintah lapangan, tetapi juga dengan informasi dari kendaraan yang berkolaborasi.

Misi kami telah berevolusi dari memiliki serangkaian tugas tetap yang terbatas yang harus disusun alih-alih serangkaian moda, moda awal saat diluncurkan, pemahaman tentang apa yang membawa kita dari satu moda ke moda lain, dan perilaku apa yang berlaku di setiap moda. Moda dapat dimasuki dan keluar beberapa kali, dalam urutan yang tepat yang tidak diketahui pada waktu peluncuran, tergantung pada apa yang mereka rasakan dan bagaimana mereka diperintahkan di lapangan.


#### 1.4.2   Konfigurasi Perilaku Tanpa Deklarasi Mode Hirarkis   
Perilaku dapat dikonfigurasi untuk misi tanpa menggunakan deklarasi mode hierarkis - dukungan untuk HMD merupakan tambahan yang relatif baru pada kemudi. HMD adalah alat untuk mengatur perilaku mana yang diam atau berpartisipasi dalam keadaan mana. Pertimbangkan contoh misi alfa di Bagian ??? , dan berkas perilaku di Daftar ??? . Dengan memeriksa berkas perilaku, dan sedikit bereksperimen dengan penampil selama simulasi, kendaraan tampaknya selalu dalam satu dari tiga mode - (a) diam, (b) mengamati titik jalan, atau (c) kembali ke titik peluncuran. Ini dicapai dengan parameter kondisi untuk dua perilaku. Hanya ada dua variabel yang terlibat dalam kondisi perilaku, DEPLOY dan RETURN . Jika dibatasi pada nilai Boolean, tabel di bawah mengonfirmasi pengamatan bahwa hanya ada tiga mode yang mungkin.

| DEPLOY |	RETURN |	Mode |
| -------| --------| ------|
| true |	true |	Returning |
| true |	false |	Surveying |
| false |	true |	Idle |
| false |	false |	Idle |
