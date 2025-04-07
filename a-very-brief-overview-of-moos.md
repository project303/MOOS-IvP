# Tinjauan Singkat tentang MOOS

## 1   Tinjauan Singkat tentang MOOS​ 
MOOS sering digambarkan sebagai middleware otonomi yang menyiratkan bahwa ia adalah semacam perekat yang menghubungkan sekumpulan aplikasi tempat pekerjaan sebenarnya terjadi. MOOS memang menghubungkan sekumpulan aplikasi, yang salah satunya adalah IvP Helm. MOOS bersifat lintas platform dan mandiri serta bebas ketergantungan. Ia tidak memerlukan pustaka pihak ketiga lainnya. Setiap aplikasi mewarisi antarmuka MOOS generik yang implementasinya menyediakan cara yang kuat dan mudah digunakan untuk berkomunikasi dengan aplikasi lain dan mengendalikan frekuensi relatif di mana aplikasi menjalankan serangkaian fungsi utamanya. Karena kombinasi kemudahan penggunaan, perluasan umum, dan keandalannya, ia telah digunakan di kelas oleh siswa tanpa pengalaman sebelumnya, serta pada banyak latihan lapangan yang diperluas dengan sumber daya robotik yang substansial yang dipertaruhkan. Untuk membingkai pembahasan selanjutnya tentang IvP Helm, isu-isu dasar mengenai aplikasi MOOS diperkenalkan di sini.

### 1.1   Komunikasi antarproses dengan Publish/Subscribe
MOOS memiliki topologi seperti bintang seperti yang digambarkan pada Gambar ??? . Aplikasi dalam komunitas MOOS (MOOSApp) memiliki koneksi ke satu Basis Data MOOS (disebut MOOSDB) yang terletak di inti rangkaian perangkat lunak. Semua komunikasi terjadi melalui aplikasi server pusat ini. Jaringan memiliki properti berikut:
- Tidak ada komunikasi antarteman.
- Komunikasi antara klien dan server dimulai oleh klien, yaitu, MOOSDB tidak pernah melakukan upaya yang tidak diminta untuk menghubungi MOOSApp secara tiba-tiba
- Setiap klien memiliki nama yang unik.
- Klien tertentu tidak perlu mengetahui keberadaan klien lainnya.
- Satu klien tidak mengirimkan data ke klien lain - data hanya dapat dikirim ke MOOSDB dan dari sana ke klien lain. Versi pustaka modern memiliki latensi kurang dari satu milidetik saat mengangkut muatan multi-MB antarproses.
- Jaringan bintang dapat didistribusikan ke sejumlah mesin yang menjalankan kombinasi apa pun dari sistem operasi yang didukung.
- Lapisan komunikasi mendukung sinkronisasi jam di seluruh klien yang terhubung dan pada sisi yang sama, dapat mendukung "akselerasi waktu" di mana semua klien yang terhubung beroperasi dalam aliran waktu yang dipercepat - sesuatu yang sangat berguna dalam simulasi yang melibatkan banyak proses yang didistribusikan ke banyak mesin.
- data dapat dikirim dalam potongan kecil sebagai paket "string" atau "ganda" atau dalam paket biner yang besarnya berapa pun.

### 1.2   Isi Pesan
API komunikasi dalam MOOS memungkinkan data untuk dikirimkan antara MOOSDB dan klien. Makna data tersebut bergantung pada peran klien. Akan tetapi, bentuk data tersebut tidak dibatasi oleh MOOS, meskipun demi kenyamanan, MOOS menawarkan dukungan khusus untuk muatan "ganda" dan string yang kecil. (Perlu dicatat bahwa versi MOOS yang sangat awal hanya memungkinkan data untuk dikirim sebagai string atau ganda - tetapi pembatasan ini sekarang sudah lama hilang.) Data dikemas ke dalam pesan yang berisi informasi penting lainnya yang ditunjukkan dalam .

| Variable	| Meaning | 
| ----------| --------| 
| Name	| The name of the data | 
| String Value	| Data in string format | 
| Double Value	| Numeric double float data | 
| Source	| Name of client that sent this data to the MOOSDB | 
| Auxiliary	| Supplemental message information, e.g., IvP behavior source | 
| Time	| Time at which the data was written | 
| Data Type	| Type of data (STRING or DOUBLE or BINARY) | 
| Message Type	| Type of Message (usually NOTIFICATION) | 
| Source Community	| The community to which the source process belongs | 

Seringkali lebih mudah untuk mengirim data dalam format string, misalnya string "Type=EST,Name=AUV,Pos=[3x1]{3.4,6.3,0.2 "} dapat menggambarkan estimasi posisi kendaraan yang disebut "AUV" sebagai vektor kolom 3x1. Ini dapat dibaca manusia dan tidak memerlukan pembagian dan sinkronisasi file header untuk memastikan pengirim dan penerima memahami cara menginterpretasikan data (seperti halnya dengan data biner). Sangat umum bagi aplikasi MOOS untuk berkomunikasi dengan data string dalam rangkaian pasangan "nama=nilai" yang dipisahkan koma.

- String dapat dibaca manusia.
- Semua data menjadi jenis yang sama.
- File pencatatan dapat dibaca manusia (dapat dikompresi untuk penyimpanan).
- Memutar ulang berkas log hanyalah kasus membaca string dari sebuah berkas dan "melemparnya" kembali ke MOOSDB sesuai urutan waktu.
- Konten dan susunan internal string yang dikirimkan oleh aplikasi dapat diubah tanpa perlu mengompilasi ulang konsumen (pelanggan data tersebut) - pengguna tidak akan memahami bidang data baru, tetapi mereka tidak akan mengalami crash.

Di atas adalah manfaat yang dipahami dengan baik dari pengiriman data ASCII yang dapat dijelaskan sendiri. Namun, banyak aplikasi menggunakan tipe data yang tidak memungkinkan serialisasi bertele-tele menjadi string --- pikirkan misalnya tentang data gambar kamera yang dihasilkan pada 40Hz dalam warna penuh. Pada titik ini kebutuhan untuk mengirim data biner menjadi jelas dan tentu saja MOOS mendukungnya secara transparan (dan aplikasi pLogger mendukung pencatatan dan pemutaran ulang).

Pada titik ini, pengguna harus memastikan bahwa data biner dapat diinterpretasikan oleh semua klien dan bahwa setiap gangguan pada struktur data didistribusikan dan dikompilasi ke setiap klien. Di sinilah alat serialisasi modern seperti "Google Protocol Buffers" menemukan aplikasinya. Alat ini menawarkan cara yang mudah untuk membuat serial struktur data yang kompleks menjadi aliran biner. Yang terpenting, alat ini menawarkan kompatibilitas ke depan -- memungkinkan untuk memperbarui dan menambah struktur data dengan bidang baru dengan pengetahuan yang menenangkan bahwa semua aplikasi yang ada akan tetap dapat menginterpretasikan data - alat ini tidak akan mengurai penambahan baru.

### 1.3   Penanganan Surat - Publikasikan/Berlangganan - di MOOS   
Setiap aplikasi MOOS merupakan klien yang memiliki koneksi ke MOOSDB. Koneksi ini dibuat di sisi klien dan klien mengelola mesin berulir yang mengoordinasikan komunikasi dengan MOOSDB. Hal ini sepenuhnya menyembunyikan kerumitan dan pengaturan waktu komunikasi dari aplikasi lainnya dan menyediakan serangkaian metode kecil yang terdefinisi dengan baik untuk menangani transfer data. Aplikasi dapat:

1. Publikasikan data - keluarkan pemberitahuan pada data yang disebutkan namanya.
2. Mendaftar untuk mendapatkan pemberitahuan mengenai data bernama.
3. Kumpulkan pemberitahuan pada data bernama - membaca email.

#### 1.3.1   Publikasi Data
Data dipublikasikan sebagai pasangan - variabel dan nilai - yang merupakan inti dari pesan MOOS yang dijelaskan dalam . Klien memanggil perintah Notify(VarName, VarValue) jika sesuai dalam kode klien. Perintah di atas diimplementasikan untuk nilai string, double, dan biner, dan kolom lainnya yang dijelaskan dalam diisi secara otomatis. Setiap notifikasi menghasilkan entri lain di "kotak keluar" klien, yang dalam versi MOOS yang lebih lama, dikosongkan saat berikutnya MOOSDB menerima panggilan masuk dari klien atau dalam versi terbaru, langsung dikirim ke semua klien yang berminat.

#### 1.3.2   Mendaftar untuk Notifikasi
Asumsikan bahwa daftar nama data yang dipublikasikan telah disediakan oleh penulis aplikasi MOOS tertentu. Misalnya, aplikasi yang terhubung ke sensor GPS dapat mempublikasikan data yang disebut GPS_X dan GPS_Y . Aplikasi lain dapat mendaftarkan minatnya pada data ini dengan berlangganan atau mendaftar untuk data tersebut. Aplikasi dapat mendaftar untuk notifikasi menggunakan satu metode Register() yang menentukan nama data dan tingkat maksimum klien ingin diberi tahu bahwa data telah diubah. Parameter terakhir ditentukan dalam hal waktu minimum yang memungkinkan antara notifikasi untuk variabel bernama. Misalnya, menyetelnya ke nol akan mengakibatkan klien menerima setiap notifikasi perubahan yang dikeluarkan pada variabel tersebut. MOOS V10 dan yang lebih baru juga mendukung langganan "wildcard". Misalnya, klien dapat mendaftar untuk "*:*" untuk menerima semua pesan dari semua klien lainnya. Atau "GPS_*:?NAV" untuk menerima pesan yang dimulai dengan "GPS_" dari proses apa pun dengan nama empat huruf yang diakhiri dengan "NAV" .

#### 1.3.3   Membaca Email
Klien dapat menanyakan kapan saja apakah telah menerima pemberitahuan baru dari MOOSDB dengan memanggil metode Fetch . Fungsi tersebut mengisi daftar pesan pemberitahuan dengan bidang yang diberikan dalam . Perhatikan bahwa satu panggilan ke Fetch dapat mengakibatkan munculnya beberapa pemberitahuan yang sesuai dengan data dengan nama yang sama. Ini menyiratkan bahwa beberapa perubahan telah dilakukan pada data sejak percakapan klien-server terakhir. Namun, perbedaan waktu antara pesan-pesan serupa ini tidak akan pernah kurang dari yang ditentukan dalam fungsi Register() yang dijelaskan di atas. Dalam aplikasi umum, perintah Fetch dipanggil atas nama klien tepat sebelum metode Iterate() , dan pesan-pesan ditangani dalam metode OnNewMail() yang kelebihan beban pengguna .

### 1.4   Fungsi yang kelebihan beban dalam Aplikasi MOOS [atas]   
MOOS menyediakan kelas dasar yang disebut CMOOSApp yang menyederhanakan penulisan aplikasi MOOS baru sebagai subkelas turunan. Di balik kelas CMOOSApp terdapat loop yang berulang kali memanggil fungsi yang disebut Iterate() yang secara default tidak melakukan apa pun. Salah satu tugas sebagai penulis aplikasi baru yang mendukung MOOS adalah menyempurnakan fungsi ini dengan kode yang membuat aplikasi melakukan apa yang kita inginkan. Di balik layar, loop keseluruhan dalam CMOOSApp ini juga memeriksa apakah data baru telah dikirimkan ke aplikasi. Jika sudah, fungsi virtual lain, OnNewMail() , akan dipanggil. Ini adalah fungsi tempat kode ditulis untuk memproses data yang baru dikirimkan.

> **Gambar 1.1: Fungsi virtual utama dari kelas dasar aplikasi MOOS**
> 
> Alur eksekusi setelah Run() dipanggil pada kelas yang berasal dari CMOOSApp . Gulungan menunjukkan di mana pengguna fungsionalitas CMOOSApp akan menulis kode baru yang mengimplementasikan apa pun yang diinginkan dari aplikasi baru. Perhatikan bahwa bukan itu masalahnya (seperti yang mungkin disarankan di atas) bahwa email disurvei - dalam mantra modern MOOS, email didorong ke klien dan OnNewMail() dipanggil secara sinkron segera setelah Iterate() tidak berjalan.

Peran dari tiga fungsi virtual dibahas di bawah ini. Aplikasi pHelmIvP memang mewarisi dari CMOOSApp dan membebani fungsi-fungsi ini. Kelas dasar berisi fungsi-fungsi virtual lainnya ( OnConnectToServer() dan OnDisconnectFromServer() ).

#### 1.4.1   Metode Iterate ()   
Dengan mengganti fungsi **CMOOSApp::Iterate()** dalam kelas turunan baru, penulis membuat fungsi yang darinya pekerjaan yang ditugaskan untuk dilakukan aplikasi dapat diatur. Dalam aplikasi **pHelmIvP** , metode ini akan mempertimbangkan keputusan kendaraan terbaik berikutnya, biasanya dalam bentuk menentukan nilai untuk arah kendaraan, kecepatan, dan kedalaman. Kecepatan **Iterate()** dipanggil oleh metode **SetAppFreq()** atau dengan menentukan parameter **AppTick** dalam file misi. Perhatikan bahwa frekuensi yang diminta menentukan frekuensi maksimum di mana Iterate() akan dipanggil - itu tidak menjamin bahwa itu akan dipanggil pada kecepatan yang diminta. Misalnya jika Anda menulis kode dalam Iterate() yang membutuhkan waktu 1 detik untuk menyelesaikannya, tidak mungkin metode ini dapat dipanggil pada lebih dari 1 Hz. Jika Anda ingin memanggil Iterate() secepat mungkin, cukup minta frekuensi nol - tetapi Anda mungkin ingin mempertimbangkan kembali mengapa Anda memerlukan aplikasi yang rakus seperti itu.

#### 1.4.2   Metode OnNewMail ()   
Tepat sebelum Iterate() dipanggil, kelas dasar CMOOSApp menentukan apakah ada surel baru, yaitu, apakah beberapa proses lain telah memposting data yang sebelumnya telah didaftarkan oleh klien, seperti dijelaskan di atas. Jika ada surel baru yang menunggu, kelas dasar CMOOSApp memanggil fungsi virtual **OnNewMail()** , yang biasanya dibebani oleh aplikasi. Surel tiba dalam bentuk daftar objek CMOOSMsg (lihat ). Programmer bebas untuk mengulang koleksi ini dengan memeriksa siapa yang mengirim data, apa yang berkaitan dengannya, berapa lama data tersebut, apakah data tersebut berupa string atau numerik dan untuk bertindak atau memproses data tersebut sebagaimana mestinya. Dalam versi MOOS terkini , OnNewMail() dapat dipanggil secara langsung dan cepat sebagai respons terhadap surel baru yang diterima oleh utas komunikasi back-end. Arsitektur ini memungkinkan waktu respons yang sangat cepat (sub ms) antara klien yang memposting data dan data tersebut diterima dan ditangani oleh semua pihak yang berkepentingan.

#### 1.4.3   Metode OnStartUp ()   
Fungsi **OnStartUp()** dipanggil tepat sebelum aplikasi memasuki loop abadinya sendiri yang digambarkan dalam . Ini adalah aplikasi yang mengimplementasikan kode inisialisasi aplikasi, dan khususnya membaca parameter konfigurasi (termasuk yang mengubah perilaku default kelas dasar CMOOSApp) dari sebuah file.


### 1.5   File Konfigurasi Misi MOOS   
Setiap proses MOOS dapat membaca parameter konfigurasi dari berkas misi yang menurut konvensi memiliki ekstensi **.moos** . Secara tradisional, proses MOOS berbagi berkas misi yang sama semaksimal mungkin. Misalnya, biasanya ada satu berkas misi umum untuk semua proses MOOS yang berjalan pada mesin tertentu. Setiap proses MOOS memiliki informasi yang terkandung dalam blok konfigurasi dalam berkas *.moos . Blok tersebut dimulai dengan pernyataan


>    ProcessConfig = NamaProses 


di mana ProcessName adalah nama unik yang akan digunakan aplikasi saat menghubungkan ke MOOSDB. Blok konfigurasi dibatasi oleh kurung kurawal. Di dalam kurung kurawal terdapat kumpulan pernyataan parameter, satu per baris. Setiap pernyataan ditulis sebagai:


>    ParameterName = nilai 


di mana **nilai** dapat berupa string atau nilai numerik apa pun. Semua aplikasi yang berasal dari CMOOSApp mewarisi beberapa opsi konfigurasi penting. Opsi terpenting untuk aplikasi turunan CMOOSApp adalah CommsTick dan AppTick . Yang pertama mengonfigurasi seberapa sering utas komunikasi berbicara dengan MOOSDB dan yang terakhir seberapa sering (kira-kira) Iterate() akan dipanggil.


Parameter juga dapat ditetapkan pada level "global", yaitu, tidak dalam blok konfigurasi proses tertentu. Tiga parameter yang wajib dan biasanya ditemukan di bagian atas semua file *.moos adalah: ServerHost yang menamai alamat IP yang terkait dengan server MOOSDB yang diluncurkan dengan file ini, ServerPort yang menamai nomor port tempat server MOOSDB berkomunikasi dengan klien, dan Community yang menamai komunitas yang terdiri dari server dan klien. Contoh ditunjukkan pada baris 1-3 di .



### 1.6   Meluncurkan Grup Aplikasi MOOS dengan Antler 
Antler menyediakan cara yang sederhana dan ringkas untuk memulai misi MOOS yang terdiri dari beberapa proses MOOS, alias komunitas MOOS . Misalnya, jika berkas misi yang diinginkan adalah alpha.moos , jalankan perintah berikut dari shell terminal:

```bash
    $ cd moos-ivp/ivp/misi/s1_alpha
    $ pAntler alfa.moos
```

akan meluncurkan proses yang diperlukan untuk misi tersebut. Ia membaca dari blok konfigurasinya (yang dideklarasikan sebagai ProcessConfig=ANTLER ) daftar nama proses yang akan membentuk komunitas MOOS. Setiap proses yang akan diluncurkan ditentukan dengan baris dengan sintaksis umum

di mana LaunchConfiguration adalah daftar parameter=value pair yang dipisahkan koma opsional yang secara kolektif mengontrol bagaimana proses procname (misalnya pHelmIvP , atau pLogger atau MOOSDB) diluncurkan. Parameter apa yang dapat ditentukan berada di luar cakupan diskusi ini. Antler memeriksa seluruh blok konfigurasinya dan meluncurkan satu proses untuk setiap baris yang dimulai dengan sisi kiri RUN= . Ketika semua proses telah diluncurkan, Antler menunggu semuanya keluar dan kemudian menutup dirinya sendiri.

Ada banyak aspek Antler yang tidak dibahas di sini tetapi dapat ditemukan dalam dokumentasi Antler di situs web MOOS (lihat Bagian ??? ). Ini termasuk kait untuk mengubah tampilan konsol untuk setiap proses yang diluncurkan, mengendalikan jalur pencarian untuk menentukan bagaimana eksekusi ditempatkan pada sistem berkas host, meneruskan parameter ke proses yang diluncurkan, menjalankan beberapa contoh proses tertentu, dan menggunakan Antler untuk meluncurkan beberapa komunitas berbeda pada suatu jaringan.


### 1.7   Scoping dan Poking MOOSDB   
Alat penting untuk menulis dan men-debug aplikasi MOOS (dan perilaku IvP Helm) adalah kemampuan bagi pengguna untuk berinteraksi dengan komunitas MOOS yang aktif dan melihat nilai terkini dari variabel MOOS tertentu (mencakup DB) dan mengubah satu atau beberapa variabel dengan nilai yang diinginkan (mencolok DB). Berikut ini adalah alat untuk mencakup dan mencolok masing-masing. Informasi lebih lanjut tentang masing-masing dapat ditemukan di situs web Oxford atau MIT, atau dalam beberapa kasus, bagian lain dari dokumen ini.

Alat untuk menentukan ruang lingkup MOOSDB:

- **uMS** - Alat berbasis GUI yang ditulis dalam FLTK dan dipelihara serta didistribusikan dari situs web Oxford.
- **uXMS** - Alat berbasis terminal yang dikelola dan didistribusikan dari situs web MIT
- **uHelmScope** - Alat berbasis terminal yang dikhususkan untuk menampilkan informasi tentang contoh helm yang sedang berjalan, tetapi juga berisi utilitas pelingkupan serbaguna yang mirip dengan uXMS. Didistribusikan dari situs web MIT.

Alat untuk menyodok MOOSDB:

- **uMS** - Alat berbasis GUI untuk pelingkupan, yang tercantum di atas, juga menyediakan sarana untuk melakukan poking. Didistribusikan dari situs web Oxford.
- **uPokeDB** - Alat baris perintah yang ringan untuk melakukan poking pada satu atau beberapa pasangan variabel-nilai, dengan opsi untuk menentukan cakupan pada nilai sebelum dan sesudah variabel yang dipoking sebelum keluar. Didistribusikan dari situs web MIT.
- **pMarineViewer** - Alat berbasis GUI yang utamanya digunakan untuk merender jalur kendaraan dalam ruang 2D pada tampilan Geo, tetapi juga dapat dikonfigurasi untuk menyodok DB dengan pasangan nilai variabel yang terhubung ke tombol pada tampilan. Didistribusikan dari situs web MIT.
- **uTimerScript** - Memungkinkan pengguna untuk membuat skrip serangkaian poke yang telah dikonfigurasi sebelumnya ke MOOSDB dengan setiap entri dalam skrip terjadi setelah jangka waktu tertentu. Skrip dapat dijeda atau dipercepat. Peristiwa juga dapat dikonfigurasi dengan nilai acak dan terjadi secara acak dalam kurun waktu yang dipilih. Didistribusikan dari situs web MIT.
- **uTermCommand** - Alat berbasis terminal untuk memasukkan pasangan variabel-nilai yang telah ditetapkan sebelumnya ke dalam DB. Pengguna dapat mengonfigurasi alat untuk mengaitkan alias (sependek satu karakter) guna memasukkan DB dengan cepat. Didistribusikan dari situs web MIT.
- **iRemote** - Alat berbasis terminal untuk kendali jarak jauh platform robotik yang menjalankan MOOS. Alat ini dapat dikonfigurasi untuk mengaitkan poke bernilai variabel yang telah ditetapkan sebelumnya dengan tombol apa pun yang tidak dipetakan pada keyboard. Didistribusikan dari situs web Oxford.

Daftar di atas hampir pasti bukan daftar lengkap untuk menentukan ruang lingkup dan memeriksa MOOSDB, tetapi merupakan langkah awal yang baik.

### 1.8   Aplikasi MOOS Sederhana - pXRelay   
Kumpulan aplikasi yang didistribusikan dari www.moos-ivp.org berisi aplikasi MOOS yang sangat sederhana yang disebut pXRelay . Aplikasi pXRelay mendaftarkan satu variabel MOOS input dan menerbitkan satu variabel MOOS output. Aplikasi ini membuat satu publikasi pada variabel output untuk setiap pesan email yang diterima pada variabel input. Nilai yang dipublikasikan hanyalah penghitung yang menunjukkan berapa kali variabel tersebut dipublikasikan. Dengan menjalankan dua versi pXRelay (dengan nama yang berbeda) dengan variabel input/output yang saling melengkapi, kedua proses tersebut akan mengabadikan beberapa jabat tangan publikasi/langganan dasar. Aplikasi ini didistribusikan terutama sebagai contoh sederhana dari aplikasi MOOS yang memungkinkan beberapa ilustrasi dari topik-topik berikut yang diperkenalkan hingga saat ini:

- Menemukan dan meluncurkan dengan kode contoh pAntler yang didistribusikan dengan bundel perangkat lunak MOOS-IvP.
- Contoh berkas konfigurasi misi.
- Menentukan lingkup variabel pada MOOSDB yang sedang berjalan dengan alat uXMS .
- Mencolek MOOSDB dengan pasangan nilai variabel menggunakan alat uPokeDB .
- Mengilustrasikan fungsi kelebihan beban OnStartUp() , OnNewMail() , dan Iterate() dari kelas dasar CMOOSApp .

Selain menyentuh topik ini, kumpulan file di sub-direktori kode sumber pXRelay bukanlah templat yang buruk untuk membangun modul Anda sendiri.


#### 1.8.1   Menemukan dan Meluncurkan Contoh pXRelay  
Contoh misi pXRelay harus berada di pohon direktori yang sama yang berisi kode sumber. Ada satu berkas misi, xrelay.moos :

```bash
    moos-ivp/
       MOOS/
       ivp/
          misi/
             relai x/
                xrelay.moos <---- Berkas MOOS
```

Untuk menjalankan misi ini dari jendela terminal, cukup ubah direktori dan luncurkan:

```bash
   $ cd moos-ivp/ivp/misi/xrelay
   $ pAntler xrelay.moos
```

Setelah **pAntler** meluncurkan setiap proses, seharusnya ada empat jendela terminal terbuka, satu untuk setiap proses **pXRelay** , satu untuk **uXMS** , dan satu untuk MOOSDB itu sendiri.


#### 1.8.2   Menentukan Ruang Lingkup Contoh pXRelay dengan uXMS
Di antara keempat jendela yang diluncurkan dalam contoh tersebut, jendela yang perlu diperhatikan adalah jendela uXMS , yang seharusnya memiliki keluaran serupa dengan berikut ini (dikurangi nomor baris):

Daftar 1.1 - Contoh keluaran uXMS setelah contoh pXRelay diluncurkan.

```
  0 VarName (S)sumber (T)ime (C)komunitas VarValue
  1 ---------------- ---------- --------- ---------- ----------- (73)
  2 APEL n/an/an/an/a
  3 BUAH PIR n/an/an/an/a
  4 APPLES_ITER_HZ pXRelay_APPLES 14,93 xrelay 24,93561
  5 PEARS_ITER_HZ pXRelay_PEARS 14,94 xrelay 24,93683
  6 APPLES_POST_HZ n/an/an/an/a
  7 PEARS_POST_HZ n/an/an/an/a
```

Awalnya satu-satunya hal yang berubah di jendela ini adalah bilangan bulat di akhir baris 1 yang mewakili jumlah pembaruan yang ditulis ke terminal. Di sini uXMS dikonfigurasi untuk mencakup enam variabel yang ditampilkan di kolom VarName . Kolom 2 menunjukkan proses mana yang terakhir kali memposting pada variabel, kolom 3 menunjukkan kapan posting terakhir terjadi, kolom 4 menunjukkan nama komunitas tempat posting berasal, dan kolom 5 menunjukkan nilai variabel saat ini. Entri "n/a" menunjukkan bahwa suatu proses belum menulis ke variabel yang diberikan.

Ada dua proses pXRelay yang berjalan - satu di bawah alias pXRelay_APPLES yang menerbitkan variabel APPLES sebagai variabel output-nya, APPLES_ITER_HZ yang menunjukkan frekuensi di mana fungsi Iterate() dijalankan, dan APPLES_POST_HZ yang menunjukkan frekuensi di mana variabel output diposting. Ada juga proses pXRelay_PEARS dan variabel output yang sesuai.


#### 1.8.3   Penyemaian Contoh pXRelay dengan Alat uPokeDB
Saat meluncurkan contoh **pXRelay** , satu-satunya variabel yang aktif berubah adalah variabel *_ITER_HZ (baris 4-5 di ) yang mengonfirmasi bahwa loop Iterate() di setiap proses memang sedang dijalankan. Output untuk variabel lain di mencerminkan fakta bahwa kedua proses belum memulai jabat tangan. Ini dapat dimulai dengan mengetik variabel APPLES (atau PEARS ), yang merupakan variabel input untuk pXRelay_PEARS , dengan mengetik yang berikut:

```bash
  $ cd moos-ivp/ivp/misi/xrelay
  $ uPokeDB xrelay.moos APPLES=1
```

Alat **uPokeDB** akan menerbitkan pasangan variabel-nilai yang diberikan APPLES=1 ke MOOSDB . Alat ini juga mengambil file misi, **xrelay.moos** , sebagai argumen untuk membaca informasi tentang tempat MOOSDB berjalan dalam hal nama mesin dan nomor port. Outputnya akan terlihat mirip dengan berikut ini:

**Daftar 1.2** - Contoh keluaran uPokeDB setelah melakukan poking MOOSDB dengan APPLES=1 .
```
  0 SEBELUM Mencolek MOOSDB
  1 VarName (S)sumber (T)ime VarValue
  2 ---------------- ---------- ---------- -------------
  3 APEL                 
  4  
  5  
  6 SETELAH Mencolek MOOSDB
  7 VarName (S)sumber (T)ime VarValue
  8 ---------------- ---------- ---------- -------------
  9 APEL uPokeDB 40.19 1.00000"
```

Output uPokeDB pertama-tama menunjukkan nilai variabel sebelum dicolek, dan kemudian nilai setelahnya. Setelah MOOSDB dicolek seperti di atas, aplikasi pXRelay_PEARS akan menerima email ini dan, sebagai balasannya, akan menulis ke variabel output-nya PEARS , yang selanjutnya akan dibaca oleh pXRelay_APPLES dan kedua proses akan terus menulis dan membaca variabel input dan output mereka. Perkembangan ini dapat diamati di terminal uXMS , yang mungkin terlihat seperti yang ditunjukkan di:

**Daftar 1.3** - Contoh keluaran uXMS setelah contoh pXRelay disemai.
```
  0 VarName (S)sumber (T)ime (C)komunitas VarValue
  1 ---------------- ---------- -------- ---------- ----------- (221)
  2 APEL pXRelay_APEL 44.78 xrelay 151
  3 PIR pXRelay_PEARS 44.74 xrelay 151
  4 APPLES_ITER_HZ pXRelay_APPLES 44,7 xrelay 24,90495
  5 PEARS_ITER_HZ pXRelay_PEARS 44,7 xrelay 24,90427
  6 APPLES_POST_HZ pXRelay_APPLES 44,79 xrelay 8,36411
  7 PEARS_POST_HZ pXRelay_PEARS 44,74 xrelay 8,36406
```


Pada setiap penulisan ke MOOSDB, nilai variabel bertambah 1, dan perkembangan bilangan bulat dapat dipantau di kolom terakhir pada baris 2-3. Variabel APPLES_POST_HZ dan PEARS_POST_HZ mewakili frekuensi proses membuat posting ke MOOSDB. Ini tentu saja berbeda dari (tetapi dibatasi di atas oleh) frekuensi loop Iterate() karena posting dibuat dalam loop Iterate() hanya jika email telah diterima sebelum dimulainya loop. Di dunia tanpa latensi, seseorang mungkin mengharapkan frekuensi "posting" menjadi tepat setengah dari frekuensi "iterate". Kami akan mengharapkan frekuensi yang dilaporkan pada baris 6-7 tidak lebih besar dari 12,5, dan dalam kasus ini nilai sekitar 8,4 diamati sebagai gantinya.


#### 1.8.4   File Konfigurasi MOOS Contoh pXRelay
Berkas misi yang digunakan untuk contoh pXRelay , xrelay.moos dibahas di sini. Berkas ini disediakan sebagai bagian dari bundel perangkat lunak MOOS-IvP di bawah direktori "misi" sebagaimana dibahas di atas. Berkas ini dibahas di sini dalam tiga bagian mulai dari awal hingga akhir.

Bagian dari berkas xrelay.moos menyediakan tiga informasi wajib yang dibutuhkan oleh proses MOOSDB untuk peluncuran. MOOSDB adalah server dan pada baris 1 terdapat alamat IP untuk mesin tersebut, dan baris 2 menunjukkan nomor port tempat klien dapat menemukan MOOSDB setelah diluncurkan. Karena setiap MOOSDB dan kumpulan klien yang terhubung membentuk "komunitas" MOOS, nama komunitas disediakan pada baris 3. Perhatikan nama komunitas xrelay dalam berkas xrelay.moos dan nama komunitas di kolom 4 keluaran uXMS di atas.

**Daftar 1.4** - Berkas misi xrelay.moos untuk contoh pXRelay.
```
   1 ServerHost = host lokal
   2 Port Server = 9000
   3 Komunitas = xrelay
   4  
   5 //-------------------------------------------
   6 // Blok konfigurasi Antler
   7 ProsesConfig = ANTLER
   8 {
   9 MSAntaraPeluncuran = 200
  10  
  11 Jalankan = MOOSDB @ NewConsole = benar
  12 Jalankan = pXRelay @ NewConsole = benar ~ pXRelay_PEARS
  13 Jalankan = pXRelay @ NewConsole = benar ~ pXRelay_APPLES
  14 Jalankan = uXMS @ NewConsole = benar
  15 }
```

Blok konfigurasi pada baris 7-15 dari xrelay.moos dibaca oleh pAntler untuk meluncurkan proses atau klien komunitas MOOS. Baris 9 menentukan berapa lama waktu, dalam milidetik, antara peluncuran proses. Baris 11-14 memberi nama empat aplikasi MOOS yang diluncurkan dalam contoh ini. Pada baris ini, komponen "NewConsole = true" menentukan apakah jendela konsol baru akan dibuka untuk setiap proses. Coba ubah ke false - hanya jendela uXMS yang benar-benar perlu dibuka. Yang lainnya hanya memberikan konfirmasi visual bahwa suatu proses telah diluncurkan. Komponen {\small "\verb=~ pXRelay_PEARS="} pada baris 12 dan 13 memberi tahu pAntler untuk meluncurkan aplikasi ini dengan alias yang diberikan. Ini diperlukan di sini karena setiap klien MOOS perlu memiliki nama yang unik, dan dalam contoh ini dua contoh proses pXRelay sedang diluncurkan.

Pada baris 17-39 di bawah, dua aplikasi pXRelay dikonfigurasi. Perhatikan bahwa argumen untuk ProcessConfig pada baris 20 dan 32 adalah alias untuk pXRelay yang ditetapkan dalam blok konfigurasi Antler pada baris 12 dan 13. Setiap proses pXRelay dikonfigurasi sedemikian rupa sehingga variabel MOOS yang masuk dan keluar saling melengkapi pada baris 25-26 dan 37-38. Perhatikan parameter AppTick (lihat ) ditetapkan ke 25 di kedua blok konfigurasi, dan bandingkan dengan frekuensi fungsi Iterate() yang diamati yang dilaporkan dalam variabel APPLES_ITER_HZ dan PEARS_ITER_HZ di . MOOS telah melakukan pekerjaan yang cukup baik dalam contoh ini untuk menghormati frekuensi loop Iterate() yang diminta di setiap aplikasi.

**Daftar 1.5** - Berkas misi xrelay.moos - mengonfigurasi proses pXRelay.
```
  17 //-------------------------------------------
  18 // blok konfigurasi pXRelay
  19  
  20 Konfigurasi Proses = pXRelay_APPLES
  21 {
  22 Tanda centang = 25
  23 CommsTick = 25
  24  
  25 OUTGOING_VAR = APEL
  26 VAR MASUK = PIR
  27 }
  28  
  29 //-------------------------------------------
  30 // blok konfigurasi pXRelay
  31  
  32 Konfigurasi Proses = pXRelay_PEARS
  33 {
  34 Tanda Aplikasi = 25
  35 CommsTick = 25
  36  
  37 VAR_MASUK = APEL
  38 KELUAR_VAR = PIR
  39 }
```

Di bagian terakhir dari berkas xrelay.moos , yang ditunjukkan di bawah, proses uXMS dikonfigurasi. Dalam contoh ini, uXMS dikonfigurasi untuk mencakup enam variabel yang ditentukan pada baris 54-59 untuk memberikan keluaran yang ditunjukkan dalam dan . Dengan menyetel parameter stopped pada baris 49 ke false , keluaran uXMS diperbarui secara terus-menerus dan otomatis - dalam kasus ini empat kali per detik karena laju 4Hz yang ditentukan dalam baris 46-47. Parameter display_* pada baris 50-52 memastikan bahwa keluaran dalam kolom 2-4 dari keluaran uXMS diperluas.

**Daftar 1.6** - Mengonfigurasi uXMS dalam contoh pXRelay.
```
  41 //--------------------------------------------
  42 // blok konfigurasi uXMS
  43  
  44 Konfigurasi Proses = uXMS
  45 {
  46 Tanda Aplikasi = 4
  47 CommsTick = 4
  48  
  49 dijeda = salah
  50 display_source = benar
  51 display_time = benar
  52 display_community = benar
  53  
  54 var = APEL
  55 var = BUAH PIR
  56 var = APPLES_ITER_HZ
  57 var = PEARS_ITER_HZ
  58 var = APPLES_POST_HZ
  59 var = PEARS_POST_HZ
  60 }
```

#### 1.8.5   Saran untuk Hal-hal Lanjutan yang Dapat Dicoba dengan Contoh Ini  

- Lihatlah metode OnStartUp() di kelas XRelay.cpp dalam modul pXRelay dalam bundel perangkat lunak untuk melihat bagaimana penanganan parameter dalam berkas konfigurasi xrelay.moos diimplementasikan, dan langganan untuk variabel MOOS.
- Lihatlah metode OnNewMail() di kelas XRelay.cpp dalam modul pXRelay dalam bundel perangkat lunak untuk melihat bagaimana surat masuk diurai dan ditangani.
- Lihatlah metode Iterate() di kelas XRelay.cpp di modul pXRelay dalam bundel perangkat lunak untuk melihat contoh proses MOOS yang bertindak atas surat masuk dan memposting secara kondisional ke MOOSDB
- Coba ubah parameter AppTick di salah satu blok konfigurasi pXRelay dalam file xrelay.moos , mulai ulang, dan catat perubahan yang dihasilkan dalam iterasi dan frekuensi pasca dalam keluaran uXMS .
- Coba ubah parameter CommsTick di salah satu blok konfigurasi pXRelay dalam berkas xrelay.moos ke sesuatu yang jauh lebih rendah daripada parameter AppTick , mulai ulang, dan catat perubahan yang dihasilkan dalam iterasi dan frekuensi pasca dalam keluaran uXMS .


### 1.9   Aplikasi MOOS Tersedia untuk Publik
Berikut ini adalah deskripsi singkat mengenai aplikasi MOOS yang berada dalam domain publik. Ini bukanlah daftar yang lengkap. Daftar ini tidak mencakup aplikasi di luar MIT dan Oxford, dan bahkan bukan daftar lengkap aplikasi dari organisasi tersebut.


#### 1.9.1   Modul MOOS dari Oxford
- **pAntler** : Alat untuk meluncurkan kumpulan proses MOOS yang diberikan berkas misi.
- **pShare** : Alat yang memungkinkan pesan berpindah antar komunitas dan memungkinkan penggantian nama pesan saat pesan dipindahkan antar komunitas.
- **pLogger** : Sebuah logger untuk merekam aktivitas sesi MOOS. Dapat dikonfigurasi untuk merekam sebagian atau seluruh publikasi sejumlah variabel MOOS.
- **uMS** : Cakupan MOOS berbasis GUI untuk memantau satu atau lebih MOOSDB.
- **uPlayback** : Aplikasi GUI lintas platform berbasis FLTK yang dapat memuat berkas log dan memutarnya kembali ke komunitas MOOS seolah-olah pembuat data benar-benar berjalan dan mengeluarkan pemberitahuan.
- **iMatlab** : Aplikasi yang memungkinkan Matlab untuk bergabung dengan komunitas MOOS - meskipun hanya untuk mendengarkan dan menyajikan data sensor. Aplikasi ini memungkinkan koneksi ke MOOSDB dan akses ke port serial lokal.
- **iRemote** : Alat berbasis terminal untuk kendali jarak jauh platform robotik yang menjalankan MOOS. Alat ini dapat dikonfigurasi untuk mengaitkan poke bernilai variabel yang telah ditetapkan sebelumnya dengan tombol apa pun yang tidak dipetakan pada keyboard.


#### 1.9.2   Modul Pemantauan Misi
Modul pemantauan misi membantu pengguna untuk memantau misi secara menyeluruh saat misi berlangsung, atau membantu pengguna menganalisis dan men-debug misi. Dalam rilis 13.2, ini mencakup dua alat baru yang canggih untuk pemantauan appcast, uMAC dan uMACView . pMarineViewer juga telah ditingkatkan secara substansial untuk mendukung tampilan appcast.

- **pMarineViewer** : Alat GUI untuk merender kejadian di area operasi kendaraan. Alat ini memperbarui posisi kendaraan secara berulang dari laporan node yang masuk, dan akan merender beberapa tipe geometri yang dipublikasikan dari aplikasi MOOS lainnya. Penampil juga dapat mengirim pesan ke MOOSDB berdasarkan kejadian keyboard atau mouse yang dikonfigurasi pengguna. Lihat dokumentasi daring untuk pMarineViewer .
- **uHelmScope** : Cakupan berbasis terminal (non-GUI) pada proses Helm IvP yang sedang berjalan, dan variabel MOOS utama. Cakupan ini menyediakan ringkasan perilaku, status aktivitas, dan posting perilaku terkini ke MOOSDB. Alat yang sangat berguna untuk men-debug anomali helm. Lihat dokumentasi untuk uHelmScope .
- **uXMS** : Alat berbasis terminal (non GUI) untuk menentukan cakupan MOOSDB. Pengguna dapat mengonfigurasi secara tepat rangkaian variabel yang ingin mereka tentukan cakupannya dengan menamainya secara eksplisit pada baris perintah atau di blok konfigurasi MOOS. Rangkaian variabel juga dapat dikonfigurasi dengan menamai satu atau beberapa proses MOOS tempat semua variabel yang dipublikasikan oleh proses tersebut akan ditentukan cakupannya. Pengguna juga dapat menentukan cakupan berdasarkan riwayat satu variabel. Lihat dokumentasi untuk uXMS .
- **uProcessWatch** : Aplikasi ini memantau keberadaan aplikasi MOOS pada daftar pantauan. Jika satu atau beberapa aplikasi diketahui tidak ada, maka akan dicatat pada variabel MOOS PROC_WATCH_SUMMARY . uProcessWatch mendukung appcast dan akan menghasilkan ringkasan tabel singkat dari proses yang dipantau dan beban CPU yang dilaporkan oleh proses itu sendiri. Item pada daftar pantauan dapat diberi nama secara eksplisit dalam berkas konfigurasi atau disimpulkan dari blok Antler atau dari daftar DB_CLIENTS . Aplikasi dapat dikecualikan dari daftar pantauan jika diinginkan. Lihat dokumentasi untuk uProcessWatch .
- **uMAC** : Aplikasi uMAC adalah utilitas untuk Memantau AppCast. Aplikasi ini diluncurkan dan dijalankan di jendela terminal dan akan mengurai appcast yang dihasilkan dalam komunitas MOOS-nya sendiri atau appcast dari komunitas MOOS lain yang dijembatani atau dibagikan ke MOOSDB lokal. Keuntungan utama uMAC dibandingkan alat pemantauan appcast lainnya adalah pengguna dapat masuk ke kendaraan dari jarak jauh melalui ssh dan meluncurkan uMAC secara lokal di terminal. Lihat dokumentasi untuk uMAC .
- **uMACView** : Alat GUI untuk memantau appcast secara visual. Alat ini akan mengurai appcast yang dihasilkan dalam komunitas MOOS-nya sendiri atau appcast dari komunitas MOOS lain yang dijembatani atau dibagikan ke MOOSDB lokal. Kemampuannya hampir identik dengan kemampuan tampilan appcast yang dibangun dalam pMarineViewer. Alat ini dimaksudkan untuk menjadi penampil appcast bagi pengguna non-pMarineViewer. Lihat dokumentasi untuk uMACView .


#### 1.9.3   Modul Eksekusi Misi 
Modul pelaksanaan misi berpartisipasi langsung dalam pelaksanaan misi yang tepat daripada sekadar membantu memantau, merencanakan, atau menganalisis misi.

- **pNodeReporter** : Alat untuk mengumpulkan informasi node seperti posisi kendaraan saat ini, lintasan dan jenisnya, dan mempostingnya dalam satu laporan untuk dibagikan antar kendaraan atau dikirim ke layar di tepi pantai. Lihat dokumentasi untuk pNodeReporter .
- **pContactMgrV20** : Pengelola kontak menangani kendaraan lain yang diketahui di sekitarnya. Pengelola ini menangani laporan masuk yang mungkin diterima melalui aplikasi sensor atau melalui tautan komunikasi. Minimal, pengelola ini mengeposkan laporan ringkasan ke MOOSDB, tetapi dapat juga dikonfigurasi untuk mengeposkan peringatan dengan konten yang dikonfigurasi pengguna tentang satu atau beberapa kontak. Dapat digunakan bersama dengan kemudi untuk memunculkan perilaku terkait kontak untuk menghindari tabrakan, pelacakan, dll. Lihat dokumentasi untuk pContactMgrV20 .
- **pEchoVar** : Alat untuk berlangganan variabel dan menerbitkannya kembali dengan nama yang berbeda. Alat ini juga dapat digunakan untuk menarik bidang tertentu dalam publikasi string yang terdiri dari pasangan parameter=nilai yang dipisahkan koma, menerbitkan string baru menggunakan parameter yang berbeda. Lihat dokumentasi untuk pEchoVar .
- **pSearchGrid** : Aplikasi untuk menyimpan riwayat posisi kendaraan dalam kisi 2D yang ditentukan di wilayah operasi.


#### 1.9.4   Modul Simulasi Misi  
Modul simulasi misi hanya digunakan dalam simulasi. Banyak aplikasi dalam uField Toolbox juga dapat dianggap sebagai modul simulasi, tetapi modul-modul tersebut juga memiliki kasus penggunaan yang melibatkan sensor simulasi pada kendaraan fisik yang sebenarnya. Dua modul di bawah ini murni untuk kendaraan simulasi.

- **uSimMarineV22** : Simulator kendaraan 3D sederhana yang memperbarui status, posisi, dan lintasan kendaraan, berdasarkan nilai aktuator saat ini dan status kendaraan sebelumnya. Skenario penggunaan umum memiliki satu contoh uSimMarineV22 yang dikaitkan dengan setiap kendaraan yang disimulasikan. Lihat dokumentasi untuk uSimMarineV22 .
- **uSimCurrent** : Aplikasi sederhana untuk simulasi dampak arus air. Berdasarkan informasi arus lokal dari berkas yang diberikan, aplikasi ini berulang kali membaca posisi kendaraan saat ini dan menerbitkan vektor arus, yang mungkin digunakan oleh uSimMarine.


#### 1.9.5   Modul untuk Poking MOOSDB   
Mengoperasikan MOOSDB merupakan bagian umum dan penting dari pelaksanaan misi dan/atau perintah dan kontrol. Alat pMarineViewer juga berisi beberapa metode untuk mengoperasikan MOOSDB atas perintah pengguna.

- **uPokeDB** : Alat baris perintah untuk menyodok MOOSDB dengan pasangan variabel-nilai yang disediakan pada baris perintah. Alat ini menemukan MOOSDB melalui berkas misi yang disediakan pada baris perintah, atau alamat IP dan nomor port yang diberikan pada baris perintah. Alat ini akan terhubung ke DB, menunjukkan nilai sebelum menyodok, menyodok DB, dan menunggu email dari DB untuk mengonfirmasi hasil dari menyodok. Lihat dokumentasi untuk uPokeDB .
- **uTimerScript** : Memungkinkan pengguna untuk membuat skrip serangkaian poke yang telah dikonfigurasi sebelumnya ke MOOSDB dengan setiap entri dalam skrip terjadi setelah jangka waktu tertentu. Skrip dapat dijeda atau dipercepat. Peristiwa juga dapat dikonfigurasi dengan nilai acak dan terjadi secara acak dalam kurun waktu yang dipilih. Lihat dokumentasi untuk uTimerScript .
- **uTermCommand** : Aplikasi terminal untuk memasukkan pasangan variabel-nilai yang telah ditentukan sebelumnya ke dalam MOOSDB. Kunci unik dapat dikaitkan dengan setiap masukan. Lihat dokumentasi untuk uTermCommand .



#### 1.9.6   Kotak Alat Alog   
Alog Toolbox adalah seperangkat alat offline untuk menganalisis dan memanipulasi file alog yang dihasilkan oleh aplikasi pLogger yang didistribusikan dengan basis kode Oxford MOOS.

- **alogscan** : Alat baris perintah untuk melaporkan konten file MOOS .alog tertentu. Lihat dokumentasi untuk alogscan , bagian dari Alog Toolbox.
- **alogclip** : Alat baris perintah yang akan membuat file MOOS .alog baru dari file .alog tertentu dengan menghapus entri di luar rentang waktu tertentu. Lihat dokumentasi untuk alogclip , bagian dari Alog Toolbox.
- **aloggrep** : Alat baris perintah yang akan membuat file MOOS .alog baru dengan hanya menyimpan variabel MOOS atau sumber yang diberikan dari file .alog tertentu. Lihat dokumentasi untuk aloggrep , bagian dari Alog Toolbox.
- **alogrm** : Alat baris perintah yang akan membuat file MOOS .alog baru dengan menghapus variabel atau sumber MOOS yang diberikan dari file .alog yang diberikan. Lihat dokumentasi untuk alogrm , bagian dari Alog Toolbox.
- **alogview** : Alat GUI untuk menganalisis misi kendaraan dengan memetakan satu atau beberapa lintasan kendaraan di area operasi, sambil melihat plot nilai numerik apa pun dalam berkas alog. Lihat dokumentasi untuk alogview , bagian dari Kotak Alat Alog.


#### 1.9.7   Kotak Alat uField   
Kotak Perkakas uField berisi sejumlah alat untuk mendukung misi multi-kendaraan di mana setiap kendaraan terhubung ke masyarakat pesisir. Ini mencakup simulasi dan eksperimen lapangan nyata. Kotak Perkakas ini juga berisi sejumlah sensor simulasi yang bekerja di luar kendaraan di pesisir.

- **pHostInfo** : Mendeteksi secara otomatis informasi host kendaraan termasuk alamat IP, port yang digunakan oleh MOOSDB, port yang digunakan oleh pShare lokal untuk mendengarkan UDP, dan nama komunitas untuk MOOSDB lokal. Memposting informasi ini untuk memfasilitasi komunikasi antar kendaraan secara otomatis, khususnya dalam skenario multi-kendaraan di mana alamat IP lokal berubah dengan DHCP.
- **uFldNodeBroker** : Biasanya dijalankan pada kendaraan atau kendaraan simulasi dalam konteks multi-kendaraan. Digunakan untuk membuat koneksi ke komunitas pesisir dengan mengirimkan informasi lokal tentang kendaraan seperti alamat IP, nama komunitas, dan nomor port yang digunakan oleh pShare untuk pesan UDP yang masuk. Agaknya komunitas pesisir menggunakan ini untuk mengetahui ke mana harus mengirim pesan UDP keluar ke kendaraan. Lihat dokumentasi untuk uFldNodeBroker , bagian dari uField Toolbox.
- **uFldShoreBroker** : Biasanya dijalankan di komunitas pesisir. Mengambil laporan dari kendaraan jarak jauh yang menjelaskan cara menghubungi mereka. Mengirim permintaan pendaftaran ke pShare di pesisir untuk menjembatani daftar variabel yang disediakan pengguna ke kendaraan. Setelah mengetahui kendaraan, JAKE akan membuat jembatan FOO_ALL dan FOO_JAKE ke JAKE, untuk semua variabel yang dikonfigurasi pengguna tersebut. Lihat dokumentasi untuk uFldShoreBroker , bagian dari uField Toolbox.
- **uFldNodeComms** : Alat di tepi pantai untuk mengelola komunikasi antar kendaraan. Alat ini memiliki pengetahuan tentang semua posisi kendaraan berdasarkan laporan node yang masuk. Komunikasi dapat dibatasi berdasarkan jangkauan kendaraan, frekuensi pesan, atau ukuran pesan. Pesan juga dapat diblokir berdasarkan afiliasi tim. Lihat dokumentasi untuk uFldNodeComms , bagian dari uField Toolbox.
- **uFldMessageHandler** : Alat untuk menangani pesan masuk dari node lain. Pesan tersebut berupa string yang berisi sumber dan tujuan pesan serta variabel dan nilai MOOS. Aplikasi ini cukup mengeposkan konten pasangan variabel-nilai pesan ke MOOSDB lokal. Lihat dokumentasi untuk uFldMessageHandler , bagian dari uField Toolbox.
- **uFldScope** : Biasanya dijalankan di komunitas pesisir. Mengambil informasi dari serangkaian laporan masuk yang dikonfigurasi pengguna dan mengurai informasi penting ke dalam format tabel yang ringkas. Laporan dapat berupa laporan apa pun dalam bentuk pasangan parameter-nilai yang dipisahkan koma.
- **uFldPathCheck** : Biasanya dijalankan di komunitas pesisir. Mengambil laporan simpul dari kendaraan jarak jauh dan menghitung kecepatan kendaraan saat ini serta jarak tempuh total dan mempostingnya dalam dua laporan ringkas. Penghitungan odometri dapat disetel ulang ke nol oleh aplikasi lain. Lihat dokumentasi untuk uFldPathCheck , bagian dari uField Toolbox.
- **uFldHazardSensor** : Biasanya dijalankan di komunitas pesisir. Dikonfigurasi dengan sekumpulan objek dengan lokasi x,y dan klasifikasi tertentu (bahaya atau bahaya). Simulator sensor menerima serangkaian permintaan dari kendaraan jarak jauh. Ketika sensor menentukan bahwa suatu objek berada dalam medan sensor kendaraan yang meminta, sensor mungkin atau mungkin tidak mengembalikan laporan deteksi sensor untuk objek tersebut, dan mungkin juga klasifikasi yang tepat. Peluang menerima deteksi dan klasifikasi yang tepat bergantung pada konfigurasi sensor dan preferensi pengguna untuk P_D/P_FA pada kurva ROC yang berlaku.
- **uFldHazardMetric** : Aplikasi untuk menilai laporan bahaya yang masuk, mungkin dibuat oleh pengguna uFldHazardSensor setelah menjelajahi medan bahaya yang disimulasikan.
- **uFldHazardMgr** : uFldHazardMgr adalah aplikasi MOOS sederhana untuk mengelola informasi sensor bahaya dan pembuatan laporan bahaya selama misi pencarian otonom.
- **uFldBeaconRangeSensor** : Biasanya dijalankan di komunitas pesisir. Dikonfigurasi dengan satu atau beberapa beacon dengan lokasi beacon yang diketahui. Menerima permintaan jangkauan dari kendaraan jarak jauh dan mengembalikan laporan jangkauan yang menunjukkan jangkauan kendaraan tersebut ke beacon terdekat. Permintaan jangkauan mungkin dijawab atau tidak, tergantung pada jangkauan ke beacon. Laporan mungkin memiliki tambahan noise dan mungkin atau mungkin tidak menyertakan ID beacon. Lihat dokumentasi untuk uFldBeaconRangeSensor , bagian dari uField Toolbox.
- **uFldContactRangeSensor** : Biasanya dijalankan di komunitas pesisir. Mengambil laporan dari kendaraan jarak jauh, mencatat posisi mereka. Mengambil permintaan jangkauan dari kendaraan jarak jauh dan mengembalikan laporan jangkauan yang menunjukkan jangkauan kendaraan tersebut ke kendaraan di dekatnya. Permintaan jangkauan mungkin dijawab atau tidak, tergantung pada jangkauan antar-kendaraan. Laporan mungkin juga memiliki noise yang ditambahkan ke nilai jangkauannya. Lihat dokumentasi untuk uFldContactRangeSensor , bagian dari uField Toolbox.



