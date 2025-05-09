# Module 08
###### Deanita Sekar Kinasih
###### 2306229405


## 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

-   Unary RPC

    Unary RPC adalah metode komunikasi gRPC paling sederhana, dimana client mengirimkan satu request tunggal ke server dan menerima balasan satu response tunggal. Metode ini menciptakan pola komunikasi 1:1 yang cocok untuk interaksi cepat dan langsung. Karakteristik synchronous dari Unary RPC cocok digunakan ketika pengguna perlu menunggu response sebelum melanjutkan prosesnya. Unary RPC ideal untuk skenario proses autentikasi pengguna yang membutuhkan validasi kredensial dan memberikan status secara langsung. Selain itu, Unary RPC sangat sesuai untuk pengambilan data spesifik seperti menampilkan detail produk pada aplikasi e-commerce atau mengambil informasi profil pengguna. Notifikasi instan juga merupakan skenario penggunaan yang tepat, seperti mengirim email konfirmasi atau pemberitahuan.

-   Server Streaming RPC

    Server Streaming RPC menawarkan pola komunikasi 1:M dimana client mengirimkan satu request dan server menanggapi dengan mengirimkan serangkaian response secara bertahap dalam bentuk stream. Metode ini efisien untuk mentransmisikan data yang besar atau dinamis karena memungkinkan server mengirimkan informasi secara progresif tanpa harus menunggu seluruh proses pengumpulan data selesai terlebih dahulu. Server Streaming RPC bermanfaat untuk skenario yang memerlukan pembaruan berkala seperti layanan cuaca yang menampilkan kondisi terkini, aplikasi keuangan yang menyajikan perubahan harga saham secara real-time, atau notifikasi progres unduhan file. Metode ini juga unggul dalam pengiriman data berukuran besar seperti streaming video playlist atau transfer file yang dipecah menjadi beberapa bagian.

-   Bi-directional Streaming RPC

    Bi-directional Streaming RPC mewakili bentuk komunikasi paling dinamis dan interaktif dalam protokol gRPC dengan pola komunikasi M:N. Dalam metode ini, baik client maupun server dapat saling mengirimkan banyak pesan secara simultan tanpa urutan yang pasti, menciptakan komunikasi dua arah yang real-time. Fleksibilitas ini memungkinkan kedua pihak untuk berkomunikasi secara bersamaan, menjadikannya ideal untuk aplikasi yang membutuhkan interaksi responsif dan dinamis. Bi-directional Streaming RPC sangat cocok untuk aplikasi kolaborasi real-time seperti platform chat dengan pertukaran pesan secara terus-menerus atau sistem kolaborasi dokumen yang memungkinkan pengeditan bersama. Selain itu, metode ini sangat cocok untuk layanan Internet of Things (IoT) yang memerlukan komunikasi dua arah antara perangkat dan server pusat untuk pemantauan dan kontrol.


## 2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?

-   Authentication

    Tanpa authentication yang kuat, penyerang dapat dengan mudah terhubung ke server. Untuk mengatasi risiko ini, disarankan mengimplementasikan Token-Based Authentication menggunakan `JWT (JSON Web Token)` atau integrasi `OAuth2` untuk memverifikasi identitas secara akurat. Pada Rust, library seperti `jsonwebtoken` atau `tonic-jwt` dapat dimanfaatkan untuk validasi token di sisi server. Selain itu, Mutual TLS (mTLS) juga merupakan solusi yang direkomendasikan untuk verifikasi dua arah antara pengguna dan server. Dalam implementasinya, framework `tonic` dengan middleware dapat digunakan untuk mengekstrak dan memvalidasi token dari metadata gRPC dengan efisien.

-   Authorization
    
    Tanpa authorization yang tepat, pengguna dengan role rendah dapat mengakses fitur atau data yang seharusnya terbatas, misalnya fitur admin-only. Solusi yang direkomendasikan adalah menerapkan Role-Based Access Control (RBAC) dengan mendefinisikan role pengguna seperti user dan admin, serta membatasi akses ke endpoint atau resource berdasarkan role tersebut. Implementasi RBAC di Rust dapat memanfaatkan middleware untuk pengecekan hak akses menggunakan library seperti `tower` atau `tonic-intercept`. Dalam praktiknya, dapat dilakukan penambahan layer authorization di handler gRPC yang memeriksa metadata request untuk mengetahui role pengguna.

-   Data Encryption

    Tanpa data encryption yang memadai, data yang ditransmisikan berisiko dicuri, termasuk password, token, dan informasi sensitif lainnya. Solusi utama yang direkomendasikan adalah menerapkan Transport Layer Security (TLS) untuk mengenkripsi seluruh komunikasi antara pengguna dan server. Pada Rust, konfigurasi server gRPC dengan `tonic` dan `rustls` dapat dilakukan untuk mendukung koneksi HTTPS yang aman. Selain itu, data sensitif juga perlu dienkripsi saat disimpan (at rest) menggunakan algoritma seperti `AES-256` sebelum disimpan di database.

Dengan menerapkan langkah-langkah tersebut, gRPC dalam Rust dapat menjaga kerahasiaan, integritas, dan ketersediaan data, serta memastikan hanya pengguna sah yang dapat berinteraksi dengan sistem.


## 3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

Implementasi Bi-directional Streaming gRPC memiliki beberapa tantangan kompleks, terutama dalam skenario aplikasi real-time seperti platform chat. Tantangan utama meliputi manajemen konkurensi yang membutuhkan penanganan race condition, deadlock, dan skalabilitas ketika banyak pengguna mengirim dan menerima pesan secara bersamaan. Untuk mengatasi ini, penggunaan `async/await` dengan runtime seperti `Tokio` dan `MPSC channels` menjadi rekomendasi solusi untuk mengelola komunikasi asynchronous secara efisien.

Penanganan koneksi juga menjadi tantangan signifikan karena pengguna dapat terputus secara tiba-tiba akibat jaringan yang buruk. Implementasi mekanisme `heartbeat/ping-pong` dan pengaturan `timeout` yang tepat dapat membantu memantau status koneksi dan melakukan `graceful shutdown` saat diperlukan.

Error handling menjadi lebih kompleks karena kesalahan pada satu stream dapat memengaruhi stream lainnya apabila tidak diisolasi dengan baik. Penggunaan `Result` dan `operator ?` di Rust serta isolasi error dengan `spawn_blocking` dapat membantu pengelolaan propagasi error dengan efisien.

Tantangan lainnya meliputi message ordering yang dapat terganggu oleh paralelisasi, manajemen memori dan backpressure untuk mencegah buffer overflow, serta thread safety untuk menghindari data race pada shared state. Penggunaan `sequence number` atau `timestamp` untuk pesan, penerapan `backpressure` dengan membatasi ukuran channel, dan pemanfaatan `Arc<Mutex<T>>` atau `RwLock` untuk sinkronisasi data menjadi solusi yang efektif untuk permasalahan tersebut.


## 4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?

`tokio_stream::wrappers::ReceiverStream` menawarkan beberapa kelebihan signifikan dalam implementasi streaming dalam gRPC Rust. Kelebihan utamanya adalah integrasi yang mulus dengan ekosistem Tokio melalui compatibility tinggi dan pemanfaatan MPSC Channels yang mengubah `mpsc::Receiver` menjadi `Stream` untuk digunakan dengan framework seperti `tonic`. Wrapper ini mampu mengurangi kompleksitas kode dengan sinkronisasi asynchronous tanpa pengelolaan thread manual dan meningkatkan reusability code. Selain itu, ReceiverStream bersifat responsif dan ringan serta memiliki mekanisme backpressure alami untuk mencegah overload sistem. 

Namun, `tokio_stream::wrappers::ReceiverStream` memiliki beberapa kelemahan. Salah satu kelemahan utamanya adalah hanya mendukung komunikasi satu arah sehingga tidak ideal untuk bidirectional streaming tanpa modifikasi tambahan. Manajemen error juga harus dilakukan secara manual karena tidak ada mekanisme built-in untuk menangani kesalahan dalam stream. Dalam skala besar, ReceiverStream dapat mengalami overhead yang signifikan dan berpotensi menjadi bottleneck. Wrapper ini juga kurang efisien untuk transfer data berukuran besar yang memerlukan fragmentasi manual, serta membutuhkan kode adapter tambahan untuk integrasi dengan gRPC. 

Oleh karena itu, ReceiverStream sangat direkomendasikan untuk aplikasi dengan server/client streaming yang membutuhkan data real-time dengan throughput sedang dan mengutamakan kesederhanaan implementasi. Namun, sebaiknya dihindari untuk bidirectional Streaming gRPC, transfer data besar, atau aplikasi dengan mekanisme pemulihan error otomatis.


## 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

Untuk meningkatkan reusabilitas, modularitas, dan maintainability gRPC Rust dalam jangka panjang, terdapat beberapa strategi yang dapat diterapkan. Pemisahan logika bisnis dari layanan gRPC memungkinkan perubahan pada protokol tanpa memengaruhi logika bisnis, serta memudahkan pengujian logika bisnis tanpa ketergantungan pada framework gRPC. Struktur proyek yang direkomendasikan meliputi direktori terpisah untuk gRPC (definisi protobuf, service, dan handler) dan core (implementasi logika bisnis).

Penggunaan modul bersama untuk reuse code dapat mengurangi duplikasi dan meningkatkan konsistensi. Strategi ini dilengkapi dengan penerapan generics dan traits untuk membuat komponen yang dapat bekerja dengan berbagai tipe data sehingga kode dapat diadaptasi untuk entitas berbeda. Penggunaan design pattern juga direkomendasikan untuk konstruksi objek yang jelas dan terstruktur, meminimalkan kesalahan pada objek dengan banyak parameter. Struktur proyek sebaiknya diorganisasi berdasarkan tanggung jawab (models, services, repositories, utils, config, dan gRPC).

Abstraksi dengan trait untuk layanan memungkinkan penggantian implementasi dengan mudah, sementara pemanfaatan async/await dengan runtime Tokio mendukung concurrency dan scalability. Dependency injection meningkatkan modularitas dan kemudahan pengujian dengan menyuntikkan dependensi ke dalam layanan. Penerapan error handling menggunakan tipe error custom dan konversi error gRPC secara global untuk response error yang konsisten juga perlu diterapkan. Selain itu, dokumentasi yang baik dengan menggunakan `///` pada setiap modul/fungsi dan pengujian menyeluruh untuk memastikan kode yang mudah dipelihara dan diperluas dalam jangka panjang.


## 6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

Langkah tambahan dalam implementasi MyPaymentService untuk menangani proses payment yang lebih kompleks, yaitu:

-   Validasi data dan keamanan input

    Mencegah serangan seperti SQL Injection, XSS, atau data yang tidak valid. 

-   Authentication dan authorization

    Memastikan hanya pengguna terautentikasi dengan hak akses yang dapat melakukan transaksi.

-   Interaksi dengan database secara aman

    Memastikan konsistensi data dan mencegah race condition. 

-   Integrasi dengan Payment Gateway eksternal

    Menangani berbagai metode payment. 

-   Pemantauan status secara real-time

    Memberikan update status payment secara langsung. 

-   Notifikasi hasil payment

    Mengirimkan pemberitahuan ke user. 

-   Penanganan error dan logging

    Memudahkan debugging dan pemulihan sistem.


## 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

Adopsi gRPC sebagai protokol komunikasi memberikan dampak signifikan pada desain arsitektur sistem terdistribusi. Salah satu dampak utamanya adalah efisiensi komunikasi contract-first melalui Protobuf. Hal ini meningkatkan konsistensi dan meminimalkan kesalahan komunikasi antar layanan karena struktur data dan endpoint telah terstandardisasi. 

gRPC juga mendukung interoperabilitas multi-bahasa dan platform, memungkinkan layanan dalam Rust berkomunikasi dengan layanan Java, Python, atau Go tanpa perlu menulis parser kustom. Komunikasi dengan sistem non-gRPC (REST/SOAP) juga dimungkinkan melalui gateway seperti gRPC-Gateway. Performa tinggi dengan HTTP/2 menjadi keunggulan lain gRPC yang memungkinkan multiplexing dan streaming. HTTP/2 mendukung pengiriman banyak request dan response secara paralel dalam satu koneksi, menghasilkan latensi rendah yang ideal untuk aplikasi real-time dan efisiensi bandwidth melalui format biner Protobuf yang lebih kecil dibandingkan JSON/XML.

Secara keseluruhan, adopsi gRPC dapat membangun arsitektur yang terstandardisasi, efisien, dan scalable. Untuk sistem heterogen yang melibatkan banyak platform, kombinasi gRPC dengan API Gateway dan adapters dapat menjadi kunci keberhasilan implementasi.


## 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

HTTP/2 menawarkan beberapa keunggulan signifikan dibandingkan HTTP/1.1 atau kombinasi HTTP/1.1 dengan WebSocket. Keunggulan utamanya adalah multiplexing yang memungkinkan pengiriman banyak request dan response secara paralel dalam satu koneksi TCP, menyelesaikan permasalahan head-of-line blocking yang sering terjadi pada HTTP/1.1. HTTP/2 juga menawarkan kompresi header (HPACK) yang dapat mengurangi ukuran header hingga 90%, streaming bawaan yang mendukung unary, server/client streaming, dan bidirectional streaming, serta efisiensi kinerja berkat Protobuf dan koneksi persisten yang mengurangi overhead. 

Meskipun demikian, HTTP/2 memiliki kelemahan, seperti compatibility yang terbatas dengan server, CDN, atau perangkat lama. Selain itu, terdapat kompleksitas debugging karena Protobuf lebih sulit dibaca dibandingkan JSON, ketergantungan pada TLS yang menambah kompleksitas, serta overhead resource.

Dibandingkan dengan HTTP/1.1 dengan WebSocket, HTTP/2 unggul dalam komunikasi real-time dengan bidirectional streaming, multiplexing tanpa batasan, serta kompresi header dan payload. Sementara itu, HTTP/1.1 dengan WebSocket memiliki keunggulan dalam hal compatibility yang lebih universal dan kemudahan debugging. Dapat disimpulkan bahwa HTTP/2 lebih cocok untuk sistem terdistribusi kompleks yang membutuhkan kinerja, efisiensi, dan kemampuan real-time tinggi seperti microservices architecture, aplikasi yang membutuhkan throughput tinggi dan latency rendah, serta infrastruktur modern.


## 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

REST API dan gRPC memiliki perbedaan mendasar dalam pola komunikasi dan kemampuan real-time. REST API menggunakan model request-response 1:1 yang bersifat stateless dengan request independen, sedangkan gRPC memanfaatkan streaming M:N yang bersifat stateful dengan koneksi persisten. Dalam REST API, koneksi baru dibuat untuk setiap request, sedangkan gRPC menggunakan satu koneksi TCP untuk banyak stream.

Dari segi responsivitas dan komunikasi real-time, REST API memiliki karakteristik dimana client mengirim satu request dan server mengirim satu response sekali. Untuk mendapatkan update real-time, client harus melakukan polling. Pendekatan ini dapat menyebabkan masalah latensi tinggi karena delay antara polling membuat data tidak benar-benar real-time serta overhead bandwidth karena banyaknya request HTTP. Sebaliknya, gRPC memungkinkan client dan server saling mengirim pesan secara asynchronous dalam satu koneksi. Keunggulan ini memberikan latensi rendah karena tidak ada delay polling dan efisiensi bandwidth berkat pengurangan overhead HTTP header dan koneksi TCP.

Dalam implementasinya, REST API dengan polling memiliki kelemahan seperti empty responses jika tidak ada data baru serta pemborosan bandwidth. Sementara gRPC dengan bidirectional streaming memungkinkan pesan dikirim tanpa delay dan efisien untuk banyak client. Dengan demikian, REST API sebaiknya dipilih apabila komunikasi tidak bersifat real-time dan memiliki prioritas simpilicty, compatibility, dan statelessness. Sebaliknya, gRPC menjadi pilihan tepat ketika dibutuhkan interaksi real-time dengan latensi minimal dan efisiensi tinggi.


## 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

Pemilihan antara Protocol Buffers (Protobuf) di gRPC dan JSON di REST API memiliki dampak signifikan pada beberapa aspek pengembangan. Dari segi validasi data dan keamanan, Protobuf menggunakan skema ketat yang didefinisikan dalam file `.proto`, menawarkan validasi otomatis, dan memiliki risiko rendah terhadap injeksi. JSON bersifat schema-less dengan struktur data fleksibel, tetapi memerlukan validasi manual dan lebih rentan terhadap masalah keamanan seperti XSS/SQL injection.

Dalam hal performa, Protobuf menggunakan format biner yang ringkas, proses serialisasi lebih cepat, dan efisien untuk data besar. Sementara itu, JSON menggunakan format teks yang lebih verbosa, proses parsing teks yang lebih lambat, dan kurang efisien untuk bandwidth terutama pada array kompleks. Dalam segi fleksibilitas, Protobuf memerlukan pembaruan file `.proto` dan regenerasi kode untuk perubahan struktur, sedangkan JSON lebih mudah beradaptasi dengan perubahan struktur data tanpa perlu skema tetap.

Untuk interoperabilitas, Protobuf menawarkan pembuatan kode otomatis di berbagai bahasa melalui protoc dengan konsistensi lintas platform, sementara JSON tidak memiliki generasi kode otomatis dan bergantung pada implementasi parser. Dari segi keterbacaan, Protobuf tidak dapat dibaca langsung dan memerlukan tools khusus untuk debugging, sedangkan JSON mudah dibaca dan diinspeksi secara langsung.

Protobuf dengan gRPC cocok untuk microservices, IoT/real-time, throughput tinggi, dan komunikasi internal. REST API dengan JSON lebih cocok untuk API publik, frontend web, integrasi dengan sistem eksternal, dan pengembangan cepat. Kedua pendekatan dapat digunakan secara bersamaan dalam satu sistem.