---
Pertemuan 03: Sinkronisasi dan Koordinasi pada Sistem Terdistribusi"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
Usat Jalung: 255410017 
dosen: "Dr. Bambang Purnomosidi D. P."
from: tambahakan teks yang kurang
rakuman  di buat : awalan di buat manual laul di lenkapi dengan ia
---

# Sinkronisasi dan Koordinasi

Sinkronisasi berarti mengatur kerja banyak proses dan komputer supaya hasilnya tetap benar dan tidak saling bertabrakan. Contoh masalahnya mudah dilihat di toko online. Stok produk X ada 10. Pembeli A dan pembeli B sama-sama membaca stok 10, lalu masing-masing membeli satu. Keduanya menulis stok baru menjadi 9. Padahal sudah terjadi dua pembelian, jadi stok seharusnya 8. Kejadian seperti ini disebut data race atau race condition. Selain menjaga data tetap benar, sinkronisasi juga membantu mencegah deadlock (proses saling menunggu tanpa akhir), membagi beban kerja, dan menjaga keadaan aplikasi tetap sama di semua komputer.

Dua istilah tadi sedikit berbeda. Data race terjadi di tingkat memori, yaitu dua thread mengakses data yang sama, minimal satu menulis, tanpa pengaman. Race condition terjadi di tingkat logika program, yaitu hasil yang benar bergantung pada urutan kejadian yang tidak bisa ditebak, seperti saldo bank yang jadi minus karena dua penarikan berjalan bersamaan. Data race diatasi dengan mutex atau lock, sedangkan race condition diatasi dengan merancang ulang alurnya, misalnya memakai transaksi database.

Untuk urusan waktu ada dua pendekatan. Pendekatan pertama menyamakan jam yang asli (waktu fisik). Standar yang dipakai sekarang adalah UTC yang berdasar pada jam atom (TAI). Jam atom ada di satelit GPS, lalu waktunya diteruskan lewat GPS dongle ke server, kemudian ke komputer dan ponsel lewat protokol NTP atau PTP. NTP sudah ada sejak 1985 dan di Linux bisa dipakai lewat program chrony. Akurasinya beberapa milidetik lewat internet dan puluhan mikrodetik di jaringan lokal. PTP lebih presisi, tetapi kartu jaringannya harus mendukung hardware timestamping.

Pendekatan kedua memakai waktu logis, yaitu hanya mengurus urutan sebab-akibat tanpa peduli jam asli. Setelah Lamport clock, ada vector clock. Setiap proses menyimpan satu angka untuk tiap proses lain dalam bentuk daftar, misalnya [0, 0, 0] untuk tiga proses. Setiap ada kejadian, angka milik sendiri ditambah satu. Saat menerima pesan, ambil angka terbesar untuk tiap posisi, lalu tambah angka milik sendiri. Kejadian A dianggap terjadi sebelum B kalau semua angka A lebih kecil atau sama dengan B dan ada minimal satu yang lebih kecil. Kalau tidak begitu, kedua kejadian itu dianggap berjalan bersamaan dan tidak saling bergantung.

Topik berikutnya adalah mutual exclusion, yaitu memastikan hanya satu proses yang boleh masuk ke bagian kode yang memakai sumber daya bersama (critical section) pada satu waktu. Ada tiga cara. Cara terpusat memakai satu koordinator yang memberi izin. Ini sederhana, tetapi kalau koordinatornya rusak semuanya berhenti. Cara Ricart-Agrawala tidak butuh koordinator karena setiap proses meminta izin ke semua proses lain, tetapi jumlah pesannya banyak. Cara token memakai satu token yang berpindah-pindah, dan hanya pemegang token yang boleh masuk. Ini hemat pesan, tetapi sulit mengetahui kalau token hilang.

Terakhir, kadang sistem perlu satu pemimpin untuk mengelola sumber daya dan mengambil keputusan bersama. Kalau pemimpin rusak atau ada komputer baru yang bergabung, sistem harus memilih pemimpin baru secara otomatis. Algoritma yang dipakai antara lain Bully, Ring, serta Raft dan Paxos. Contoh nyatanya, ZooKeeper memakai Zab yang mirip Paxos, dan Kubernetes memakai Raft lewat etcd.

**yang harus di ingat yah itu:** tanpa sinkronisasi, data bisa rusak karena proses saling menimpa. Waktu bisa diatur dengan jam fisik (NTP dan PTP) atau jam logis (Lamport dan vector clock), dan akses bersama dijaga dengan mutual exclusion serta pemilihan pemimpin.
