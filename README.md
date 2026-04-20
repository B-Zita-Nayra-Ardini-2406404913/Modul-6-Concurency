Nama: Zita Nayra Ardini
NPM: 2406404913
Kelas: Pemrograman Lanjut B

## Commit 1 Reflection notes
> 1. Jelaskan apa yang dilakukan handle_connection()!
handle_connection() adalah fungsi yang digunakan untuk menangani request yang masuk ke server. Fungsi ini akan menerima request dari browser klien, lalu request-nya dibaca baris per baris menggunakan BufferedReader. Baris ini lalu dikumpulkan ke dalam vektor string (Vec<String>) lalu dicetak ke layar.

> 2. Jelaskan kenapa pakai .take_while(|line| !line.is_empty())!
Penggunaan .take_while(|line| !line.is_empty()) bertujuan untuk membaca request baris per baris hingga bertemu baris kosong. Dalam protokol HTTP, baris kosong menandai selesainya header. Dengan cara ini, kita memastikan hanya header yang diproses, bukan body request atau konten lain yang menyusul setelah header.

