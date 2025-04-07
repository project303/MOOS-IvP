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

#### 1.3.1   Publikasi Data    [atas]
Data dipublikasikan sebagai pasangan - variabel dan nilai - yang merupakan inti dari pesan MOOS yang dijelaskan dalam . Klien memanggil perintah Notify(VarName, VarValue) jika sesuai dalam kode klien. Perintah di atas diimplementasikan untuk nilai string, double, dan biner, dan kolom lainnya yang dijelaskan dalam diisi secara otomatis. Setiap notifikasi menghasilkan entri lain di "kotak keluar" klien, yang dalam versi MOOS yang lebih lama, dikosongkan saat berikutnya MOOSDB menerima panggilan masuk dari klien atau dalam versi terbaru, langsung dikirim ke semua klien yang berminat.

#### 1.3.2   Mendaftar untuk Notifikasi    [atas]
Asumsikan bahwa daftar nama data yang dipublikasikan telah disediakan oleh penulis aplikasi MOOS tertentu. Misalnya, aplikasi yang terhubung ke sensor GPS dapat mempublikasikan data yang disebut GPS_X dan GPS_Y . Aplikasi lain dapat mendaftarkan minatnya pada data ini dengan berlangganan atau mendaftar untuk data tersebut. Aplikasi dapat mendaftar untuk notifikasi menggunakan satu metode Register() yang menentukan nama data dan tingkat maksimum klien ingin diberi tahu bahwa data telah diubah. Parameter terakhir ditentukan dalam hal waktu minimum yang memungkinkan antara notifikasi untuk variabel bernama. Misalnya, menyetelnya ke nol akan mengakibatkan klien menerima setiap notifikasi perubahan yang dikeluarkan pada variabel tersebut. MOOS V10 dan yang lebih baru juga mendukung langganan "wildcard". Misalnya, klien dapat mendaftar untuk "*:*" untuk menerima semua pesan dari semua klien lainnya. Atau "GPS_*:?NAV" untuk menerima pesan yang dimulai dengan "GPS_" dari proses apa pun dengan nama empat huruf yang diakhiri dengan "NAV" .

#### 1.3.3   Membaca Email    [atas]
Klien dapat menanyakan kapan saja apakah telah menerima pemberitahuan baru dari MOOSDB dengan memanggil metode Fetch . Fungsi tersebut mengisi daftar pesan pemberitahuan dengan bidang yang diberikan dalam . Perhatikan bahwa satu panggilan ke Fetch dapat mengakibatkan munculnya beberapa pemberitahuan yang sesuai dengan data dengan nama yang sama. Ini menyiratkan bahwa beberapa perubahan telah dilakukan pada data sejak percakapan klien-server terakhir. Namun, perbedaan waktu antara pesan-pesan serupa ini tidak akan pernah kurang dari yang ditentukan dalam fungsi Register() yang dijelaskan di atas. Dalam aplikasi umum, perintah Fetch dipanggil atas nama klien tepat sebelum metode Iterate() , dan pesan-pesan ditangani dalam metode OnNewMail() yang kelebihan beban pengguna .
