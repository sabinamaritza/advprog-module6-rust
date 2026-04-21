# Reflection

### Commit 1 Reflection Notes
#### Method handle_connection

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