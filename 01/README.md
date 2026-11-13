---
Pertemuan 01: Pengenalan Sistem Terdistribusi dan Terdesentralisasi"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
Usat Jalung: 255410017 
dosen: "Dr. Bambang Purnomosidi D. P."
---

# Pengenalan Sistem Terdistribusi dan Terdesentralisasi

Aplikasi yang kita pakai sehari-hari seperti Google, Netflix, atau YouTube dipakai oleh jutaan orang pada waktu yang sama dan harus bisa diakses kapan saja. Kalau semuanya dijalankan di satu komputer saja, layanan itu pasti tidak sanggup. Karena itu dibutuhkan sistem terdistribusi, yaitu sekumpulan komputer yang bekerja sama lewat jaringan, tetapi bagi pengguna terlihat seperti satu sistem saja. Pengguna tidak perlu tahu data mereka disimpan di server mana, atau apa yang terjadi kalau satu server mati. Dua hal yang penting di sini adalah skalabilitas, yaitu kemampuan sistem menangani beban yang makin besar, dan fault tolerance, yaitu kemampuan sistem tetap jalan walaupun sebagian komponennya rusak.

Ada tiga bentuk arsitektur dasar. Yang pertama client-server, di mana banyak pengguna terhubung ke satu server pusat. Bentuk ini sederhana, tetapi kalau servernya mati maka semuanya ikut mati. Yang kedua peer-to-peer, di mana semua komputer kedudukannya sama dan saling terhubung langsung tanpa server pusat, seperti BitTorrent dan Bitcoin. Yang ketiga hybrid, yaitu campuran keduanya. Aplikasi seperti WhatsApp dan Telegram memakai supernode sebagai penghubung, sementara pengiriman file bisa langsung antar pengguna.

Selain terdistribusi, ada juga istilah terdesentralisasi. Bedanya terletak pada fokusnya. Sistem terdistribusi membagi beban kerja ke banyak komputer, sedangkan sistem terdesentralisasi membagi kontrol atau kekuasaan sehingga tidak ada satu pihak yang memegang semuanya. Keputusan diambil bersama lewat kesepakatan yang disebut konsensus. Contoh sistem terdistribusi adalah Google File System, Apache Kafka, dan Netflix. Contoh sistem terdesentralisasi adalah blockchain, DNS, dan BitTorrent. Dua sistem ini saling berkaitan, tetapi tidak sama.

Materi ini juga membahas CAP Theorem dari Eric Brewer. Ada tiga hal yang diinginkan dari sistem data, yaitu data selalu sama di semua tempat (consistency), layanan selalu bisa dipakai (availability), dan sistem tetap jalan walau jaringan antar komputer terputus (partition tolerance). Karena jaringan yang terputus pasti bisa terjadi, kita harus memilih salah satu saat itu terjadi. Kalau memilih tetap melayani pengguna, datanya bisa sementara tidak sama. Kalau memilih menjaga data tetap sama, sebagian permintaan harus ditolak dulu. Di dalam slide, MongoDB dicontohkan sebagai pilihan yang pertama dan Apache Cassandra sebagai pilihan yang kedua. Terkait ini ada dua jenis konsistensi data. Strong consistency artinya hasilnya selalu sama di semua tempat, cocok untuk bank. Eventual consistency artinya data akan sama pada akhirnya, cukup untuk media sosial.

Di bagian akhir dibahas tantangan dan arah ke depan. Tantangannya antara lain latency (berapa lama sebuah permintaan diproses), throughput (berapa banyak yang bisa diproses), serangan keamanan seperti DDoS dan Sybil attack (penyerang membuat banyak identitas palsu supaya punya pengaruh besar), rumitnya mengelola banyak komputer, dan soal tata kelola pada sistem terdesentralisasi. Tren yang sedang berkembang adalah cloud-native dengan Kubernetes dan Docker, edge dan fog computing, Web3 dan DeFi, serta federated learning.

**yang harus di ingat yah itu :** sistem terdistribusi berarti berbagi pekerjaan, sistem terdesentralisasi berarti berbagi kekuasaan, dan keduanya menjadi dasar teknologi modern.
