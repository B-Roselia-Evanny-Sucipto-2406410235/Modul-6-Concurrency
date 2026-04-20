# Refleksi

### Milestone 1
Method `handle_connection`:
- `BufReader`: membungkus `TcpStream` ke dalam `BufReader`untuk meningkatkan efisiensi pembacaan data dengan melakukan buffering.
- Membaca data yang masuk berdasarkan baris baru `.lines()`
- `take_while(|line| !line.is_empty())`: permintaan HTTP dipisahkan dari bagian body oleh satu baris kosong. Program akan berhenti membaca data setelah menemukan baris kosong.
- `.collect()` akan mengumpulkan semua baris teks ke dalam sebuah `Vec<String>`

Jika mengakses `127.0.0.1:7878` melalui browser, kita akan melihat detail request header seperti `GET / HTTP/1.1`, `Host`, `User-Agent`, dan lainnya.

### Milestone 2
`handle_connection` dimodifikasi agar tidak hanya membaca request, tetapi juga memberikan respon balik kepada browser berupa file HTML.
- `fs::read_to_string("hello.html")` untuk membaca isi dari file `hello.html`
- Respon HTTP yang valid harus memiliki struktur tertentu, seperti status line `HTTP/1.1 200 OK` yang menandakan permintaan berhasil diproses, header `Content-Length: {length}` memberi tahu browser berapa data yang akan dikirim, blank line, dan body `{contents}` yang berisi teks dari `hello.html`
- `stream.write_all(response.as_bytes())` mengembalikan data ke klien dalam bentuk bytes, penggunaan `write_all` memastikan bahwa seluruh isi respon terkirim melalui koneksi TCP

Sekarang, ketika mengakses `http://127.0.0.1:7878`, browser tidak lagi menunjukkan status loading, tetapi browser akan merender dan menampilkan isi dari file `hello.html`, karena sudah menerima response 200 OK dari server.
![Commit 2 screen capture](/assets/images/commit2.png)

### Milestone 3
Pada perubahan ini, dilakukan penambahan pengecekan kondisi. Jika klien memitna path selain `/`, maka server tidak akan memberikan respon 200 OK, melainkan mengirimkan status `HTTP/1.1 404 NOT FOUND` dan menampilkan halaman `404.html`. Kemudian, saya juga melakukan refactoring pada bagian `if` dan `else` untuk baca file dan mengirim respon. `if` digunakan sebagai pernyataan penugasaan untuk variabel `status_line` dan `filename`, memisahkan logika pembacaan file dan pengiriman respon ke luar blok condition, dan menggunakan tuple dan destructuring untuk menetapkan nilai variabel berdasarkan kondisi secara ringkas.
![Commit 3 screen capture](/assets/images/commit3.png)
![Commit 3 screen capture bad](/assets/images/commit3bad.png)