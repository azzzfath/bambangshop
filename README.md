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

#### Reflection Publisher-3
