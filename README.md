# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [X] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [X] Commit: `Create Subscriber model struct.`
    -   [X] Commit: `Create Notification model struct.`
    -   [X] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [X] Commit: `Implement add function in Subscriber repository.`
    -   [X] Commit: `Implement list_all function in Subscriber repository.`
    -   [X] Commit: `Implement delete function in Subscriber repository.`
    -   [X] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [X] Commit: `Create Notification service struct skeleton.`
    -   [X] Commit: `Implement subscribe function in Notification service.`
    -   [X] Commit: `Implement subscribe function in Notification controller.`
    -   [X] Commit: `Implement unsubscribe function in Notification service.`
    -   [X] Commit: `Implement unsubscribe function in Notification controller.`
    -   [X] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [X] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [X] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [X] Commit: `Implement publish function in Program service and Program controller.`
    -   [X] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [X] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. **Apakah kita masih butuh interface (atau trait di Rust) dalam kasus BambangShop ini?**

   Dalam Observer pattern seperti yang dijelaskan oleh buku *Head First Design Patterns*, `Subscriber` biasanya didefinisikan sebagai interface agar fleksibel dan bisa mengakomodasi berbagai jenis subscriber dengan implementasi berbeda. Tapi dalam kasus BambangShop, kita hanya punya satu jenis subscriber dengan cara kerja yang seragam: menerima notifikasi via HTTP POST. Karena semua subscriber diperlakukan sama, menggunakan satu `struct` `Subscriber` sudah cukup. Tidak ada kebutuhan untuk behavior polymorphic di sini, sehingga penggunaan `trait` belum dibutuhkan saat ini.

2. **Apakah `Vec` cukup atau perlu `DashMap` seperti sekarang?**

   Karena `url` pada `Subscriber` bersifat unik untuk setiap tipe produk, maka menyimpan subscriber dalam `Vec` akan menyulitkan operasi seperti pencarian atau penghapusan. Kita harus traverse seluruh list untuk cari berdasarkan URL. Sebaliknya, `DashMap` memberikan performa lebih baik karena mendukung akses langsung ke elemen berdasarkan key (dalam hal ini URL), dan juga sudah thread-safe. Jadi untuk efisiensi dan kemudahan implementasi, `DashMap` adalah pilihan yang lebih tepat.

3. **Apakah masih perlu `DashMap` kalau kita pakai Singleton pattern saja?**

   Singleton pattern bisa digunakan untuk membuat instance global, tapi tidak menyelesaikan masalah concurrency. Rust punya sistem ownership yang ketat untuk menjaga *thread safety*, dan ketika kita ingin mutable access dari banyak thread, kita perlu *thread-safe data structure*. Dalam kasus ini, `DashMap` menyediakan map yang aman untuk digunakan secara bersamaan oleh banyak thread. Jadi walaupun kita pakai Singleton, kita tetap perlu data structure seperti `DashMap` di dalamnya agar aman dari race condition.


#### Reflection Publisher-2

1. **Kenapa kita perlu pisahkan “Service” dan “Repository” dari Model?**

   Memisahkan `Service` dan `Repository` dari `Model` adalah praktik *separation of concerns* yang sangat penting. Kalau `Model` berisi sekaligus data, logic bisnis, dan akses data (ke “database”), maka tanggung jawabnya jadi terlalu besar (*God Object*). Dengan membagi:
   - `Repository` hanya fokus ke akses data (get, add, delete),
   - `Service` mengatur logika bisnis (validasi, transformasi data, atau memicu notifikasi),
   kita jadi punya kode yang lebih modular, mudah diuji, dan gampang dirawat. Misalnya saat logika notifikasi berubah, kita cukup ubah `Service`, tanpa ganggu `Model` atau `Repository`.

2. **Apa yang terjadi kalau kita hanya gunakan Model?**

   Kalau semua tanggung jawab ditaruh di `Model`, maka `Model` akan punya terlalu banyak logika dan dependency ke banyak bagian lain. Contohnya:
   - `Model` `Product` bisa tiba-tiba punya kode untuk menambahkan subscriber, atau kirim notifikasi.
   - `Subscriber` bisa punya logic untuk filtering produk.
   
   Ini bikin kode susah dibaca dan tinggi coupling antar model. Kalau satu model berubah, model lain bisa ikut rusak. Ujung-ujungnya, scalability turun dan debugging jadi mimpi buruk.

3. **Eksplorasi dan pengalaman dengan Postman**

   Ya, saya sudah eksplor lebih dalam tentang Postman, dan alat ini sangat membantu untuk uji coba API. Fitur-fitur yang saya suka dan berguna:
   - **Collection & Environment**: saya bisa simpan endpoint, body, dan variabel base URL untuk dipakai berulang.
   - **History**: sangat membantu kalau mau kirim ulang request yang sebelumnya.
   - **Visual feedback**: mempermudah ngecek respons API secara langsung (tanpa harus curl di terminal).
   - **Pre-request & Test scripts**: ini powerful banget kalau nanti mau otomatisasi pengujian di proyek kelompok.

   Dengan Postman, saya bisa fokus ngembangin dan ngetes fitur tanpa harus bikin front-end-nya dulu.


#### Reflection Publisher-3

1. **Observer Pattern yang digunakan dalam tutorial ini adalah Push Model.**  
   Publisher secara aktif mengirimkan data (notifikasi) ke semua subscriber ketika event tertentu terjadi, seperti saat produk dibuat, dihapus, atau dipromosikan. Subscriber hanya perlu menyediakan endpoint `/receive` dan tidak perlu melakukan polling data secara aktif.

2. **Keuntungan dan kerugian jika menggunakan Pull Model:**

   - *Keuntungan Pull Model:*  
     - Subscriber punya kontrol lebih besar: bisa pilih kapan dan seberapa sering ngecek update.
     - Publisher lebih ringan bebannya karena tidak perlu memikirkan semua subscriber secara aktif.

   - *Kerugian Pull Model (dalam konteks tutorial ini):*  
     - Subscriber harus melakukan polling berkala, yang bisa membebani jaringan.
     - Respon terhadap event jadi tidak real-time, ada delay tergantung interval polling.
     - Logika untuk nyimpan state perubahan jadi lebih rumit.

   Jadi, Push model lebih cocok buat tutorial ini karena lebih langsung dan real-time.

3. **Jika tidak menggunakan multi-threading dalam proses notifikasi:**  
   Notifikasi akan dikirim satu per satu secara blocking. Artinya, kalau ada banyak subscriber, publisher harus menunggu setiap request selesai dulu baru lanjut ke subscriber berikutnya. Hal ini bisa bikin aplikasi lambat atau bahkan stuck kalau salah satu subscriber lama merespons atau error. Dengan multi-threading, kita bisa kirim notifikasi ke semua subscriber secara paralel tanpa menghambat proses utama aplikasi.
