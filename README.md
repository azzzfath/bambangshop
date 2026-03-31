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
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

### 1. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or `trait` in Rust) in this BambangShop case, or a single Model `struct` is enough?

- Dalam konteks desain pola Observer yang ideal, menggunakan `trait` (sebagai *interface* di Rust) jauh lebih baik daripada hanya menggunakan sebuah `struct` Model tunggal. Menggunakan `trait` memungkinkan kita untuk mematuhi prinsip *Open-Closed Principle* (OCP). Jika ke depannya BambangShop ingin menambahkan tipe *subscriber* yang berbeda (misalnya: `EmailSubscriber`, `PushNotificationSubscriber`, dll), Publisher tidak perlu dimodifikasi sama sekali; kita hanya perlu membuat `struct` baru yang mengimplementasikan `trait` tersebut.
- Namun, jika *scope* aplikasi BambangShop ini dijamin sangat kaku dan **hanya** akan pernah melayani satu jenis *subscriber* (yaitu webhook URL) selamanya, maka menggunakan satu `struct` Model saja secara teknis sudah cukup untuk membuat program berjalan, meskipun hal ini membuat kode menjadi kurang *scalable* dan *extensible* untuk pengembangan masa depan.

### 2. `id` in `Program` and `url` in `Subscriber` is intended to be unique. Explain based on your understanding, is using `Vec` (list) sufficient or using `DashMap` (map/dictionary) like we currently use is necessary for this case?

- Penggunaan `DashMap` (atau *map/dictionary* secara umum) sangat direkomendasikan dan bisa dibilang esensial untuk kasus ini dibandingkan menggunakan `Vec` (*list*). 
- Karena `id` dan `url` bersifat unik, operasi yang akan paling sering kita lakukan adalah mencari (untuk melihat apakah *subscriber* sudah ada), menghapus, atau memperbarui data berdasarkan nilai unik tersebut. 
- Jika menggunakan `Vec`, kita harus melakukan iterasi (looping) ke seluruh elemen satu per satu yang memakan waktu $O(N)$, yang akan sangat lambat ketika jumlah *subscriber* bertambah banyak. Dengan `DashMap`, kita menggunakan `id` atau `url` sebagai *key*, sehingga operasi pencarian, penambahan, dan penghapusan memiliki kompleksitas waktu mendekati $O(1)$. Selain itu, *map/dictionary* secara bawaan akan memastikan setiap *key* unik, sehingga mencegah duplikasi data secara otomatis.

### 3. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (`SUBSCRIBERS`) static variable, we used the `DashMap` external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need `DashMap` or we can implement Singleton pattern instead?

- Sebenarnya, *Singleton pattern* dan *Thread-safe data structures* (seperti `DashMap`) menyelesaikan dua masalah yang berbeda, dan di Rust (khususnya untuk *web server* yang *concurrent*), kita seringkali membutuhkan kombinasi keduanya. 
- Singleton adalah *creational pattern* yang memastikan hanya ada satu *instance* dari sebuah objek (global state). Penggunaan makro `lazy_static!` pada `SUBSCRIBERS` sebenarnya **sudah** mengimplementasikan bentuk dari pola Singleton.
- Namun, Singleton saja tidak cukup di Rust. Karena banyak *request* (dan karenanya, banyak *thread*) yang akan mencoba membaca dan memodifikasi Singleton tersebut secara bersamaan, *compiler* Rust akan mencegahnya untuk menghindari *data race*. 
- Kita membutuhkan mekanisme sinkronisasi untuk membuatnya *thread-safe*. `DashMap` menyediakan struktur data *map* yang *lock/synchronization*-nya sudah diatur secara internal. Jika kita tidak menggunakan `DashMap`, kita tetap mengimplementasikan Singleton, namun kita harus membungkus `HashMap` biasa dengan `Mutex` atau `RwLock` secara manual agar kompilasi berhasil dan program aman dari tabrakan memori antar thread.

#### Reflection Publisher-2

### 1. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?

- Memisahkan *Service* dan *Repository* dari *Model* adalah penerapan langsung dari prinsip *Single Responsibility Principle* (SRP) dan *Separation of Concerns* (SoC).
- Jika semua logika bisnis dan akses penyimpanan disatukan ke dalam *Model* (sering disebut *Fat Model*), kode akan menjadi sangat besar, kompleks, dan sulit dipelihara seiring berjalannya waktu.
- Dengan pemisahan ini, setiap lapisan punya tugas spesifik:
  - **Model:** Hanya berfokus mendefinisikan struktur data (representasi tabel/objek).
  - **Repository:** Hanya berfokus pada operasi akses data (*database/storage*), seperti *query*, *insert*, *update*, *delete*.
  - **Service:** Hanya berfokus pada logika bisnis dan aturan aplikasi, menghubungkan data dari *Repository* ke *Controller*.
- Pemisahan ini membuat kode jauh lebih mudah dibaca, lebih mudah digunakan kembali (*reusable*), dan sangat mempermudah proses *Unit Testing* karena kita bisa me-*mock* bagian *Repository* atau *Service* secara independen.

### 2. What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?

- Jika kita hanya menggunakan Model tanpa pemisahan lapisan, kode kita akan menjadi sangat *tightly coupled* (sangat bergantung satu sama lain) dan berpotensi menjadi *spaghetti code*.
- Bayangkan interaksi antara `Program`, `Subscriber`, dan `Notification`. Jika logika bisnis disatukan di Model, maka Model `Program` harus tahu cara meng-*query* `Subscriber` dari *database*, lalu harus tahu cara merakit objek `Notification` dan memicu pengirimannya.
- Kompleksitas setiap Model akan meningkat drastis. Perubahan kecil pada cara kita menyimpan `Subscriber` bisa memaksa kita untuk merombak kode di dalam Model `Program`. Selain itu, hal ini sangat berisiko memunculkan *circular dependency* (misalnya `Program` meng-*import* `Subscriber`, dan `Subscriber` meng-*import* `Program`) yang akan membuat *compiler* Rust *error*.

### 3. Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.

- Ya, Postman sangat membantu dalam menguji API (*endpoint* aplikasi) yang sedang dikembangkan tanpa perlu membuat tampilan *frontend/UI*-nya terlebih dahulu. Kita bisa dengan mudah melakukan simulasi berbagai macam HTTP request (GET, POST, DELETE, dll) lengkap dengan *Header* dan *Body*-nya untuk melihat apakah respons server (seperti status kode 200 OK atau 201 Created) sudah sesuai ekspektasi.
- Beberapa fitur Postman yang menurut saya sangat menarik dan berguna untuk *Group Project* atau proyek rekayasa perangkat lunak ke depannya adalah:
  - **Collections:** Memungkinkan kita mengelompokkan berbagai *request* API secara rapi dalam satu folder (*workspace*), sehingga mudah dibagikan ke anggota tim lain.
  - **Environment Variables:** Sangat berguna untuk menyimpan variabel yang sering berubah, seperti `base_url`. Jadi kita bisa dengan mudah berganti dari *testing* di `localhost:8000` ke URL *production* tanpa harus mengubah URL di setiap *request* satu per satu.
  - **Save Responses / Examples:** Kita bisa menyimpan contoh hasil *response* (JSON) dari API, yang nantinya bisa berfungsi sebagai dokumentasi otomatis agar tim *Frontend* tahu bentuk data yang akan mereka terima.

#### Reflection Publisher-3

### 1. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?

- Pada tutorial ini, kita menggunakan variasi **Push model**.
- Hal ini terlihat pada fungsi `NotificationService::notify` dan `Subscriber::update`. Setiap kali ada *event* baru (seperti pembuatan, promosi, atau penghapusan produk), *Publisher* (BambangShop) secara aktif membuat *payload* data (Notifikasi) dan langsung "mendorong" (mengirimkan) data tersebut ke URL masing-masing *Subscriber* melalui HTTP POST request. *Subscriber* pasif dan hanya menunggu data datang.

### 2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)

Jika kita menggunakan **Pull model** (di mana *Subscriber* yang harus me-*request* data produk terbaru dari *Publisher*):

- **Kelebihan:** - *Subscriber* memiliki kontrol penuh atas kapan mereka siap memproses data. Hal ini mencegah server *Subscriber* mengalami *overload* jika *Publisher* mengirimkan terlalu banyak notifikasi sekaligus dalam waktu singkat.
  - *Publisher* tidak perlu tahu detail URL *Subscriber* dan tidak perlu mengelola *HTTP client* untuk mengirim data keluar.
- **Kekurangan:** - **Tidak *Real-time*:** *Subscriber* harus melakukan pengecekan berulang-ulang (*polling*) ke server BambangShop (misal setiap 5 menit) untuk melihat apakah ada produk baru.
  - **Sangat tidak efisien:** Jika tidak ada produk baru seharian, ribuan *request polling* dari *Subscriber* hanya akan membuang-buang *resource* server (CPU & *Bandwidth*) BambangShop secara sia-sia.
  - *Publisher* harus menyimpan riwayat (*history*) semua notifikasi di *database* agar *Subscriber* bisa menariknya kapan saja.

### 3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.

- Jika kita tidak menggunakan *multi-threading* (misalnya kita menghapus bagian `thread::spawn` di dalam *looping* fungsi `notify`), maka proses pengiriman notifikasi akan berjalan secara *synchronous* atau *blocking*.
- Akibatnya, ketika seorang *user* (atau admin) menambahkan produk baru via API, *request* tersebut tidak akan langsung selesai. Server harus melakukan HTTP POST satu per satu ke **setiap** *Subscriber* dan menunggu prosesnya selesai.
- Jika ada 1.000 *subscriber*, atau jika ada salah satu server *subscriber* yang sedang *down* dan lambat merespons (*timeout*), maka API kita akan tertahan (*hang*). Pengguna yang menekan tombol "Create Product" akan merasakan *loading* yang sangat lama (lambat) hanya karena menunggu proses notifikasi di *background* selesai. Dengan *multi-threading*, server bisa langsung mengembalikan respons `201 Created` ke *user* dengan cepat, sementara notifikasi dikirimkan secara paralel di *background*.
