# Reflection

### Commit 1 Reflection Notes
#### Inside handle_connection method

Fungsi `handle_connection` menerima parameter `TcpStream` yang merepresentasikan 
koneksi antara server dan client. Di dalam fungsi tersebut, `BufReader` berfungsi untuk membaca data
HTTP request dari stream secara efisien. HTTP request yang dikirim oleh browser terdiri dari 
beberapa baris seperti request line (misalnya "GET / HTTP/1.1") dan header seperti Host, 
User-Agent, dan lainnya.

Metode `.lines()` digunakan untuk membaca setiap baris dari request, lalu `
.map(|result| result.unwrap())` digunakan untuk mengambil nilai string dari hasil pembacaan. 
Setelah itu, `.take_while(|line| !line.is_empty())` berfungsi untuk menghentikan pembacaan 
ketika mencapai baris kosong, yang berarti akhir dari header HTTP sudah dicapai.

Hasil dari pembacaan tersebut disimpan dalam sebuah vector bernama `http_request`, yang 
kemudian ditampilkan ke console menggunakan `println!`. Dari output yang di dapat, 
terlihat bahwa browser mengirim berbagai informasi tambahan seperti jenis browser, encoding, 
dan preferensi bahasa.

### Commit 2 Reflection Notes
#### Inside handle_connection method after modification

Pada milestone ini, saya mempelajari bagaimana server dapat memberikan response berupa halaman HTML
kepada browser. Pada fungsi `handle_connection`, server tidak hanya membaca request, tetapi juga
mulai membangun response HTTP yang lengkap.

`fs::read_to_string` berfungsi untuk membaca isi file HTML (hello.html), lalu panjang konten akan
dihitung untuk digunakan dalam header `Content-Length`. Header ini diperlukan untuk memberi tahu
browser seberapa besar data yang akan diterima, sehingga halaman dapat dirender dengan benar.

Response HTTP dibuat dengan mengikuti struktur tertentu, dimulai dengan status line
(misalnya "HTTP/1.1 200 OK"), kemudian header, lalu baris kosong, dan terakhir isi body (HTML).
Semua bagian tersebut digabungkan menggunakan `format!` sebelum dikirim melalui `stream.write_all`.

![Commit 2 Screenshot](img/commit2.png)