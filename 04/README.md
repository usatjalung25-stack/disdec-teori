---
Pertemuan 04: Konsistensi dan Replikasi pada Sistem Terdistribusi"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
Usat Jalung: 255410017 
dosen: "Dr. Bambang Purnomosidi D. P."
from: tambahakan teks yang kurang
rakuman  di buat : awalan di buat manual laul di lenkapi dengan ia
---
# Konsistensi dan Replikasi

Konsistensi artinya semua komputer dalam sistem memiliki data yang sama, atau paling tidak terlihat sama bagi pengguna. Data yang disimpan di banyak tempat bisa berbeda satu sama lain. Contohnya saat nasabah mentransfer uang dari rekening A ke B. Kalau server pertama sudah mengurangi saldo A tetapi server kedua belum menambah saldo B, nasabah bisa gagal bertransaksi karena saldo B belum terlihat. Itulah sebabnya konsistensi penting.

Pertemuan ini mengulang CAP Theorem dengan contoh toko online yang punya dua server, satu untuk wilayah barat dan satu untuk wilayah timur Indonesia. Kalau jaringan antar keduanya terputus, ada dua pilihan. Memilih availability berarti pengguna masih bisa belanja, tetapi stok bisa sementara tidak cocok. Memilih consistency berarti transaksi ditolak dulu sampai jaringan pulih, supaya stok tetap benar.

Model konsistensi adalah aturan tentang kapan perubahan data bisa dilihat oleh pihak lain. Urutannya dari yang paling ketat sampai yang paling longgar adalah sebagai berikut. Linearizable membuat sistem terlihat seperti satu komputer saja, dipakai antara lain di etcd, ZooKeeper, dan Spanner, misalnya untuk memastikan dua orang tidak bisa mendaftar dengan username yang sama. Sequential membuat semua orang melihat urutan operasi yang sama walau agak terlambat. Causal menjaga urutan operasi yang saling bergantung, misalnya komentar tidak boleh muncul sebelum postingannya. Read-your-writes menjamin seseorang langsung melihat hasil tulisannya sendiri, seperti pesan yang langsung tampil di layar pengirim. Monotonic read menjamin seseorang tidak melihat data yang lebih lama dari yang pernah ia lihat, jadi timeline tidak mundur saat di-refresh. Eventual consistency hanya menjamin data akhirnya sama, seperti pada DynamoDB dan Cassandra, dengan prinsip yang disebut BASE. Semakin kuat konsistensinya, semakin lama waktu respons, karena sistem harus menunggu semua komputer sama dulu. Sebaliknya konsistensi yang longgar membuat respons cepat, tetapi datanya bisa belum terbaru.

Replikasi data adalah membuat dan menjaga salinan data yang sama di beberapa komputer. Manfaatnya, sistem tetap tersedia kalau satu server rusak, pengguna bisa membaca dari server terdekat sehingga lebih cepat, risiko kehilangan data lebih kecil, dan beban baca bisa dibagi. Jenisnya ada master-slave, yaitu satu server yang menulis dan yang lain hanya membaca, serta multi-master yang semuanya boleh menulis tetapi rawan konflik. Dari sisi waktu, replikasi sinkron menunggu semua salinan selesai sehingga konsisten tetapi lambat, replikasi asinkron langsung selesai lalu mengirim ke salinan belakangan sehingga cepat tetapi bisa terlambat, dan semi-sinkron menunggu satu replika saja.

Kalau dua pengguna mengubah data yang sama, konflik harus diselesaikan. Ada empat cara. Last Write Wins memakai perubahan dengan waktu terbaru. Vector clock memakai urutan dari vector clock. CRDT memakai struktur data yang otomatis bisa digabung, misalnya penghitung pengunjung yang diambil nilai terbesarnya lalu dijumlahkan. Operational Transformation menyesuaikan posisi suatu perubahan terhadap perubahan lain, seperti pada teks "abc" yang disisipi huruf di depan oleh satu orang dan dihapus huruf terakhirnya oleh orang lain, sehingga hasil akhirnya sama-sama "xab".

Ada lima protokol replikasi yang dibahas. Primary-based memakai satu server utama. Quorum-based menganggap operasi berhasil kalau cukup banyak server setuju, dengan syarat jumlah penulis ditambah jumlah pembaca lebih besar dari jumlah seluruh server. Pada tiga server dengan syarat tulis dua dan baca dua, tulisan yang sudah disetujui dua server pasti terbaca oleh pembaca yang menanyai dua server. State machine replication membuat semua replika menjalankan perintah yang sama dalam urutan yang sama lewat log milik pemimpin. Gossip protocol menyebarkan kabar dari satu komputer ke komputer lain secara acak sampai semuanya tahu, dipakai Cassandra. Chain replication menyusun komputer berantai, tulisan masuk dari ujung depan dan bacaan diambil dari ujung belakang. Studi kasus yang dipakai adalah PostgreSQL 18 dengan streaming replication dan Apache Ignite 3 dengan master-master.

**yang harus di ingat yah itu:** makin kuat konsistensi, makin lambat responsnya. Replikasi membuat sistem lebih tersedia dan cepat, tetapi harus disertai cara menjaga salinan tetap sama dan cara menyelesaikan konflik.
