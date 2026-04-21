# Reflection

### Commit 1 Reflection Notes
#### Inside handle_connection method

Fungsi `handle_connection` menerima parameter `TcpStream` yang merepresentasikan 
koneksi antara server dan client. Di dalam fungsi tersebut, `BufReader` berfungsi untuk membaca data
HTTP request dari stream secara efisien. HTTP request yang dikirim oleh browser terdiri dari 
beberapa baris seperti request line (misalnya "GET / HTTP/1.1") dan header seperti Host, 
User-Agent, dan lainnya.

Metode `.lines()` digunakan untuk membaca setiap baris dari request, lalu 
`.map(|result| result.unwrap())` digunakan untuk mengambil nilai string dari hasil pembacaan. 
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

### Commit 3 Reflection Notes
#### How to split between responses and why the refactoring is needed

Ketika bagian error reponse pertama ditambahkan, kode langsung menggunakan `if-else` untuk splitting
proses pembuatan response, sehingga terdapat duplikasi kode pada bagian pembacaan file, 
perhitungan panjang konten, dan pengiriman response.

Karena code duplication merupakan code smell, perlu dilakukan refactoring dengan cara memisah
bagian yang berbeda dan bagian yang sama. Bagian yang berbeda hanya `status_line` dan 
`filename`, yang ditentukan berdasarkan request, seperti "GET / HTTP/1.1". Lalu, bagian yang
sama, seperti membaca file, menghitung panjang konten, membuat response string, dan pengiriman 
ke stream hanya perlu ditulis satu kali saja di luar kondisi `if-else`. Pada kode yang telah di 
refactor, tuple `(status_line, filename)` digunakan untuk menyimpan hasil dari percabangan kondisi.

Refactoring ini diperlukan karena meningkatkan readability dan maintainability kode. Sehingga jika 
nantinya ingin menambahkan lebih banyak route atau jenis response, hanya perlu menambahkan
kondisi baru tanpa menyalin ulang seluruh proses pembuatan response. Selain itu, kode menjadi 
lebih clean dan lebih mudah dipahami.

![Commit 3 Screenshot](img/commit3.png)

### Commit 4 Reflection Notes
#### Single thread causing slow request

Pada milestone ini, dilakukan simulasi request yang lambat pada server dengan menggunakan 
`thread::sleep`. Pada endpoint `/sleep`, server akan menunda response selama 10 detik 
sebelum mengirimkan hasil ke browser.

Dalam single-threaded server, server hanya dapat menangani satu request dalam satu waktu. 
Sehingga, ketika ada request yang membutuhkan waktu yang lama untuk diproses, request lain 
yang masuk harus menunggu hingga proses yang lama tersebut selesai.

Saat satu tab mengakses endpoint `/sleep`, tab lain yang mengakses endpoint biasa (`/`) juga ikut 
tertunda. Ini menunjukkan bahwa server tidak dapat memproses request secara paralel.

Kondisi ini menjadi masalah besar ketika server digunakan oleh banyak pengguna, karena satu 
request yang lambat dapat menghambat sistem secara keseluruhan. Oleh karena itu, pendekatan
single-threaded tidak efisien untuk aplikasi yang membutuhkan performa tinggi.

### Commit 5 Reflection Notes
#### How ThreadPool works

Pada milestone ini, dilakukan implementasi multithreaded server menggunakan ThreadPool untuk 
mengatasi keterbatasan server single-threaded. ThreadPool pada tutorial ini terdiri dari beberapa 
worker thread yang disimpan dalam vector, serta sebuah channel (`mpsc`) yang digunakan untuk 
mengirim job dari main thread ke worker.

Ketika server menerima koneksi baru, koneksi tersebut tidak langsung diproses, tetapi dibungkus 
menjadi sebuah closure dan dikirim sebagai job melalui `sender` ke dalam channel menggunakan 
fungsi `execute`. Job ini kemudian akan diambil oleh salah satu worker thread yang sedang idle.

Setiap worker berjalan dalam loop dan terus menunggu job menggunakan `receiver.lock().unwrap().recv()`.
Untuk memastikan keamanan dalam akses data bersama, digunakan `Arc` dan `Mutex`. `Arc` memungkinkan 
receiver untuk dimiliki oleh banyak thread, sedangkan `Mutex` memastikan hanya satu thread yang dapat
mengambil job dari channel dalam satu waktu.

Pada implementasi ini, lock hanya digunakan saat mengambil job, kemudian dilepas sebelum job 
dieksekusi. Hal ini memungkinkan worker lain tetap dapat mengambil job secara paralel tanpa saling 
menghambat.

Dengan menggunakan ThreadPool, server dapat menangani beberapa request secara bersamaan. Hal ini
terlihat ketika endpoint `/sleep` tidak lagi memblokir request lain, karena request tersebut 
diproses oleh thread yang berbeda. Sehingga, performa dan responsivitas server meningkat 
secara signifikan dibandingkan dengan pendekatan single-threaded.

### Bonus Reflection Notes

Pada bagian bonus, terdapat perubahan fungsi `new` dengan fungsi `build` untuk membuat ThreadPool 
dengan pendekatan yang lebih aman. Perbedaan utama antara keduanya adalah pada cara menangani error.

Pada fungsi `new`, digunakan `assert!` untuk memastikan ukuran thread pool lebih dari nol. 
Jika kondisi ini tidak terpenuhi, program akan langsung berhenti. Pendekatan ini kurang fleksibel 
karena tidak memberikan kesempatan bagi program untuk menangani error dengan lebih baik.

Sebagai alternatif, fungsi `build` mengembalikan tipe `Result<ThreadPool, String>`, sehingga 
jika terjadi kesalahan, seperti ukuran thread pool bernilai nol, fungsi akan mengembalikan `Err` 
daripada menghentikan program. Hal ini memungkinkan pemanggil fungsi untuk menangani error 
tersebut dengan lebih aman, misalnya dengan menampilkan pesan error atau mengambil tindakan lain.

Penggunaan `Result` juga merupakan praktik yang umum dalam Rust untuk meningkatkan keandalan 
program. Dengan pendekatan ini, program menjadi lebih robust dan tidak mudah crash dikarenakan 
kesalahan yang seharusnya bisa ditangani.
