---
title: "Minggu 04 — Konsistensi dan Replikasi pada Sistem Terdistribusi"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
sks: 3
prodi: Informatika
institusi: UTDI
dosen: "Dr. Bambang Purnomosidi D. P."
minggu: 4
topik: [konsistensi, CAP, model konsistensi, replikasi, quorum, CRDT, OT, gossip]
---
FROM
┌─────────────────────────────────────────────┐
│              FORM RANGKUMAN                 │
├─────────────────────────────────────────────┤
│                                             │
│  Judul       : ___________________________  │
│                                             │
│  Mata Kuliah : ___________________________  │
│                                             │
│  Kode        : ___________________________  │
│                                             │
│  SKS         : ___________________________  │
│                                             │
│  Prodi       : ___________________________  │
│                                             │
│  Institusi   : UTDI                         │
│                                             │
│  Dosen       : Dr. Bambang Purnomosidi D.P. │
│                                             │
│  Minggu      : 4                            │
│                                             │
│  Topik       : ___________________________  │
│                ___________________________  │
│                ___________________________  │
│                                             │
└─────────────────────────────────────────────┘
# Minggu 04 — Konsistensi dan Replikasi

## Tujuan Pembelajaran
- Menjelaskan konsep **konsistensi** dan **model konsistensi**.
- Menjelaskan konsep **replikasi data**.
- Menjelaskan **protokol dan strategi replikasi**.
- Menganalisis studi kasus replikasi di industri (PostgreSQL, Apache Ignite).

## Konsep Utama & Penjelasan

### 1. Konsistensi dan mengapa penting
- Menjamin semua node memiliki data yang **sama (atau tampak sama)** meski tersebar dan diubah bersamaan.
- Tanpa konsistensi, pengguna melihat versi data berbeda. Contoh: transfer dari A ke B, saldo A sudah berkurang tetapi saldo B belum bertambah di server lain.

### 2. Teorema CAP
- Partisi jaringan pasti terjadi, sehingga kondisi ideal C+A+P sekaligus tidak mungkin.
- Contoh e-commerce (Server A barat, Server B timur, jaringan terputus):
  - **AP**: tetap bisa belanja, tetapi stok sementara tidak sinkron.
  - **CP**: transaksi ditolak sampai jaringan pulih (mengorbankan availability).

### 3. Model konsistensi (dari kuat ke lemah)
| Model | Inti | Contoh |
|---|---|---|
| **Linearizable** | Paling kuat; operasi tampak atomik dan real-time, seperti satu node | etcd, ZooKeeper, Spanner; cek keunikan username |
| **Sequential** | Semua node melihat urutan operasi yang sama (tak harus real-time) | Urutan posting media sosial |
| **Causal** | Operasi yang bergantung harus terjaga urutannya | Komentar tidak muncul sebelum postingnya |
| **Read-your-writes** | Penulis langsung melihat tulisannya sendiri | Pesan langsung tampil di layar pengirim |
| **Monotonic read** | Tidak membaca versi yang lebih lama dari yang pernah dilihat | Timeline tidak "mundur" saat refresh |
| **Eventual** | Akhirnya konsisten bila tidak ada update baru | DynamoDB, Cassandra |

- **Strong consistency** biasanya memakai Two-Phase Commit atau Paxos (contoh: perbankan).
- **BASE** (untuk eventual): *Basically Available, Soft state, Eventual consistency*.

### 4. Trade-off latensi vs konsistensi
- Konsistensi ↑ → latensi ↑; konsistensi ↓ → latensi ↓.
- Contoh dua pusat data **Jakarta** dan **Yogyakarta**:
  - *Strong*: tunggu kedua pusat data sama sebelum menjawab (data konsisten, respons lambat).
  - *Eventual*: jawab langsung, sinkronisasi menyusul (cepat, data bisa berbeda sementara).

### 5. Replikasi data
- Membuat dan memelihara **salinan data** yang sama di banyak node.
- **Tujuan**: availability, performance (baca dari server terdekat), reliability, skalabilitas (beban baca dibagi).

### 6. Jenis replikasi
- **Master–Slave (Primary–Secondary)**: satu node menulis, lainnya membaca. Konsisten, tapi master jadi bottleneck (PostgreSQL streaming replication).
- **Multi-Master (P2P)**: semua node baca/tulis. Fleksibel tapi rawan konflik (PostgreSQL logical replication, Cassandra, MongoDB, Apache Ignite).
- **Synchronous**: update ditulis ke semua node dulu. Konsisten, lambat (Google Spanner).
- **Asynchronous**: update dikirim setelah commit di master. Cepat, bisa terjadi delay (MySQL async, DynamoDB).

### 7. Strategi sinkronisasi replika
| Dimensi | Pilihan |
|---|---|
| Waktu | **Asynchronous** (tercepat, risiko kehilangan data tertinggi), **Semi-synchronous** (tunggu satu replika; dipakai MySQL), **Synchronous** (tunggu semua replika) |
| Arah | master-slave, master-master, master-master asimetris |
| Cara | **Pull-based**, **Push-based** (CDC + WebSocket/SSE), **Hybrid** |
| Penanganan konflik | **LWW**, **Vector Clock**, **CRDT**, **OT** |

### 8. Protokol replikasi
1. **Primary-based** (master–slave).
2. **Quorum-based**: operasi sukses bila dikonfirmasi jumlah node tertentu; syarat **W + R > N**.
3. **State Machine Replication**: replika mengeksekusi perintah yang sama dalam urutan yang sama lewat log yang di-commit leader.
4. **Gossip Protocol**: node menyebarkan status ke 1–3 peer acak secara berulang (dipakai Cassandra untuk cek node dan replikasi).
5. **Chain Replication**: node tersusun berantai (head → … → tail); write masuk ke head, read dari tail.

## Contoh / Studi Kasus

**Quorum** (N=3, W=2, R=2 → 2+2 > 3):
```
Write x=100 → dikirim ke Node A, B, C
  A: ACK   B: ACK   C: lambat/timeout
  2 ACK terkumpul (W=2) → write dianggap BERHASIL

Read x → minta ke 2 node (R=2), mis. A dan C
  A: x=100 (terbaru)   C: mungkin masih versi lama
  Irisan dengan node yang ikut write menjamin data terbaru terbaca
```

**State machine replication**: A, B, C awal `x=10`. Klien kirim `SET x=20` ke leader A → A beri nomor urutan, tulis ke log, kirim ke B dan C → setelah B dan C konfirmasi, perintah "committed" → semua mengeksekusi sehingga `x=20` → leader membalas klien.

**CRDT (counter pengunjung website)**: dua pengguna mencatat lokal `{A:5, B:0}` dan `{A:0, B:3}`. Saat sinkronisasi, aturan **max per entri**: `A=max(5,0)=5`, `B=max(0,3)=3` → total 5+3 = **8**.

**OT (Operational Transformation)**: teks `"abc"`. A menyisipkan `x` di indeks 0, B menghapus `c` di indeks 2. Tanpa transformasi hasilnya berbeda (`xac` vs `xab`). Dengan OT, operasi hapus B ditransformasi menjadi indeks 3 di sisi A, sehingga keduanya sama-sama menghasilkan `"xab"`.

**Studi kasus lain**: PostgreSQL 18 streaming replication dan Apache Ignite 3 master-master (detail ada pada lecture notes dan source code yang menyertai materi).

## Poin Penting (Key Takeaways)
- Semakin kuat konsistensi, semakin besar latensi; pilih model sesuai kebutuhan (bank = strong, media sosial = eventual).
- Replikasi meningkatkan **availability, performa, reliabilitas, dan skalabilitas**, tetapi memunculkan masalah sinkronisasi dan **konflik** yang ditangani LWW, vector clock, CRDT, atau OT.
- Protokol seperti **quorum (W+R>N)**, state machine replication, gossip, dan chain replication adalah cara praktis menjaga salinan data tetap selaras.

## Bahan Diskusi
1. Apa kelebihan dan kekurangan strong vs eventual consistency?
2. Bagaimana jika replikasi gagal di satu node?

## Glosarium
| Istilah | Pengertian |
|---|---|
| Konsistensi | Semua node melihat data yang sama (atau tampak sama) |
| Replikasi | Menyalin dan memelihara data di banyak node |
| BASE | Basically Available, Soft state, Eventual consistency |
| Quorum | Jumlah minimal node yang harus setuju agar operasi sah |
| LWW | Last Write Wins, tulisan dengan timestamp terbaru menang |
| CRDT | Conflict-free Replicated Data Type, struktur data yang bergabung otomatis tanpa konflik |
| OT | Operational Transformation, transformasi operasi untuk menyatukan edit bersamaan |
| CDC | Change Data Capture, penangkapan perubahan data di sumber |
| Gossip protocol | Penyebaran informasi lewat pertukaran acak antar node |
| Linearizability | Bentuk strong consistency paling kuat |