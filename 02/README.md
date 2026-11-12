## Pertemuan 02: Proses dan Komunikasi Antar Proses
## format yang saya gunakan: 
buat kan rakuman Materi Minggu 01.02.03.04 : .......
Prompt: .....
buat dalam format REAMDE.md

### 1. Proses pada Sistem Terdistribusi
- Proses berjalan pada node yang berbeda dan tidak memiliki shared memory.
- Oleh karena itu, komunikasi antar proses (IPC) sangat penting.

### 2. Model Komunikasi Pesan
- **Langsung**: Menggunakan socket (TCP/UDP) untuk komunikasi point-to-point.
- **Tidak Langsung**: Melalui perantara seperti message queue atau model **Pub/Sub** (Publisher/Subscriber).

### 3. IPC (Inter-Process Communication)
- **Shared Memory**: Tidak cocok untuk sistem terdistribusi karena tidak ada memori bersama.
- **Message Passing**: Proses bertukar pesan melalui jaringan (contoh: MPI).
- **RPC (Remote Procedure Call)**: Memanggil prosedur di komputer lain seolah-olah lokal. Contoh: gRPC.
- **MOM (Message-Oriented Middleware)**: Infrastruktur untuk komunikasi asinkron melalui pesan. Contoh protokol: AMQP, MQTT.

### 4. Komunikasi Sinkron & Asinkron
- **Sinkron**: Klien mengirim permintaan dan menunggu respons dari server.
- **Asinkron**: Klien mengirim permintaan dan dapat melanjutkan pekerjaan lain tanpa menunggu respons.

### 5. Masalah pada Komunikasi
- **Reliability**: Memastikan pesan terkirim lengkap dan berurutan (solusi: ACK, retransmisi, TCP).
- **Ordering**: Memastikan pesan diproses dalam urutan yang logis.
- **Security**: Enkripsi dan autentikasi.
- **Performance**: Latency dan bandwidth.

### 6. Sinkronisasi Waktu
- **Waktu Fisik**: Menyelaraskan jam komputer dengan standar global (UTC) menggunakan protokol seperti **NTP** atau **PTP** (lebih presisi). Sumber waktu bisa dari atomic clock via GPS.
- **Waktu Logis**: Menentukan urutan kejadian tanpa terikat waktu nyata.
    - **Lamport Logical Clock**: Setiap kejadian menambah counter. Saat menerima pesan, ambil nilai max antara clock lokal dan clock pengirim, lalu tambah 1. Ini memastikan urutan sebab-akibat (causal ordering).
    - **Vector Clock**: Pengembangan dari Lamport Clock untuk mendeteksi kejadian yang **konkuren** (independen).

### 7. Koordinasi Proses
- **Mutual Exclusion**: Memastikan hanya satu proses yang mengakses critical section.
    - **Centralized**: Satu koordinator mengatur izin.
    - **Ricart-Agrawala**: Terdistribusi penuh, menggunakan timestamp dan pesan OK dari semua proses lain.
    - **Token-Based**: Sebuah token unik beredar; hanya pemegang token yang bisa masuk.
- **Pemilihan Koordinator (Leader Election)**: Memilih pemimpin baru saat koordinator gagal. Algoritma: Bully, Ring, atau konsensus (Raft, Paxos).
