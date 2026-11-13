---
Pertemuan 02: Proses dan Komunikasi Antar Proses pada Sistem Terdistribusi
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi
kode: FIF25002T
dosen: "Dr. Bambang Purnomosidi D. P.
---

# Pertemuan 02: Proses dan Komunikasi Antar Proses

Proses adalah program yang sedang berjalan. Program itu sendiri hanya tumpukan instruksi yang tersimpan di disk. Begitu dijalankan, program dimuat ke memori dan menjadi proses, lalu sistem operasi mengatur kapan proses itu mendapat giliran memakai CPU. Sistem operasi mengelola proses lewat tiga hal, yaitu membuat proses baru, memindahkan proses ke tempat lain, dan memantau kondisi proses.

Dalam sistem terdistribusi, setiap komputer (node) menjalankan prosesnya sendiri. Supaya semuanya bisa bekerja seperti satu sistem, proses-proses itu harus saling bertukar informasi. Contohnya aplikasi ojek online yang punya layanan terpisah untuk data pengemudi, pelacakan lokasi, dan penghitung biaya, atau chat dan video call di WhatsApp. Masalahnya, proses yang berada di komputer berbeda tidak bisa memakai memori yang sama. Itu sebabnya cara komunikasi lewat shared memory tidak cocok dipakai di sistem terdistribusi.

Cara yang dipakai adalah saling mengirim pesan. Pengiriman pesan bisa langsung memakai socket, bisa tidak langsung lewat antrean pesan (message queue), atau memakai pola publish/subscribe, di mana pengirim menaruh pesan di sebuah topik dan semua pelanggan topik itu menerimanya. Ada empat jenis komunikasi antar proses. Pertama shared memory yang tadi sudah disebut tidak cocok. Kedua message passing, dengan standar bernama MPI. Ketiga RPC (Remote Procedure Call), yaitu memanggil fungsi yang ada di komputer lain seolah-olah fungsi itu ada di komputer sendiri, contohnya gRPC. Keempat middleware berbasis pesan (MOM) yang mengirim pesan secara tidak menunggu, dengan protokol seperti AMQP, MQTT, dan XMPP.

Komunikasi juga dibedakan menjadi sinkron dan asinkron. Pada komunikasi sinkron, pengirim menunggu sampai jawaban datang. Pada komunikasi asinkron, pengirim melanjutkan pekerjaan lain dan jawaban diurus saat sudah tiba. Dalam praktiknya ada beberapa masalah yang harus diatasi. Pesan bisa hilang, jadi dibutuhkan konfirmasi dan pengiriman ulang. Urutan pesan bisa kacau, jadi perlu antrean yang menjaga urutan. Pesan juga harus aman dengan enkripsi dan autentikasi, dan kecepatan jaringan harus diperhatikan. Protokol dasarnya TCP, UDP, dan HTTP, sedangkan gRPC, GraphQL, dan REST berjalan di atas HTTP.

Satu persoalan lagi adalah waktu. Setiap komputer punya jam sendiri, dan jam-jam itu lama-lama bergeser. Jam bisa disamakan dengan bantuan jam atom, GPS, dan protokol NTP atau PTP. Tetapi kalau yang penting hanya urutan kejadian, bisa dipakai logical clock buatan Leslie Lamport. Caranya, setiap kejadian menambah angka penghitung satu. Saat mengirim pesan, angka itu ikut dikirim. Saat menerima pesan, komputer mengambil angka yang lebih besar antara miliknya dan yang diterima, lalu menambah satu. Dengan cara ini urutan kejadian tetap benar walau jam komputernya tidak sama. Di bagian akhir disebut juga alat koordinasi seperti ZooKeeper dan etcd, protokol konsensus Raft dan Paxos, serta isu terbaru seperti edge computing, observability, dan Kubernetes.

**Yang harus di ingat yah itu :** karena tidak ada memori bersama antar komputer, komunikasi dilakukan dengan mengirim pesan, dan urutan kejadian dijaga dengan logical clock.
