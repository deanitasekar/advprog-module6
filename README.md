# advprog-module6
###### Deanita Sekar Kinasih
###### 2306229405

## 1. Commit 1 (Handle-connection, check response)
What is inside the handle_connection method?

Method `handle_connection` berfungsi untuk memproses koneksi TCP dengan cara:

- Menerima parameter `stream` bertipe `TcpStream`
- Menggunakan `BufReader` untuk membungkus stream dan membaca data secara efisien baris per baris. `BufReader` membantu buffering data sehingga bisa dibaca per baris
- Membaca seluruh request HTTP dengan menggunakan method `lines()` untuk mendapatkan iterator baris-baris, `map(|result| result.unwrap())` untuk mengambil string dari result, `take_while(|line| !line.is_empty())` untuk membaca baris per baris hingga baris kosong (baris kosong menandakan akhir dari header HTTP), dan `collect()` untuk mengumpulkan semua baris ke dalam `Vec<String>` bernama `http_request`
- Mencetak request HTTP yang diterima ke konsol dengan format debug menggunakan `println!("Request: {:#?}", http_request)` untuk debugging

Secara keseluruhan, method ini memiliki tanggung jawab untuk membaca setiap baris dari koneksi TCP hingga menemukan baris kosong dan mencetak request HTTP ke konsol.

## 2. Commit 2 (Returning HTML)
![Commit 2 screen capture](/assets/images/commit2.png)

What you have learned about the new code the handle_connection?

```rust
let status_line = "HTTP/1.1 200 OK"; 
    let contents = fs::read_to_string("hello.html").unwrap(); 
    let length = contents.len(); 
  
    let response = 
        format!("{status_line}\r\nContent-Length: 
 {length}\r\n\r\n{contents}"); 
  
    stream.write_all(response.as_bytes()).unwrap();
```

Melalui pembaruan kode pada fungsi `handle_connection`, fungsi ini tidak hanya membaca request HTTP, tetapi juga mengirimkan respon balik. Server membuat status line HTTP dengan kode `HTTP/1.1 200 OK` yang menandakan bahwa request berhasil diproses, lalu membaca `hello.html` menggunakan fungsi `fs::read_to_string()` dan menggunakan `unwrap()` untuk mengambil string dengan mengabaikan kemungkinan error. Kemudian, server menghitung panjang konten HTML dengan `contents.len()` untuk dimasukkan ke dalam header HTTP. Server menyusun respons lengkap dalam format HTTP dengan menggabungkan status line, header Content-Length, baris kosong pemisah, dan isi HTML menggunakan `format!()`. Respon ini dikonversi menjadi bytes dengan `as_bytes()` dan dikirimkan kembali melalui koneksi TCP yang sama menggunakan kode `stream.write_all()`.

## 3. Commit 3 (Validating request and selectively responding)
![Commit 3 screen capture](/assets/images/commit3.png)

How to split between response and why the refactoring is needed?

Untuk memisahkan respons pada server HTTP, digunakan pengecekan `request_line` untuk menentukan jenis respons yang tepat. Server akan memeriksa apakah nilai `request_line` sama dengan `GET/HTTP/1.1`. Jika kondisi ini terpenuhi, server akan mengirimkan respons berisi file `hello.html` dengan status `200 OK`. Namun, apabila kondisi tidak terpenuhi, server akan mengirimkan respons yang berisi file `404.html` dengan status `494 NOT FOUND`. Pendekatan ini memungkinkan server memberikan respons yang sesuai berdasarkan jenis request yang diterima.

Refactoring diperlukan karena terdapat duplikasi kode dalam implementasi awal. Blok `if-else` hanya berbeda pada dua baris kode (`status_line` dan `contents`). sementara bagian lainnya identik. Refactoring ini dapat menghilangkan redundasi sehingga meningkatkan keterbacaan kode, memudahkan pemeliharaan, dan membuat kode lebih mudah dikembangkan.

## 4. Commit 4 (Simulation of slow request)
![Commit 4 screen capture](/assets/images/commit4.png)

Why it works like that?

Simulasi yang dilakukan menunjukkan kelemahan dari server single-thread dalam menangani banyak request secara bersamaan. Ketika menambahkan rute `/sleep` yang menunda eksekusi selama 10 detik secara sengaja dengan `thread::sleep(Duration::from_secs(10))`, terlihat bagaimana satu request yang lambat dapat memblokir seluruh server.

Dalam simulasi ini, saat membuka dua jendela browser (satu untuk mengakses `/sleep` dan satu lagi untuk mengakses halaman utama `/`), kedua request tidak dapat diproses secara bersamaan. Server harus menyelesaikan request pertama sebelum memproses request kedua. Hal ini terjadi karena server hanya memiliki satu thread, sehingga hanya bisa mengerjakan satu tugas pada satu waktu. Untuk mengatasi ini, dibutuhkan pendekatan yang dapat menangani banyak request secara paralel, seperti multi-threading atau asynchronous processing.