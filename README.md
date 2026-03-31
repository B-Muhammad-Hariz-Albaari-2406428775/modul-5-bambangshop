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
**1. Do We Need an Interface/Trait, or is a Single Model Struct Enough?**

Ya, Meskipun hanya ada satu Subscriber model sekarang, trait penting karena:
Decouple contract dari implementasi: Trait mendefinisikan kontrak observer terpisah dari implementasi Subscriber
Mengikuti SOLID principles: Repository harus depend pada interface, bukan concrete type
Future extensibility: Jika nanti ada EmailSubscriber, SMSSubscriber, dll., trait sudah siap
Sesuai Observer pattern: Head First Design Patterns menggunakan interface justru karena value-nya adalah multiple implementations

Tanpa trait, menyesuaikan untuk multiple observer types memerlukan refactoring besar.

**2. Is Vec Sufficient, or is DashMap Necessary?**

DashMap is necessary,
Kompleksitas lookup: Vec = O(n), DashMap = O(1). Dengan 10,000 subscribers, Vec butuh ~5,000 komparasi vs DashMap = 1
Kompleksitas deletion: Vec = O(n) (harus shift elements), DashMap = O(1)
Uniqueness by design: DashMap otomatis enforce url unique via key, Vec harus manual check
Scalability: Jutaan subscribers akan menjadi bottleneck di Vec

url adalah unique identifier, membutuhkan key-based data structure (DashMap), bukan sequential (Vec).

**3. Do We Need DashMap, or Can Singleton Pattern Alone Be Sufficient?**

Need both. They solve different problems.

Jika hanya menggunakan Singleton + Mutex<HashMap>:

Seluruh HashMap akan dikunci setiap kali ada operasi.
Misalnya Thread A sedang menambah data kategori "electronics".
Thread B tidak bisa mengakses kategori lain seperti "fashion" sampai lock dilepas.
Akibatnya, banyak request bersamaan bisa membuat program menjadi lambat karena semua harus menunggu satu sama lain.

DashMap bekerja berbeda:

DashMap membagi data ke beberapa bagian (bucket).
Setiap bagian memiliki lock sendiri.
Jadi beberapa thread bisa mengakses key yang berbeda secara bersamaan tanpa saling menunggu.

#### Reflection Publisher-2
**1. Mengapa Memisahkan "Service" dan "Repository" dari Model?**

Pemisahan ini mengikuti Single Responsibility Principle (SRP). 
Repository hanya handle data access, Service handle business logic, Model jadi pure data structure. 
Ini membuat code lebih testable (test logic tanpa mock database), 
maintainable (change database engine hanya affect Repository), 
dan reusable (satu Service bisa pakai multiple Repository).

**2. Apa yang Terjadi Jika Hanya Menggunakan Model?**

Kompleksitas melonjak drastis karena tight coupling. Program Model harus tahu cara subscribe/unsubscribe, 
Subscriber tahu validate & query, Notification tahu track status—semua mixed di satu tempat.
Akibatnya sulit test (butuh setup semua models), sulit maintain (perubahan di satu model 
affect yang lain), sulit reuse logic. Dengan Service-Repository separation, orchestration jadi
explicit dan centralized di service layer.

**3. Eksplorasi Postman**

Postman sangat membantu test API tanpa perlu frontend. Features yang berguna:

Environment variables: reuse base URL di semua requests
Tests tab: validate response otomatis (check status code, field existence)
Collections: organize & share endpoints dengan team
Mock server: frontend bisa develop tanpa tunggu backend ready
Newman CLI: load testing untuk simulate concurrent users

#### Reflection Publisher-3

**1. Observer Pattern Variation yang Digunakan**

Dalam tutorial ini, kita menggunakan Push model. 
Publisher (NotificationService) secara aktif mendorong data (Notification payload) kepada
subscribers melalui method subscriber.update(payload). 
Setiap kali ada event (product created atau published), 
NotificationService langsung mengirim notification object lengkap ke semua subscribers tanpa subscribers meminta data terlebih dahulu.

**2. Keuntungan dan Kerugian Pull Model**

Keuntungan:
- Subscribers bisa memutuskan kapan dan data apa yang perlu di-pull dari publisher
- Mengurangi beban network jika subscribers tidak perlu semua updates (selective pulling)
- Decoupling lebih kuat, publisher tidak perlu maintain list of subscribers, cukup expose data via endpoint
- Effisien untuk subscribers yang jarang membutuhkan updates

Kerugian:
- Subscribers harus implement polling logic, yang introduce complexity di client side
- Latency meningkat, ada delay antara event terjadi dan subscriber menyadarinya (tergantung polling interval)
- Network overhead malah bisa lebih besar jika banyak subscribers polling frequently tapi tidak ada data baru
- Subscribers harus tahu kapan harus pull, memerlukan external trigger atau timer logic

Untuk case ini, Push model lebih cocok karena notifikasi produk adalah time-sensitive events yang butuh immediate delivery ke semua subscribers. Pull model akan membuat subscribers tertinggal berita penting.

3. Dampak Tanpa Multi-threading

- Blocking behavior: Ketika ProductService::publish(id) dipanggil, thread utama akan menunggu semua subscribers menerima notification sebelum response dikirim ke client. Jika ada 1000 subscribers dan setiap HTTP request ke subscriber butuh 1 detik, total waiting time bisa 1000 detik.
- Poor user experience: Shop owner yang publish product akan mengalami hang/delayed response. API endpoint akan terasa sangat slow.
- Bottleneck di network I/O: Karena HTTP requests ke subscribers sequential, jika satu subscriber lambat atau timeout, semua subscriber di belakangnya jadi terpengaruh (cascade delay).
-  Server resource under-utilization: Thread hanya bisa handle satu subscriber request pada waktu tertentu, sehingga CPU idle sambil menunggu I/O response.
- Scalability issue: Semakin banyak subscribers, semakin lama operasi publish. Tidak scalable untuk production dengan ribuan subscribers.