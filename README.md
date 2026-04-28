# Module 6 - Concurrency

## Commit 1 Reflection notes

Pada tahap ini saya mempelajari cara membuat web server sederhana menggunakan Rust standard library. Saya menggunakan `TcpListener` untuk membuat server mendengarkan koneksi TCP pada alamat `127.0.0.1:7878`. Ketika browser mengakses alamat tersebut, server menerima koneksi melalui iterator `listener.incoming()`. Setiap koneksi yang berhasil diterima direpresentasikan sebagai stream, meskipun pada tahap ini stream tersebut belum digunakan untuk membaca request atau mengirim response. Pesan `Connection established!` yang muncul di terminal menunjukkan bahwa server sudah berhasil menerima koneksi dari browser. Saya juga memahami bahwa browser dapat membuat lebih dari satu koneksi, sehingga pesan tersebut bisa muncul beberapa kali. Pada tahap ini server masih single-threaded, sehingga setiap request masih diproses satu per satu.

Pada tahap ini saya mempelajari bagaimana sebuah web server sederhana menerima koneksi dari browser menggunakan `TcpListener`. Setiap koneksi yang masuk direpresentasikan sebagai `TcpStream`, lalu stream tersebut dikirim ke fungsi `handle_connection` untuk diproses lebih lanjut. Saya juga mempelajari penggunaan `BufReader` untuk membaca isi request dari browser secara efisien. Dengan method `.lines()`, server dapat membaca HTTP request baris demi baris. Bagian `.take_while(|line| !line.is_empty())` digunakan untuk membaca header request sampai menemukan baris kosong, karena dalam format HTTP baris kosong menandakan akhir dari header. Dari output terminal, saya bisa melihat bahwa browser mengirim request seperti `GET / HTTP/1.1` beserta beberapa header seperti `Host`, `User-Agent`, dan `Accept`. Tahap ini membantu saya memahami bahwa komunikasi antara browser dan server dilakukan melalui teks HTTP request yang dikirim melalui koneksi TCP.

## Commit 2 Reflection notes

Pada tahap ini saya mempelajari cara membuat server mengirim response HTML ke browser. Sebelumnya server hanya membaca request dari browser dan mencetaknya ke terminal, tetapi belum memberikan response apa pun. Dengan menggunakan `fs::read_to_string("hello.html")`, server membaca isi file HTML sebagai string. Setelah itu server membuat HTTP response yang terdiri dari status line, header `Content-Length`, baris kosong, dan body HTML. Header `Content-Length` penting karena memberi tahu browser ukuran body response yang dikirim oleh server. Method `stream.write_all(response.as_bytes())` digunakan untuk mengirim response tersebut melalui koneksi TCP. Setelah tahap ini, browser akhirnya dapat merender halaman HTML yang dikirim oleh server Rust.

![Commit 2 screen capture](assets/images/commit2.png)

## Commit 3 Reflection notes

Pada tahap ini saya mempelajari cara membuat server memberikan response yang berbeda berdasarkan request path dari browser. Server membaca baris pertama HTTP request untuk mengetahui halaman apa yang sedang diminta oleh client. Jika request line bernilai `GET / HTTP/1.1`, server akan mengembalikan `hello.html` dengan status `200 OK`. Jika request line tidak sesuai dengan route yang dikenali, server akan mengembalikan `404.html` dengan status `404 NOT FOUND`. Saya juga memahami bahwa status line penting karena memberi informasi kepada browser apakah request berhasil atau gagal. Refactoring menggunakan `match` membuat kode lebih mudah dibaca dan lebih mudah dikembangkan ketika ingin menambahkan route baru. Dengan perubahan ini, server mulai memiliki perilaku yang lebih mirip web server sebenarnya karena dapat membedakan request valid dan request yang tidak ditemukan.

![Commit 3 screen capture](assets/images/commit3.png)

## Commit 4 Reflection notes

Pada tahap ini saya mempelajari bagaimana server single-threaded menangani request yang lambat. Saya menambahkan route `/sleep` yang menjalankan `thread::sleep(Duration::from_secs(10))` sebelum mengirim response. Ketika route tersebut diakses, server berhenti sementara selama 10 detik untuk memproses request itu. Karena server masih berjalan secara single-threaded, request lain tidak dapat diproses sampai request `/sleep` selesai. Hal ini terlihat ketika saya membuka `/sleep` lalu membuka `/` di tab lain, halaman `/` juga harus menunggu. Dari percobaan ini saya memahami bahwa single-threaded server kurang cocok untuk menangani banyak pengguna secara bersamaan. Masalah ini menjadi alasan mengapa pada tahap berikutnya server perlu dibuat multithreaded menggunakan thread pool.

![Commit 4 screen capture](assets/images/commit4.png)

## Commit 5 Reflection notes

Pada tahap ini saya mempelajari cara mengubah server single-threaded menjadi multithreaded menggunakan ThreadPool. Sebelumnya, setiap request diproses langsung oleh main thread sehingga request lain harus menunggu jika ada request yang lambat seperti `/sleep`. Dengan ThreadPool, server membuat beberapa worker thread yang siap menerima pekerjaan dari main thread. Setiap koneksi yang masuk dibungkus sebagai job, lalu dikirim melalui channel kepada worker yang tersedia. Penggunaan `Arc<Mutex<Receiver<Job>>>` memungkinkan beberapa worker berbagi receiver yang sama dengan aman. `Arc` digunakan agar receiver dapat dimiliki oleh beberapa thread, sedangkan `Mutex` memastikan hanya satu worker yang mengambil job pada satu waktu. Setelah perubahan ini, request `/` tetap bisa diproses meskipun request `/sleep` sedang berjalan, karena keduanya dapat ditangani oleh thread yang berbeda. Dari tahap ini saya memahami bahwa concurrency membantu server tetap responsif ketika menghadapi request yang lambat atau banyak request secara bersamaan.