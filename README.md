Nama: Zita Nayra Ardini
NPM: 2406404913
Kelas: Pemrograman Lanjut B

## Commit 1 Reflection notes
> 1. Jelaskan apa yang dilakukan handle_connection()!
handle_connection() adalah fungsi yang digunakan untuk menangani request yang masuk ke server. Fungsi ini akan menerima request dari browser klien, lalu request-nya dibaca baris per baris menggunakan BufferedReader. Baris ini lalu dikumpulkan ke dalam vektor string (Vec<String>) lalu dicetak ke layar.

> 2. Jelaskan kenapa pakai .take_while(|line| !line.is_empty())!
Penggunaan .take_while(|line| !line.is_empty()) bertujuan untuk membaca request baris per baris hingga bertemu baris kosong. Dalam protokol HTTP, baris kosong menandai selesainya header. Dengan cara ini, kita memastikan hanya header yang diproses, bukan body request atau konten lain yang menyusul setelah header.

## Commit 2 Reflection notes
>1. Jelaskan format HTTP response!
Format HTTP response terdiri atas status line yang berisi versi HTTP, kode status, dan reason phrase (contoh: `HTTP/1.1 200 OK`), diikuti oleh headers yang membawa informasi tambahan seperti `Content-Type` dan `Content-Length`, kemudian sebuah baris kosong yang menandai akhir dari header, dan terakhir body opsional yang berisi data respons seperti HTML, JSON, atau file lainnya.
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

> 2. Jelaskan kenapa fs::read_to_string digunakan untuk membaca file HTML, dan bagaimana response dikirim via stream.write_all()!
fs::read_to_string digunakan karena fungsi ini dapat membaca seluruh isi file HTML sekaligus dan mengembalikannya sebagai tipe String. Hal ini mempermudah penyusunan konten HTML sebagai bagian dari body HTTP response. Setelah isi file berhasil dibaca, kita dapat menyusun respons lengkap yang mencakup status line, headers, dan body. Selanjutnya, respons tersebut dikirim ke klien menggunakan stream.write_all(), yang bertugas menuliskan seluruh byte dari respons ke dalam aliran koneksi (stream). Dengan cara ini, klien akan menerima respons HTTP yang utuh beserta konten HTML yang diminta.

