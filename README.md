# Refleksi

### Milestone 1
Method `handle_connection`:
- `BufReader`: membungkus `TcpStream` ke dalam `BufReader`untuk meningkatkan efisiensi pembacaan data dengan melakukan buffering.
- Membaca data yang masuk berdasarkan baris baru `.lines()`
- `take_while(|line| !line.is_empty())`: permintaan HTTP dipisahkan dari bagian body oleh satu baris kosong. Program akan berhenti membaca data setelah menemukan baris kosong.
- `.collect()` akan mengumpulkan semua baris teks ke dalam sebuah `Vec<String>`
Jika mengakses `127.0.0.1:7878` melalui browser, kita akan melihat detail request header seperti `GET / HTTP/1.1`, `Host`, `User-Agent`, dan lainnya.