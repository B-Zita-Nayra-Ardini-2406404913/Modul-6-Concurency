Nama: Zita Nayra Ardini
NPM: 2406404913
Kelas: Pemrograman Lanjut B

### Commit 1 Reflection notes
> Pada milestone ini, saya mengimplementasikan fungsi handle_connection() untuk menangani request yang masuk ke server. Fungsi ini menerima request dari browser klien, lalu membaca request tersebut baris per baris menggunakan BufferedReader. Setiap baris kemudian dikumpulkan ke dalam vektor string (Vec<String>) dan dicetak ke layar untuk debugging.

> Penggunaan .take_while(|line| !line.is_empty()) bertujuan untuk membaca request hanya sampai ditemukan baris kosong. Dalam protokol HTTP, baris kosong menandakan akhir dari header request. Dengan cara ini, saya memastikan bahwa hanya bagian header yang diproses, sementara body request (jika ada) atau konten lain setelah header tidak ikut terbaca. Ini penting agar server tidak salah menginterpretasikan data yang seharusnya tidak diproses di tahap ini.

### Commit 2 Reflection notes
>Pada milestone ini, saya memahami struktur format HTTP response. Format ini terdiri atas status line yang berisi versi HTTP, kode status, dan reason phrase (contoh: HTTP/1.1 200 OK), kemudian diikuti oleh headers yang membawa informasi tambahan seperti Content-Type dan Content-Length, lalu sebuah baris kosong sebagai penanda akhir header, dan terakhir body opsional yang berisi data respons seperti HTML, JSON, atau file lainnya.
Contoh lengkapnya:
```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 123

<html>
<body>
<h1>Hello, World!</h1>
</body>
</html>
```
> Adapun, fs::read_to_string digunakan karena fungsi ini dapat membaca seluruh isi file HTML sekaligus dan mengembalikannya sebagai tipe String. Hal ini mempermudah penyusunan konten HTML sebagai bagian dari body HTTP response. Setelah isi file berhasil dibaca, kita dapat menyusun respons lengkap yang mencakup status line, headers, dan body. Selanjutnya, respons tersebut dikirim ke klien menggunakan stream.write_all(), yang bertugas menuliskan seluruh byte dari respons ke dalam aliran koneksi (stream). Dengan cara ini, klien akan menerima respons HTTP yang utuh beserta konten HTML yang diminta.

### Commit 3 Reflection notes
> Pada milestone ini, saya memahami pentingnya validasi request. Validasi diperlukan untuk memastikan bahwa server hanya memproses permintaan yang sesuai dengan protokol HTTP dan format yang diharapkan. Hal ini penting untuk mencegah server memproses permintaan yang tidak valid, yang dapat menyebabkan kesalahan, kerentanan keamanan, atau bahkan crash pada server. Dengan melakukan validasi, saya dapat memastikan bahwa hanya permintaan yang benar-benar valid yang akan diproses, sehingga meningkatkan stabilitas dan keamanan aplikasi.

> Dalam implementasinya, validasi request dapat memeriksa beberapa aspek seperti metode HTTP yang digunakan, path URL, serta versi HTTP. Jika permintaan tidak memenuhi kriteria validasi, server dapat merespons dengan kode status HTTP yang sesuai, seperti 400 Bad Request, untuk memberi tahu klien bahwa permintaan mereka tidak valid. Dengan demikian, validasi request membantu menjaga integritas dan keamanan server serta memberikan pengalaman pengguna yang lebih baik. Dalam konteks program ini, jika klien mengirim request yang tidak valid, server akan merespons dengan status HTTP/1.1 404 NOT FOUND dan menampilkan halaman 404.html.

### Commit 4 Reflection notes
> Pada milestone ini, saya menggunakan pola match untuk mencocokkan request_line yang dikirim oleh klien. Jika request-nya "GET / HTTP/1.1", server merespons dengan status 200 OK dan menampilkan file hello.html. Jika request-nya "GET /sleep HTTP/1.1", server akan tidur selama 10 detik menggunakan thread::sleep() sebelum merespons. Untuk semua request lainnya, server akan merespons dengan status 404 NOT FOUND dan menampilkan file 404.html.

> Namun, karena server masih bersifat single-threaded, ketika satu klien sedang dilayani, klien lain harus menunggu. Masalah ini menjadi sangat serius saat ada request ke /sleep yang membutuhkan waktu 10 detik — selama waktu tersebut, semua klien lain terblokir total. Fenomena ini disebut head-of-line blocking. Akibatnya, server bisa terasa lambat, tidak responsif, atau bahkan crash jika banyak klien mencoba mengakses secara bersamaan. Dari sini saya belajar bahwa server single-threaded tidak cocok untuk aplikasi web yang melayani banyak klien. Solusinya adalah mengimplementasikan multi-threading atau asynchronous processing agar server dapat melayani banyak klien secara simultan tanpa saling memblokir.

### Commit 5 Reflection notes
> Pada milestone ini, saya mengimplementasikan multithreaded server menggunakan ThreadPool. ThreadPool bekerja dengan cara membuat sejumlah Worker di awal program, di mana setiap Worker memiliki thread sendiri yang terus berjalan menunggu job masuk melalui channel. Ketika browser mengirim request, pool.execute() membungkus closure handle_connection ke dalam sebuah Box lalu mengirimnya lewat sender ke channel. Worker yang sedang idle akan mengambil job tersebut menggunakan receiver.lock().unwrap().recv().unwrap(), di mana Arc<Mutex<>> digunakan untuk memastikan hanya satu Worker yang mengambil job pada satu waktu tanpa race condition. Dengan adanya 4 Worker, server kini mampu menangani hingga 4 request secara bersamaan tanpa saling memblokir satu sama lain.

> Urutan pelepasan kunci Mutex menjadi sangat penting. Penggunaan let job = receiver.lock().unwrap().recv().unwrap() dilanjutkan job() secara terpisah memastikan kunci Mutex dilepas segera setelah job diambil, sehingga Worker lain bisa langsung mengambil job berikutnya saat Worker pertama sedang mengerjakan requestnya. Berbeda jika menggunakan while let Ok(job) = receiver.lock().unwrap().recv() yang terlihat lebih ringkas namun justru membuat kunci Mutex tertahan sepanjang eksekusi job(), sehingga Worker lain tidak bisa mengambil job dan efeknya kembali seperti single-threaded server. Ini mengajarkan saya bahwa di Rust, lifetime dari temporary value sangat berpengaruh pada perilaku program, dan perbedaan kecil dalam penulisan kode bisa berdampak besar pada performa konkurensi.

### Bonus Reflection notes
> Pada bonus ini, saya membuat fungsi build sebagai alternatif dari fungsi new pada ThreadPool. Perbedaan utamanya terletak pada cara penanganan error — fungsi new menggunakan assert!(size > 0) yang akan langsung menyebabkan panic jika ukuran pool adalah nol, sedangkan fungsi build mengembalikan Result<ThreadPool, String> sehingga pemanggil dapat menangani kondisi error secara eksplisit menggunakan match atau unwrap_or_else. Pendekatan ini lebih idiomatik di Rust karena memberikan kontrol penuh kepada pemanggil untuk memutuskan apa yang harus dilakukan ketika pembuatan ThreadPool gagal.

> Dari perbandingan keduanya, saya memahami bahwa konvensi di Rust adalah fungsi new diasumsikan selalu berhasil dan boleh panic jika invariant tidak terpenuhi, sedangkan fungsi build atau try_new digunakan ketika kegagalan adalah sesuatu yang wajar terjadi dan harus ditangani dengan baik. Dalam konteks produksi, penggunaan build jauh lebih aman karena program tidak akan tiba-tiba crash — melainkan bisa memberikan pesan error yang informatif kepada pengguna atau mencoba strategi pemulihan lain sebelum benar-benar berhenti.