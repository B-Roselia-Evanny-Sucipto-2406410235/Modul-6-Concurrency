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
Pada perubahan ini, dilakukan penambahan pengecekan kondisi. Jika klien memitna path selain `/`, maka server tidak akan memberikan respon 200 OK, melainkan mengirimkan status `HTTP/1.1 404 NOT FOUND` dan menampilkan halaman `404.html`. Kemudian, saya juga melakukan refactoring pada bagian `if` dan `else` untuk baca file dan mengirim respon. `if` digunakan sebagai pernyataan penugasaan untuk variabel `status_line` dan `filename`, memisahkan logika pembacaan file dan pengiriman respon ke luar blok condition, dan menggunakan tuple dan destructuring untuk menetapkan nilai variabel berdasarkan kondisi secara ringkas. Dengan struktur yang baru, kode menjadi lebih mudah dimaintain, dan jika suatu saat ingin mengubah cara server mengirimkan respon atau menambahkan header baru, saya hanya perlu mengubahnya di satu tempat, bukan di setiap blok `if` atau `else`.
![Commit 3 screen capture](/assets/images/commit3.png)
![Commit 3 screen capture bad](/assets/images/commit3bad.png)

### Milestone 4
Pada perubahan ini, ditambahkan rute baru `GET /sleep HTTP/1.1` yang akan menghentikan eksekusi selama 10 detik. Setelah mencoba membuka `/sleep` di satu tab browser dan segera mencoba membuka `/` di tab lain, saya menyadari bahwa halaman utama ikut tertunda dan tidak bisa terbuka sampai proses sleep 10 detik selesai. Hal ini terjadi karena server saat ini masih bersifat single threaded. Server hanya bisa memproses satu permintaan dalam satu waktu. Jika satu permintaan memakan waktu lama seperti simulasi sleep tadi, maka permintaan lain akan mengantre di belakangnya. Ini menunjukkan keterbatasan server yang tidak efisien untuk menangani banyak pengguna sekaligus.

### Milestone 5
Pada perubahan ini, saya mengubah server yang awalnya single-threaded menjadi multithreaded menggunakan `ThreadPool`. `Arc<Mutex<mpsc::Receiver<Job>>>`, bagian `Arc` memungkinkan beberapa worker memiliki kepemilikan bersama atas satu `receiver`, sedangkan `Mutex` menjamin hanya satu worker yang bisa mengakses `receiver` dalam satu waktu untuk mengambil tugas, sehingga mencegah terjadinya race condition. `mpsc::channel` adalah sistem antrean yang digunakan untuk mengirim `Job` dari main thread ke para worker. `sender` dimiliki oleh `ThreadPool`, sementara `receiver` dibagikan ke pada worker. Setiap `worker` bertanggung jawab mengelola satu thread. Ia terus menerus melakukan looping untuk mencoba mengunci mutex dan menerima tugas dari channel. Jika ada tugas, worker akan menjalankannya. Tapi jika tidak ada, worker tersebut akan menunggu tanpa memakan banyak daya CPU. Saat `pool.execute` dipanggil di `main.rs`, ia membungkus fungsi `handle_connection` ke dalam `Box` sebagai `Job`, lalu mengirimkannya lewat channel. Salah satu worker yang sedang tidak mengerjakan apapun akan menangkap job tersebut dan memproses koneksi HTTP secara independen. Sekarang, server dapat menangani beberapa permintaan secara bersamaan. Jika `/sleep` diakses, thread tersebut akan sleep selama 10 detik, namun worker lainnya akan melayani pengguna yang mengakses `/`.

### Bonus
Pada perubahan ini, saya mengimplementasikan fungsi `build` sebagai alternatif dari fungsi `new`. Pada fungsi `new`, digunakan `assert!(size > 0)`, sehingga jika pengguna memasukkan angka 0, program akan langsung berhenti paksa atau panic, dan ini dianggap kurang fleksibel. Sedangkan fungsi `build`, mengembalikan tipe `Result<ThreadPool, &'static str>`, sehingga ini memberikan kesempatan bagi pemanggil fungsi untuk menangani kesalahan tanpa menghentikan seluruh program. Jika diberikan input 0 untuk ukuran thread poolnya, maka sekarang program memberikan pesan eror yang informatif. Penggunaan `Result` memberikan kontrol yang lebih baik kepada pengembang dibandingkan penggunaan `assert!`.