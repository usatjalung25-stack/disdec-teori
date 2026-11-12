---
title: "Minggu 02 — Proses dan Komunikasi Antar Proses pada Sistem Terdistribusi"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
sks: 3
prodi: Informatika
institusi: UTDI
dosen: "Dr. Bambang Purnomosidi D. P."
minggu: 2
topik: [proses, IPC, message passing, RPC, MOM, microservices, logical clock]
---
# Minggu 02 — Proses dan Komunikasi Antar Proses (IPC)

## Tujuan Pembelajaran
- Menjelaskan konsep **proses** pada sistem terdistribusi.
- Menjelaskan berbagai jenis **komunikasi antar proses (IPC)**.
- Menganalisis isu **sinkronisasi, reliabilitas, dan keamanan**.
- Mengaitkan konsep ke contoh nyata: microservices, cloud, distributed messaging.

## Konsep Utama & Penjelasan

### 1. Proses dan manajemen proses
- **Program** disimpan di disk; saat dijalankan menjadi **proses** di RAM. Proses dapat berisi beberapa **thread** yang dijadwalkan oleh *scheduler* OS ke CPU (*context switch*, *preemption*).
- Manajemen proses: **process creation**, **process migration**, **process monitoring**.

### 2. Proses dan komunikasi pada sistem terdistribusi
- Setiap node menjalankan proses; agar bekerja sebagai satu kesatuan logis, proses antar node harus **berkomunikasi**.
- Proses bisa berada di node berbeda, sehingga **tidak ada shared memory** → dibutuhkan IPC khusus.
- Contoh: multiplayer games, Gojek/Grab (UI ↔ service driver, pelacakan lokasi, hitung biaya), chat/video call WhatsApp.

### 3. Model komunikasi pesan
- **Langsung**: memakai *socket*.
- **Tidak langsung**: *mailbox / message queue*.
- **Pub/Sub**: publisher mengirim ke *topic*, subscriber menerima.

### 4. Empat jenis IPC
| Jenis | Keterangan |
|---|---|
| **Shared Memory** | Tidak cocok untuk sistem terdistribusi (tidak ada akses memori antar node) |
| **Message Passing** | Standar **MPI** (versi 5.0 menurut slide); implementasi Python: `mpi4py`; OpenMPI |
| **RPC** | Klien memanggil prosedur di server lewat *stub*; contoh **gRPC** (Proto Request/Response, lintas bahasa) |
| **Middleware (MOM)** | Infrastruktur pertukaran pesan **asinkron** antar aplikasi |

**Standar/protokol MOM**: XATMI, AMQP (enterprise apps), HLA IEEE 1516, MQTT (perangkat terbatas sumber daya), DDS (OMG), XMPP.

### 5. Sinkron vs asinkron
- **Sinkron**: klien mengirim request lalu **menunggu** respons.
- **Asinkron**: klien **lanjut bekerja**, respons ditangani saat tiba.

### 6. Masalah pada komunikasi antar proses
| Masalah | Solusi |
|---|---|
| **Reliability** (pesan lengkap & berurutan) | Acknowledgement, transmisi ulang, protokol andal seperti TCP |
| **Ordering** | Pipes (FIFO), message queues, shared memory, sockets/message passing, RPC |
| **Security** | Enkripsi, autentikasi |
| **Performance** | Perhatikan latency & bandwidth |

### 7. Protokol komunikasi
- Protokol dasar: **TCP, UDP, HTTP**.
- Berjalan di atas HTTP sebagai transport: **gRPC, GraphQL, REST**.
- HTTP/2 memakai TCP + TLS; HTTP/3 memakai **QUIC di atas UDP**.

### 8. Sinkronisasi clock dan logical clock (pengantar)
- Tiap node punya clock lokal yang **bergeser** (*drift*) akibat latensi jaringan dan perangkat keras.
- Sinkronisasi eksternal: atomic clock (TAI), GPS, UTC, lalu protokol **NTP** atau **PTP** (PTP lebih presisi).
- **Lamport Logical Clock** tidak terkait waktu nyata:
  1. Setiap kejadian → counter +1.
  2. Saat mengirim pesan → sertakan counter.
  3. Saat menerima → ambil `max(lokal, pesan)` lalu +1.
- Pengembangan lanjutan: Vector clocks (Fidge & Mattern, 1988), Plausible clocks, Chain clocks, Interval Tree clocks, Blum clocks.

### 9. Koordinasi proses, tantangan, dan isu terkini
- **Koordinasi**: Apache ZooKeeper, etcd, Consul, ClickHouse Keeper, Oxia, KRaft.
- **Tantangan & solusi**: message loss (retransmission + ACK), deadlock (timeout, detection algorithm), data inconsistency (Raft, Paxos), security (SSL/TLS, OAuth2).
- **Current issues**: Edge & Fog Computing, observability & tracing (telemetry), container orchestration (Kubernetes, Docker Swarm).

## Contoh / Studi Kasus

**Socket server (echo)**
```python
import socket

HOST = '127.0.0.1'
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print(f"Connected by {addr}")
        while True:
            data = conn.recv(1024)
            if not data:
                break
            print(f"Received: {data.decode()}")
            conn.sendall(data)  # echo back
```

**Socket client**
```python
import socket

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect(('127.0.0.1', 65432))
    s.sendall(b"Hello, world!")
    data = s.recv(1024)

print(f"Received from server: {data.decode()}")
```

**Lamport clock (3 service)**: Sales → Product → Invoice, semua mulai dari 0.
1. Sales mencatat transaksi → clock 1, kirim ke Product dengan (1).
2. Product menerima (1), `max(0,1)=1`, +1 → 2, kirim ke Invoice dengan (2).
3. Invoice menerima (2), `max(0,2)=2`, +1 → 3, membuat faktur.

Urutan yang benar terjaga walaupun timestamp jam nyata bisa bergeser.

**Microservices**: UI terhubung ke banyak microservice, masing-masing dengan database sendiri.

## Poin Penting (Key Takeaways)
- Karena tidak ada shared memory antar node, sistem terdistribusi mengandalkan **message passing, RPC, dan middleware**.
- Pilihan **sinkron vs asinkron** dan jaminan **reliability, ordering, security** menentukan desain komunikasi antar service.
- Karena jam fisik tidak bisa dipercaya sepenuhnya, **logical clock** dipakai untuk menjaga urutan kejadian.

## Bahan Diskusi
1. Apakah GraphQL merupakan contoh RPC?
2. Apakah REST merupakan contoh RPC?
3. Apakah telemetry merupakan solusi etis?

## Glosarium
| Istilah | Pengertian |
|---|---|
| IPC | Inter-Process Communication, komunikasi antar proses |
| Socket | Titik akhir komunikasi jaringan langsung antar proses |
| RPC | Pemanggilan prosedur di mesin lain seolah lokal |
| Stub | Perwakilan lokal yang mengemas dan mengirim panggilan RPC |
| MOM | Message-Oriented Middleware |
| Pub/Sub | Pola publish–subscribe lewat topic |
| MPI | Message Passing Interface, standar message passing |
| Lamport clock | Counter logis untuk mengurutkan kejadian |
| NTP / PTP | Protokol sinkronisasi waktu (PTP lebih presisi) |