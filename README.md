# advprog-module6
###### Deanita Sekar Kinasih
###### 2306229405

## 1. Commit 1 (Handle-connection, check response)
What is inside the handle_connection method?

Method `handle_connection` berfungsi untuk memproses koneksi TCP dengan cara:

- Menerima parameter `stream` bertipe `TcpStream`
- Menggunakan `BufReader` untuk membungkus stream dan membaca data secara efisien baris per baris. `BufReader` membantu buffering data sehingga bisa dibaca per baris
- Membaca seluruh request HTTP dengan menggunakan method `lines()` untuk mendapatkan iterator baris-baris, `map(|result| result.unwrap())` untuk mengambil String dari Result, `take_while(|line| !line.is_empty())` untuk membaca baris per baris hingga baris kosong (baris kosong menandakan akhir dari header HTTP), dan `collect()` untuk mengumpulkan semua baris ke dalam `Vec<String>` bernama `http_request`
- Mencetak request HTTP yang diterima ke konsol dengan format debug menggunakan `println!("Request: {:#?}", http_request)` untuk debugging

Secara keseluruhan, method ini memiliki tanggung jawab untuk membaca setiap baris dari koneksi TCP hingga menemukan baris kosong dan mencetak request HTTP ke konsol