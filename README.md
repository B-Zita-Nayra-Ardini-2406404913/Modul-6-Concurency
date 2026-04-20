Nama: Zita Nayra Ardini
NPM: 2406404913
Kelas: Pemrograman Lanjut B

### Commit 1 Reflection notes
> handle_connection() adalah fungsi yang digunakan untuk menangani request yang masuk ke server. Fungsi ini akan menerima request dari browser klien, lalu request-nya dibaca baris per baris menggunakan BufferedReader. Baris ini lalu dikumpulkan ke dalam vektor string (Vec<String>) lalu dicetak ke layar.

> Penggunaan .take_while(|line| !line.is_empty()) bertujuan untuk membaca request baris per baris hingga bertemu baris kosong. Dalam protokol HTTP, baris kosong menandai selesainya header. Dengan cara ini, kita memastikan hanya header yang diproses, bukan body request atau konten lain yang menyusul setelah header.

### Commit 2 Reflection notes
> Format HTTP response terdiri atas status line yang berisi versi HTTP, kode status, dan reason phrase (contoh: `HTTP/1.1 200 OK`), diikuti oleh headers yang membawa informasi tambahan seperti `Content-Type` dan `Content-Length`, kemudian sebuah baris kosong yang menandai akhir dari header, dan terakhir body opsional yang berisi data respons seperti HTML, JSON, atau file lainnya.
Contoh:
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
> Validasi request diperlukan untuk memastikan bahwa server hanya memproses permintaan yang sesuai dengan protokol HTTP dan format yang diharapkan. Hal ini penting untuk mencegah server memproses permintaan yang tidak valid, yang dapat menyebabkan kesalahan, kerentanan keamanan, atau bahkan crash pada server. Dengan melakukan validasi, kita dapat memastikan bahwa hanya permintaan yang benar-benar valid yang akan diproses, sehingga meningkatkan stabilitas dan keamanan aplikasi.

> Dalam implementasi validasi request, kita dapat memeriksa beberapa aspek seperti metode HTTP yang digunakan, path URL, serta versi HTTP. Jika permintaan tidak memenuhi kriteria validasi, server dapat merespons dengan kode status HTTP yang sesuai, seperti 400 Bad Request, untuk memberi tahu klien bahwa permintaan mereka tidak valid. Dengan demikian, validasi request membantu menjaga integritas dan keamanan server serta memberikan pengalaman pengguna yang lebih baik. Dalam konteks program ini, jika user mengirim request yang tidak valid, statusnya akan "HTTP/1.1 404 NOT FOUND" dan menampilkan halaman 404.html.

### Commit 4 Reflection notes
> Kode di atas menggunakan match untuk mencocokkan request_line dari klien. Jika request-nya "GET / HTTP/1.1", server merespon dengan 200 OK dan file hello.html. Jika request-nya "GET /sleep HTTP/1.1", server akan tidur selama 10 detik sebelum merespon. Selain itu, server akan merespon dengan 404 NOT FOUND dan file 404.html.

> Karena server single-threaded, ketika satu klien sedang dilayani, klien lain harus menunggu. Masalah ini menjadi serius saat ada request ke /sleep yang membutuhkan waktu 10 detik. Selama waktu tersebut, semua klien lain terblokir total. Ini disebut head-of-line blocking. Akibatnya, server bisa terasa lambat, tidak responsif, atau bahkan crash jika banyak klien mencoba mengakses bersamaan. Solusinya adalah menggunakan multi-threading atau asynchronous processing agar server bisa melayani banyak klien secara simultan. 

### Commit 5 Reflection notes
> Pada milestone ini, saya mengimplementasikan multithreaded server menggunakan ThreadPool. ThreadPool bekerja dengan cara membuat sejumlah Worker di awal program, di mana setiap Worker memiliki thread sendiri yang terus berjalan menunggu job masuk melalui channel. Ketika browser mengirim request, pool.execute() membungkus closure handle_connection ke dalam sebuah Box lalu mengirimnya lewat sender ke channel. Worker yang sedang idle akan mengambil job tersebut menggunakan receiver.lock().unwrap().recv().unwrap(), di mana Arc<Mutex<>> digunakan untuk memastikan hanya satu Worker yang mengambil job pada satu waktu tanpa race condition. Dengan adanya 4 Worker, server kini mampu menangani hingga 4 request secara bersamaan tanpa saling memblokir satu sama lain.

> Urutan pelepasan kunci Mutex menjadi sangat penting. Penggunaan let job = receiver.lock().unwrap().recv().unwrap() dilanjutkan job() secara terpisah memastikan kunci Mutex dilepas segera setelah job diambil, sehingga Worker lain bisa langsung mengambil job berikutnya saat Worker pertama sedang mengerjakan requestnya. Berbeda jika menggunakan while let Ok(job) = receiver.lock().unwrap().recv() yang terlihat lebih ringkas namun justru membuat kunci Mutex tertahan sepanjang eksekusi job(), sehingga Worker lain tidak bisa mengambil job dan efeknya kembali seperti single-threaded server. Ini mengajarkan saya bahwa di Rust, lifetime dari temporary value sangat berpengaruh pada perilaku program, dan perbedaan kecil dalam penulisan kode bisa berdampak besar pada performa konkurensi.