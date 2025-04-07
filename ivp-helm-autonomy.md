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

Table 1.1: Kemungkinan mode yang tersirat oleh parameter kondisi pada misi alfa dalam Daftar ??? .

Namun, ada beberapa kekurangan dengan ini. Pertama, mode harus disimpulkan dari kondisi perilaku dan ini tidak mudah dalam misi dengan berkas perilaku yang lebih besar. Memetakan kondisi perilaku ke suatu mode berguna baik dalam perencanaan misi maupun pemantauan misi. Dalam misi alfa, untuk memahami mode kendaraan pada saat tertentu, kedua variabel perlu dipantau, dan tabel di atas diinternalisasi. Kekurangan kedua adalah meningkatnya kemungkinan kesalahan, dalam bentuk tidak sengaja berada dalam dua mode pada saat yang sama, atau berada dalam mode yang tidak ditentukan. Misalnya, baris 11 dalam Listing ??? seharusnya berbunyi RETURN != true , dan bukan RETURN = false . Karena tidak ada tipe Boolean untuk variabel MOOS, variabel ini dapat disetel ke "False" dan kondisi seperti yang terbaca pada baris 11 dalam Listing ??? tidak akan terpenuhi, dan kendaraan akan berada dalam status diam, meskipun faktanya DEPLOY dapat disetel ke true . Masalah ini diatasi dengan penggunaan deklarasi mode hierarkis.


#### 1.4.3   Sintaksis Deklarasi Mode Hirarkis - Misi Bravo   
Contoh diberikan untuk menunjukkan penggunaan deklarasi mode hierarkis dengan memperluas misi Alpha yang dijelaskan di Bagian ??? . Contoh misi ini dijuluki misi "Bravo" di direktori s2_bravo di samping misi Alpha s1_alpha dalam distribusi MOOS-IvP ( Bagian ??? ). Misi ini juga diberikan secara lengkap di halaman berikutnya. Mode implisit misi Alpha, yang dijelaskan di , dideklarasikan secara eksplisit dalam berkas perilaku Bravo untuk membentuk hierarki berikut:


> **Gambar 1.2: Mode hierarkis untuk misi Bravo **\
> Kendaraan akan selalu berada dalam salah satu mode yang direpresentasikan oleh simpul daun. Suatu perilaku dapat dikaitkan dengan simpul mana pun di pohon. Jika suatu perilaku dikaitkan dengan simpul internal, maka perilaku tersebut juga dikaitkan dengan semua anaknya.

Hirarki dalam dibentuk oleh konstruksi deklarasi mode di sisi kiri, diambil sebagai kutipan dari berkas bravo.bhv . Setelah deklarasi mode dibaca saat helm awalnya diluncurkan, hierarki tetap statis setelahnya. Hirarki dikaitkan dengan variabel MOOS tertentu, dalam hal ini variabel MODE . Meskipun hierarki tetap statis, mode dievaluasi ulang pada awal setiap iterasi helm berdasarkan kondisi yang dikaitkan dengan node dalam hierarki. Evaluasi mode direpresentasikan sebagai string dalam variabel MODE . Seperti yang ditunjukkan dalam variabel tersebut adalah gabungan nama-nama semua node. Evaluasi mode dimulai secara berurutan melalui setiap blok. Pada awalnya nilai variabel MODE diatur ulang ke string kosong. Setelah blok pertama dalam MODE akan ditetapkan ke "Aktif" atau "Tidak Aktif" . Ketika blok kedua dievaluasi, kondisi "MODE=Aktif" dievaluasi berdasarkan bagaimana MODE ditetapkan di blok pertama. Karena alasan ini, deklarasi mode anak perlu dicantumkan setelah deklarasi induk dalam berkas perilaku.

Setelah mode dievaluasi, pada permulaan iterasi helm, mode tersebut tersedia untuk digunakan dalam kondisi perilaku, seperti pada baris 20 dan 23 di . Perhatikan relasi "==" pada baris 18 dan 36. Ini adalah relasi pencocokan string yang cocok ketika satu sisi cocok dengan tepat satu dari komponen dalam daftar string yang dipisahkan titik dua di sisi lainnya. Jadi "Aktif" == "Aktif:Mengembalikan" , dan "Mengembalikan" == "Aktif:Mengembalikan" . Ini untuk memungkinkan perilaku dikaitkan dengan mudah dengan simpul internal terlepas dari anak-anaknya. Misalnya jika perilaku penghindaran tabrakan akan ditambahkan ke misi ini, perilaku itu dapat dikaitkan dengan mode "Aktif" daripada secara eksplisit menamai semua sub-mode dari mode "Aktif" .


**Daftar 1.1** - Misi Bravo - Penggunaan Deklarasi Mode Hirarkis.

```
   1  initialize   DEPLOY = false
   2  initialize   RETURN = false
   3     
   4  //------------------- Declaration of Hierarchical Modes
   5  set MODE = ACTIVE {
   6    DEPLOY = true
   7  } INACTIVE
   8   
   9  set MODE = SURVEYING {
  10    MODE = ACTIVE
  11    RETURN != true
  12  } RETURNING
  13   
  14  //----------------------------------------------
  15  Behavior = BHV_Waypoint
  16  { 
  17    name      = waypt_survey
  18    pwt       = 100
  19    condition = MODE == SURVEYING
  20    endflag   = RETURN = true
  21    perpetual = true
  22  
  23            lead = 8
  24     lead_damper = 1
  25           speed = 2.0   // meters per second
  26          radius = 4.0
  27       nm_radius = 10.0
  28          points = 60,-40:60,-160:150,-160:180,-100:150,-40
  29          repeat = 1
  30  }
  31    
  32  //----------------------------------------------
  33  Behavior = BHV_Waypoint
  34  {
  35    name       = waypt_return
  36    pwt        = 100
  37    condition  = MODE == RETURNING
  38    perpetual  = true
  39    endflag    = RETURN = false
  40    endflag    = DEPLOY = false
  41  
  42         speed = 2.0
  43        radius = 2.0
  44     nm_radius = 8.0
  45         point = 0,0
  46  }

```


#### 1.4.4   Contoh Deklarasi Mode Hirarkis yang Lebih Kompleks   
Contoh Bravo yang diberikan di atas, meskipun memiliki manfaat sebagai contoh kerja yang didistribusikan dengan basis kode, tidaklah rumit. Di bagian ini hierarki yang cukup rumit, meskipun fiktif, disediakan untuk menyoroti beberapa masalah dengan sintaksis. Hierarki dengan deklarasi mode yang sesuai ditunjukkan dalam . Deklarasi diberikan dalam urutan lapisan pohon yang memastikan bahwa induk dideklarasikan sebelum anak. Seperti contoh Bravo dalam , simpul yang mewakili mode yang dapat direalisasikan digambarkan dalam warna yang lebih gelap (hijau).

> **Gambar 1.3: Contoh Deklarasi Mode Hirarkis**
> Hirarki di sebelah kanan dibangun dari serangkaian deklarasi mode di sebelah kiri (dengan kondisi fiktif). Node yang lebih gelap mewakili mode yang dapat diwujudkan melalui beberapa kombinasi kondisi.

Mode "Alpha" misalnya tidak dapat direalisasikan karena memiliki anak "Delta" dan "Echo" , dengan yang terakhir ditetapkan sebagai <nilai-lain> jika kondisi yang pertama tidak terpenuhi. Mode "Bravo" dapat direalisasikan karena tidak memiliki anak. Mode "Echo" dapat direalisasikan meskipun memiliki anak karena mode "Tango" bukan <nilai-lain> dari deklarasi mode "Sierra" . Misalnya, jika tiga kondisi berikut berlaku, (a) "MISSION=SURVEYING" , (b) "SITE!=Archipelagos" , dan (c) "WATER_DEPTH=Medium" , maka nilai variabel MODE akan ditetapkan ke "Alpha:Echo" . Akhirnya, perhatikan bahwa kondisi dalam deklarasi "Sierra" , "MODE=Alpha:Echo" , ditetapkan sepenuhnya, yaitu, "MODE=Echo" tidak akan mencapai hasil yang diinginkan.


#### 1.4.5   Memantau Mode Misi pada Waktu Proses   
Mode misi dapat dipantau saat dijalankan dengan beberapa cara. Pertama, karena variabel mode diposting sebagai variabel MOOS, alat cakupan MOOS apa pun akan berfungsi, misalnya, uXMS , uMS , uHelmScope . Dengan menggunakan uHelmScope , variabel misi dapat dipantau sebagai bagian dari kemampuan cakupan MOOSDB dasar, tetapi juga ditampilkan sebagai bagian dari aplikasi uHelmScope , dan dalam keluaran AppCasting dari pHelmIvP .

Alat uHelmScope juga memiliki mode di mana seluruh hierarki mode dapat ditampilkan - hanya untuk memberikan konfirmasi visual bahwa hierarki yang ditetapkan dengan deklarasi mode dalam berkas perilaku memang sesuai dengan apa yang diinginkan pengguna. Saat ini tidak ada alat untuk secara otomatis menampilkan hierarki mode dengan cara seperti sisi kanan . Output uHelmScope untuk contoh dalam ditampilkan dalam daftar di bawah ini.


**Daftar 1.2** - Hirarki mode keluaran uHelmScope untuk contoh di .

```
   1   ModeSet Hierarchy: 
   2   ---------------------------------------------- 
   3   Alpha
   4       Delta
   5       Echo
   6           Sierra
   7           Tango
   8   Bravo
   9   Charlie
  10       Foxtrot
  11       Golf
  12   ---------------------------------------------- 
  13   CURENT MODE(S): Charlie:Foxtrot
  14                                                               
  15   Hit 'r' to resume outputs, or SPACEBAR for a single update  
```

Informasi lebih lanjut tentang fitur uHelmScope ini dapat ditemukan dalam dokumentasi uHelmScope , \cite{?} . Perlu dicatat bahwa memasukkan nilai variabel mode tidak akan berpengaruh pada operasi helm. Mode misi tidak dapat diperintah secara langsung. Variabel mode disetel ulang pada awal iterasi helm, dan helm bahkan tidak mendaftar untuk menerima email pada variabel mode.


### 1.5   Perilaku Partisipasi dalam IvP Helm   
Pekerjaan utama helm muncul saat perilaku berpartisipasi dan melakukan tugasnya, di setiap putaran loop Iterate() helm . Seperti yang digambarkan dalam , setelah mode dievaluasi ulang dengan mempertimbangkan email yang baru diterima, saatnya bagi perilaku (yah, setidaknya beberapa) untuk melangkah maju dan melakukan tugasnya.


#### 1.5.1   Kondisi Perilaku Berjalan   
Pada setiap iterasi tunggal, suatu perilaku dapat berpartisipasi dengan menghasilkan fungsi objektif untuk memengaruhi keluaran kemudi atas ruang keputusannya. Tidak semua perilaku berpartisipasi dalam hal ini, dan kriteria utama untuk partisipasi adalah apakah perilaku tersebut telah memenuhi setiap "kondisi jalan" atau tidak. Berikut adalah kondisi yang ditetapkan dalam berkas perilaku dalam bentuk:

```
    condition = <logic-expression>
```

Sintaks <logic-expression> dijelaskan dalam Lampiran ??? . Kondisi dibangun dari ekspresi relasional sederhana, perbandingan variabel MOOS dengan nilai literal yang ditentukan, atau perbandingan variabel MOOS satu sama lain. Kondisi juga dapat melibatkan kombinasi logika Boolean dari ekspresi relasi. Suatu perilaku dapat mendasarkan kondisinya pada variabel MOOS apa pun seperti:

```
    condition = (DEPLOY=true) and (STATION_KEEP != true)
```

Kondisi lari juga dapat dinyatakan dalam bentuk mode kemudi, seperti yang dijelaskan berikut ini:

```
    condition = (MODE == LOITERING)
```

Semua variabel MOOS yang terlibat dalam ekspresi kondisi proses secara otomatis dilanggan oleh helm ke MOOSDB.


#### 1.5.2   Kondisi Perilaku dan Deklarasi Mode   
Penggunaan deklarasi mode hierarkis berpotensi menyederhanakan ekspresi yang digunakan sebagai kondisi berjalan. Kondisi dalam praktiknya dapat dibatasi pada:

```
    condition = <mode-variable> = <mode-value>, or
    condition = <mode-variable> == <mode-value>.
```

Kondisi digunakan dengan cara ini dengan misi Bravo di , sebagai alternatif penggunaannya dalam contoh misi Alpha dalam Daftar ??? .

Perhatikan penggunaan relasi sama dengan dua di atas. Relasi ini digunakan untuk mencocokkan dengan string yang digunakan untuk merepresentasikan mode hierarki. Kedua string tersebut cocok jika komponen yang diurutkan dari satu sisi merupakan bagian dari komponen yang diurutkan dari sisi lainnya. Komponen dipisahkan dengan titik dua. Misalnya, menggunakan hierarki ilustratif dari:

```
        "Alpha:Echo:Sierra" == "Sierra"
        "Alpha:Echo:Sierra" == "Echo:Sierra"
        "Alpha:Echo:Sierra" == "Alpha"
                   "Sierra" == "Alpha:Echo:Sierra"
          "Charlie:Foxtrot" == "Charlie:Foxtrot"

        "Alpha:Echo:Sierra" != "Alpha:Sierra"
```


#### 1.5.3   Perilaku Menjalankan Status 
Pada setiap iterasi helm, suatu perilaku mungkin berada dalam salah satu dari empat keadaan yang digambarkan dalam:

> **Gambar 1.4: Status Perilaku**\
> Suatu perilaku dapat berada dalam salah satu dari empat status ini pada setiap iterasi loop Iterate() helm . Status ditentukan dengan memeriksa variabel MOOS yang disimpan secara lokal dalam buffer informasi helm.

- Idle: Suatu perilaku dikatakan idle jika belum selesai dan belum memenuhi kondisi jalannya sebagaimana dijelaskan di atas pada Bagian ??? . Helm akan memanggil fungsi onIdleState() dari perilaku idle .
- Running: Suatu perilaku berjalan jika telah memenuhi kondisi jalannya dan belum selesai . Helm akan memanggil fungsi onRunState() dari perilaku yang sedang berjalan sehingga memberikan kesempatan pada perilaku tersebut untuk memberikan kontribusi pada fungsi objektif.
- Active: Suatu perilaku aktif jika sedang berjalan dan memang menghasilkan fungsi objektif saat diminta. Ada sejumlah alasan mengapa suatu perilaku yang sedang berjalan mungkin tidak aktif. Misalnya, perilaku penghindaran tabrakan di mana objek perilaku tersebut cukup jauh.
- Complete: Suatu perilaku lengkap ketika perilaku itu sendiri menentukannya sebagai lengkap. Terserah kepada pembuat perilaku untuk mengimplementasikannya, dan beberapa perilaku mungkin tidak akan pernah selesai. Fungsi setComplete() didefinisikan secara umum pada tingkat superkelas perilaku, untuk dipanggil oleh pembuat perilaku. Ini menyediakan beberapa langkah standar yang harus diambil setelah selesai, seperti memasang endflag , yang dijelaskan di bawah ini. Setelah suatu perilaku berada dalam status lengkap , perilaku tersebut akan tetap berada dalam status tersebut secara permanen. Semua perilaku memiliki parameter durasi yang didefinisikan untuk memungkinkannya dikonfigurasi untuk mencapai batas waktu jika diinginkan. Ketika batas waktu terjadi, status perilaku akan ditetapkan menjadi lengkap .


#### 1.5.4   Bendera Perilaku dan Pesan Perilaku   
Perilaku dapat mengirim sejumlah pesan, yaitu, pasangan variabel-nilai, pada setiap iterasi yang diberikan (lihat ). Pesan-pesan ini dapat menjadi penting untuk mengoordinasikan perilaku satu sama lain dan dengan proses MOOS lainnya. Ini juga dapat sangat berharga untuk memantau dan men-debug perilaku yang dikonfigurasi untuk misi tertentu. Agar lebih akurat, perilaku tidak mengirim pesan ke MOOSDB, mereka meminta helm untuk mengirim pesan atas namanya. Helm mengumpulkan permintaan ini dan menerbitkannya ke MOOSDB di akhir loop Iterate() . Ia juga memfilternya untuk duplikat berturut-turut seperti yang dibahas di Bagian ??? .

Ada metode standar, yang dapat dikonfigurasi dalam berkas perilaku, untuk mengeposkan pesan berdasarkan status jalannya perilaku. Ini disebut sebagai tanda perilaku, dan ada beberapa jenis, endflag , idleflag , runflag , activeflag , inactiveflag , dan spawnflag . Pasangan variabel-nilai yang mewakili setiap tanda ditetapkan dalam berkas perilaku untuk perilaku yang sesuai. Lihat baris 11 dalam Daftar ??? misalnya.

- endflag : Endflag diposkan sekali saat perilaku memasuki status lengkap . Pasangan variabel-nilai yang mewakili endflag diberikan dalam parameter endflag dalam berkas perilaku. Beberapa endflag dapat dikonfigurasi untuk suatu perilaku.
- idleflag : Idleflag dipasang oleh helm saat perilaku memasuki status idle . Pasangan variabel-nilai yang mewakili idleflag diberikan dalam parameter idleflag di berkas perilaku. Beberapa idleflag dapat dikonfigurasi untuk suatu perilaku.
- runflag : Runflag diposting oleh helm saat perilaku memasuki status berjalan dari status diam . Runflag diposting tepat saat idleflag tidak. Pasangan variabel-nilai yang mewakili runflag diberikan dalam parameter runflag di berkas perilaku. Beberapa runflag dapat dikonfigurasi untuk suatu perilaku.
- activeflag : Activeflag dipasang oleh helm saat perilaku memasuki status aktif . Pasangan variabel-nilai yang mewakili activeflag diberikan dalam parameter activeflag di berkas perilaku. Beberapa activeflag dapat dikonfigurasi untuk suatu perilaku.
- inactiveflag : Inactiveflag dipasang oleh helm saat perilaku memasuki status yang bukan status aktif . Pasangan variabel-nilai yang mewakili inactiveflag diberikan dalam parameter inactiveflag dalam berkas perilaku. Beberapa inactiveflag dapat dikonfigurasi untuk suatu perilaku.
- spawnflag : Sebuah spawnflag dipasang oleh helm saat perilaku templat muncul. Contohnya termasuk perilaku tabrakan dan penghindaran rintangan. Pasangan variabel-nilai yang mewakili spawnflag diberikan dalam parameter spawnflag dalam berkas perilaku. Beberapa spawnflag dapat dikonfigurasi untuk suatu perilaku.

Runflag dimaksudkan untuk "melengkapi" idleflag , dengan memposting tepat saat yang lain tidak melakukannya. Mirip dengan inactiveflag dan activeflag . Situasinya ditunjukkan dalam:

> **Gambar 1.5: Bendera Perilaku **\
> Empat bendera perilaku idleflag , runflag , activeflag , dan inactiveflag diposting tergantung pada status perilaku dan dapat dianggap pelengkap dengan cara yang ditunjukkan.

Penulis perilaku dapat menerapkan perilaku mereka untuk mengirim pesan lain sesuai keinginan mereka. Misalnya, perilaku titik jalan yang digunakan dalam contoh Alfa di Bagian ??? juga menerbitkan variabel WPT_STAT dengan pesan status yang mirip dengan "vname=alpha,index=0,dist=124,eta=62" yang menunjukkan nama kendaraan, indeks titik berikutnya dalam daftar titik jalan, jarak ke titik jalan tersebut, dan perkiraan waktu kedatangan, dalam hitungan detik. (Anda mungkin ingin menjalankan kembali misi Alfa dengan cakupan uXMS pada variabel ini untuk melihatnya berubah seiring berjalannya misi.)


#### 1.5.5   Pemantauan Perilaku Menjalankan Status dan Pesan Selama Eksekusi Misi   
Status operasi untuk setiap perilaku, dirangkum pada setiap iterasi oleh helm menjadi satu string dan dipublikasikan dalam variabel IVPHELM_SUMMARY . Variabel ini dilanggan oleh alat uHelmScope dan status perilaku diurai dari variabel ini dan diringkas dalam keluaran utama uHelmScope , seperti dalam kutipan di bawah ini:

```
  12  Behaviors Active: ---------- (1)
  13    waypt_survey (13.0) (pwt=100.00) (pcs=1227) (cpu=0.01) (upd=0/0)
  14  Behaviors Running: --------- (0)
  15  Behaviors Idle: ------------ (1)
  16    waypt_return (22.8)
  17  Behaviors Completed: ------- (0)
```

Perilaku dikelompokkan ke dalam empat kemungkinan status, dengan baris ringkasan untuk setiap status, misalnya, baris 12, 14, 15, 17, yang berisi jumlah perilaku dalam status tersebut dalam tanda kurung di akhir baris. Setiap perilaku yang dikonfigurasi untuk helm muncul pada baris khusus dalam grup yang sesuai, misalnya, baris 13 dan 16. Pada baris ini tepat setelah nama perilaku, jumlah detik ditampilkan dalam tanda kurung yang menunjukkan berapa lama perilaku tersebut berada dalam status tersebut.


### 1.6   Rekonsiliasi Perilaku di Helm IvP - Optimasi Multi-Objektif   


#### 1.6.1   Fungsi IvP   
Fungsi IvP dihasilkan oleh perilaku untuk memengaruhi keputusan yang dihasilkan oleh kemudi pada iterasi saat ini (lihat ). Keputusan biasanya terdiri dari arah yang diinginkan , kecepatan , dan kedalaman , tetapi ruang keputusan kemudi dapat terdiri dari konfigurasi sembarang (lihat Bagian ??? ). Beberapa poin tentang fungsi IvP:

- Fungsi IvP didefinisikan secara linier sepotong-sepotong. Setiap bagian didefinisikan oleh interval di atas beberapa subset ruang keputusan, dan ada fungsi linier yang dikaitkan dengan setiap bagian (lihat ).
- Fungsi IvP merupakan perkiraan dari fungsi yang mendasarinya. Fungsi linier untuk satu bagian adalah perkiraan linier terbaik dari fungsi yang mendasarinya untuk bagian domain yang dicakup oleh bagian tersebut.
- Domain IvP bersifat diskret dengan batas atas dan batas bawah untuk setiap variabel, sehingga fungsi IvP dapat mencapai kesalahan nol dalam memperkirakan fungsi yang mendasarinya dengan mengaitkan bagian dengan setiap titik dalam domain. Namun, perilaku jarang perlu melakukan hal tersebut dalam praktik.
- Konstruksi fungsi Ivp dan penyelesai IvP dapat digeneralisasikan ke N dimensi.
- Bagian-bagian dalam fungsi IvP tidak harus memiliki ukuran atau bentuk yang seragam. Lebih banyak bagian dapat didedikasikan untuk bagian-bagian domain yang lebih sulit untuk didekati dengan fungsi linear.
- Fungsi IvP hanya perlu didefinisikan pada sebagian kecil domain. Perilaku tidak terpengaruh jika kemudi dikonfigurasi untuk variabel tambahan yang mungkin tidak dipedulikan oleh perilaku. Perilaku yang menghasilkan fungsi hanya pada kedalaman kendaraan tidak masalah.

Bagaimana fungsi IvP dibangun? IvP Build Toolbox adalah seperangkat alat untuk membuat fungsi IvP berdasarkan fungsi dasar yang ditetapkan pada Domain IvP. Banyak, jika tidak semua perilaku dalam dokumen ini menggunakan kotak alat ini, dan penulis perilaku baru memilikinya. Komponen utama penulisan perilaku baru adalah pengembangan "fungsi dasar", fungsi yang didekati oleh fungsi IvP dengan bantuan kotak alat. Fungsi dasar mewakili hubungan antara keputusan pimpinan kandidat dan utilitas yang diharapkan sehubungan dengan tujuan perilaku. IvP Toolbox tidak dibahas secara rinci dalam dokumen ini, tetapi ikhtisarnya diberikan di bawah ini.


#### 1.6.2   Kotak Alat Bangun IvP   
Kotak Alat IvP adalah seperangkat alat (pustaka C++) untuk membangun fungsi IvP. Kotak alat ini biasanya digunakan oleh penulis perilaku dalam serangkaian panggilan pustaka dalam implementasi perilaku (C++). Ada dua perangkat alat - alat Reflector untuk membangun fungsi IvP dalam dimensi N, dan alat ZAIC untuk membangun fungsi IvP dalam satu dimensi sebagai kasus khusus. Alat Reflector bekerja dengan menyediakan fungsi yang akan didekati dengan fungsi IvP. Alat ini hanya memerlukan fungsi ini untuk pengambilan sampel. Perhatikan fungsi Gaussian yang ditampilkan di bawah ini dalam:


> **Gambar 1.6: Render fungsi di mana A = {\tt range} = 150, = {\tt sigma} = 32,4 , = {\tt xcent} = 50, = {\tt ycent} = -150 . Domain di sini untuk x dan y berkisar dari -250 hingga 250** .

Variabel 'x' dan 'y' , masing-masing dengan rentang [-250, 250], bersifat diskrit, mengambil nilai integer. Oleh karena itu domain berisi titik, atau keputusan yang mungkin. IvP Build Toolbox dapat menghasilkan fungsi IvP yang memperkirakan fungsi ini pada domain ini dengan menggunakan ukuran bagian yang seragam, seperti yang diberikan dalam . Satu-satunya perbedaan dalam keempat fungsi bagian ini adalah jumlah dan ukuran bagian. Lebih banyak bagian ( (a)) menghasilkan perkiraan yang lebih akurat dari fungsi yang mendasarinya, tetapi membutuhkan waktu lebih lama untuk menghasilkan dan menciptakan pekerjaan lebih lanjut untuk penyelesai IvP ketika fungsi tersebut digabungkan. Fungsi IvP tidak perlu menggunakan bagian berukuran seragam.


> **Gambar 1.7: Rendering empat fungsi IvP berbeda yang mendekati fungsi dasar yang sama**\
> Fungsi dalam (a) menggunakan distribusi seragam sebanyak 7056 buah. Fungsi dalam (b) menggunakan distribusi seragam sebanyak 1024 buah. Fungsi dalam (c) dibuat dengan terlebih dahulu membangun distribusi seragam sebanyak 49 buah dan kemudian memfokuskan penyempurnaan pada subdomain fungsi. Ini disebut penyempurnaan terarah dalam kotak peralatan IvP Build. Fungsi dalam (d) dibuat dengan terlebih dahulu membangun fungsi seragam sebanyak 25 buah dan berulang kali menyempurnakan fungsi berdasarkan buah mana yang diketahui memiliki kesesuaian yang buruk dengan fungsi dasar. Ini disebut penyempurnaan cerdas dalam kotak peralatan IvP Build.


Dengan menggunakan opsi penyempurnaan terarah di IvP Build Toolbox, fungsi IvP yang awalnya seragam dapat disempurnakan lebih lanjut dengan lebih banyak bagian di subdomain yang diarahkan oleh pemanggil, dengan bagian seragam yang lebih kecil sesuai pilihan pemanggil. Hal ini dijelaskan lebih lengkap dalam dokumentasi untuk IvP Build Toolbox. Penggunaan alat ini mengharuskan pemanggil untuk memiliki beberapa ide di mana, di subdomain, penyempurnaan lebih lanjut diperlukan atau diinginkan. Sering kali penulis perilaku memang memiliki wawasan ini. Misalnya, jika salah satu variabel domain adalah arah kendaraan, mungkin ada baiknya untuk memiliki penyempurnaan halus di sekitar nilai arah yang dekat dengan arah kendaraan saat ini.

Dalam situasi lain, wawasan tentang di mana penyempurnaan lebih lanjut diperlukan mungkin tidak tersedia bagi pemanggil. Dalam kasus ini, menggunakan opsi penyempurnaan cerdas dari IvP Build Toolbox, fungsi IvP yang awalnya seragam dapat disempurnakan lebih lanjut dengan meminta kotak alat untuk secara otomatis "menilai" bagian-bagian saat dibuat. Pemeringkatan dilakukan dalam hal seberapa akurat kecocokan linier antara fungsi linier bagian dan fungsi yang mendasarinya di atas subdomain untuk bagian tersebut. Antrean prioritas dipertahankan berdasarkan nilai, dan bagian-bagian yang kecocokannya buruk dicatat, secara otomatis disempurnakan lebih lanjut, hingga batas bagian maksimum yang dipilih oleh pemanggil. Hal ini dijelaskan lebih lengkap dalam dokumentasi untuk IvP Build Toolbox

Alat Reflector bekerja dengan cara yang sama dalam dimensi N dan pada fungsi multi-moda. Satu-satunya persyaratan untuk menggunakan alat Reflector adalah menyediakan akses ke fungsi yang mendasarinya. Karena alat tersebut secara berulang mengambil sampel fungsi ini, tantangan utama bagi pengguna kotak peralatan adalah mengembangkan implementasi fungsi yang cepat. Dalam hal waktu yang dihabiskan dalam menghasilkan fungsi IvP dengan alat Reflector, pengambilan sampel fungsi yang mendasarinya biasanya merupakan tiang panjang di dalam tenda.


#### 1.6.3   Pemecah IvP dan Bobot Prioritas Perilaku   
Pemecah IvP mengumpulkan serangkaian fungsi IvP tertimbang yang dihasilkan oleh masing-masing perilaku dan menemukan titik dalam ruang keputusan yang mengoptimalkan kombinasi tertimbang tersebut. Jika setiap fungsi objektif IvP direpresentasikan oleh , dan bobot setiap fungsi diberikan oleh , solusi untuk masalah dengan k fungsi diberikan oleh:

Algoritmanya dijelaskan secara rinci dalam \cite{?} , tetapi diringkas dalam beberapa poin berikut.

- Pohon pencarian : Struktur algoritma pencarian bersifat bercabang dan terbatas. Pohon pencarian terdiri dari fungsi IvP di setiap lapisan, dan simpul di setiap lapisan terdiri dari bagian-bagian individual dari fungsi di lapisan tersebut. Simpul daun mewakili satu bagian dari setiap fungsi. Simpul di pohon dapat direalisasikan jika bagian dari simpul tersebut dan leluhurnya berpotongan, yaitu, berbagi titik-titik umum di ruang keputusan.
- Optimalitas global : Setiap titik dalam ruang keputusan berada tepat pada satu bagian dalam setiap fungsi IvP dan dengan demikian berada tepat pada satu simpul daun dari pohon pencarian. Jika pohon pencarian diperluas sepenuhnya, atau dipangkas dengan benar (hanya ketika sub-pohon yang dipangkas tidak berisi solusi optimal), maka pencarian dijamin akan menghasilkan solusi optimal global. Algoritme pencarian yang digunakan oleh penyelesai IvP memang dimulai dengan pohon yang diperluas sepenuhnya, dan menggunakan pemangkasan yang tepat untuk menjamin optimalitas global. Algoritme tersebut memungkinkan parameter untuk jaminan kemunduran terbatas dari optimalitas global - solusi yang lebih cepat dengan jaminan berada dalam persentase tetap dari optima global. Opsi ini tidak diekspos ke Helm IvP yang selalu menemukan optimum global.
- Solusi awal : Faktor kunci dari algoritma cabang-dan-batas yang efektif adalah menyemai pencarian dengan solusi awal yang layak. Dalam Helm IvP, solusi awal yang digunakan adalah solusi (biasanya heading , speed , depth ) yang dihasilkan pada iterasi helm sebelumnya. Berdasarkan pengamatan biasa, hal ini tampaknya memberikan percepatan sekitar dua kali lipat.

Dalam kasus di mana terdapat "ikatan" antara keputusan optimal, solusi yang dihasilkan oleh pemecah masalah bersifat non-deterministik. Hal ini sedikit diringankan oleh fakta bahwa solusi tersebut disemai dengan keluaran dari iterasi sebelumnya seperti yang dibahas di atas.


#### 1.6.4   Memantau IvP Solver Selama Eksekusi Misi   
Performa solver dapat dipantau dengan alat uHelmScope . Output yang ditunjukkan di bawah ini adalah kutipan dari contoh misi. Pada baris 5, total waktu yang dibutuhkan untuk memecahkan masalah optimasi multi-objektif diberikan dalam hitungan detik, dan waktu maksimum yang dibutuhkan untuk semua loop yang terekam diberikan dalam tanda kurung. Di sini adalah nol karena hanya ada satu fungsi objektif dalam contoh ini. Pada baris 6 adalah total waktu untuk membuat fungsi IvP dalam semua perilaku, dengan waktu maksimum di semua iterasi dalam tanda kurung. Pada baris 7 adalah total waktu loop - jumlah dari dua baris sebelumnya. Perilaku aktif menampilkan informasi yang berguna mengenai solver IvP. Misalnya, pada baris 13, perilaku titik jalan Survei memiliki bobot prioritas 100 dan menghasilkan 1.227 buah, yang membutuhkan waktu CPU 0,01 detik untuk membuatnya.


**Daftar 1.3** - Contoh keluaran uHelmScope yang berisi informasi tentang penyelesai IvP.

```
   1  ==============   uHelmScope Report  ============== DRIVE  (17)
   2    Helm Iteration: 66      (hz=0.38)(5)  (hz=0.35)(66)  (hz=0.56)(max)
   3    IvP functions:  1
   4    Mode(s):        Surveying
   5    SolveTime:      0.00    (max=0.00)
   6    CreateTime:     0.02    (max=0.02)
   7    LoopTime:       0.02    (max=0.02)
   8    Halted:         false   (0 warnings)
   9  Helm Decision: [speed,0,4,21] [course,0,359,360]
  10    speed = 3.00
  11    course = 177.00
  12  Behaviors Active: ---------- (1)
  13    waypt_survey (13.0) (pwt=100.00) (pcs=1227) (cpu=0.01) (upd=0/0)
  14  Behaviors Running: --------- (0)
  15  Behaviors Idle: ------------ (1)
  16    waypt_return (22.8)
  17  Behaviors Completed: ------- (0)
  18
```

Pemecah masalah dapat dipantau dan dianalisis lebih lanjut melalui dua variabel MOOS LOOP_CPU dan CREATE_CPU yang dipublikasikan pada setiap iterasi helm. Yang pertama menunjukkan waktu tunggu sistem untuk membangun setiap fungsi IvP dan memecahkan masalah optimasi multi-objektif, dan yang terakhir menunjukkan waktu untuk membuat fungsi IvP.
