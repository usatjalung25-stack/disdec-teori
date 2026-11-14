---
Pertemuan 05: Fault Tolerance dan Load Balancing dengan nginx"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
nama: "Usat Jalung"
nim: 255410017
dosen: "Dr. Bambang Purnomosidi D. P."
from: tambahakan teks yang kurang dan buat dalam bentuk format README.md
rakuman  di buat : awalan di buat manual laul di lenkapi dengan ia dan di bantu membuat format README.md
---
# Fault Tolerance

Fault tolerance adalah kemampuan sebuah sistem untuk tetap berjalan dengan benar walaupun satu atau lebih komponennya mengalami kegagalan atau kesalahan. Sistem yang punya kemampuan ini disebut fault tolerant system. Kemampuan ini penting untuk sistem dengan high availability, mission critical, maupun life-critical, misalnya sistem perbankan atau peralatan medis.

Pada sistem terdistribusi dan terdesentralisasi, komponennya sangat banyak dan terhubung lewat jaringan. Koneksi antar komponen bisa bermasalah kapan saja. Semakin kritis sebuah aplikasi, semakin tinggi tuntutan agar sistem tetap berfungsi, sehingga kegagalan harus diantisipasi sejak awal dan bukan ditangani setelah terjadi.

## Jenis Fault

Fault dibedakan menjadi empat jenis. Transient fault bersifat sementara, misalnya karena sambaran petir. Intermittent fault muncul dan hilang berulang kali, misalnya karena hardware yang overheat. Permanent fault bersifat tetap sampai komponennya diperbaiki atau diganti. Byzantine fault terjadi ketika komponen memberi hasil yang acak atau tidak konsisten, seperti pengkhianat dalam ilustrasi para jenderal yang mengepung sebuah kastil, sehingga sulit dideteksi.

## Klasifikasi Fault Berdasarkan Sumber

Berdasarkan sumbernya, fault dibagi menjadi tiga. Hardware fault, misalnya disk crash dan power failure. Software fault, misalnya bug, memory leak, dan race condition. Communication fault, misalnya packet loss dan network partition.

## Teknik untuk Mencapai Fault Tolerance

Ada lima teknik utama yang dibahas.

**Redundansi dan replikasi.** Resource tidak hanya ada di satu tempat, tetapi disalin ke beberapa node. Tujuannya menghindari single point of failure, yaitu satu komponen yang kalau rusak membuat seluruh sistem berhenti. Materinya mengacu pada pertemuan sebelumnya tentang replikasi.

**Load balancing.** Aplikasi dijalankan dalam lebih dari satu instance, yaitu satu proses aplikasi penuh yang aktif. Lalu sebuah load balancer seperti HAProxy atau nginx membagi permintaan ke instance-instance tersebut. Kalau satu instance mati, instance lain masih bisa melayani. Praktiknya dibahas di bagian bawah.

**Checkpointing dan rollback recovery.** Checkpointing berarti menyimpan status sistem secara berkala ke media penyimpan. Rollback recovery berarti mengembalikan sistem ke checkpoint terakhir setelah terjadi kegagalan. Dengan begitu sistem pulih dari status yang tersimpan, bukan mulai dari awal, sehingga kehilangan data dan waktu pemrosesan bisa ditekan.

**Heartbeat dan failure detection.** Heartbeat adalah sinyal periodik yang dikirim antar komponen untuk menunjukkan bahwa komponen itu masih hidup. Failure detection memakai heartbeat tersebut. Kalau pemantau tidak menerima heartbeat dalam batas waktu tertentu, komponen itu dianggap gagal. Pola ini penting supaya kegagalan bisa dideteksi dan ditangani tepat waktu. Contoh sederhananya adalah script `check-server.py` yang mengecek apakah sebuah server di host dan port tertentu masih "up" atau sudah "down".

**Consensus dan coordination.** Consensus adalah proses ketika beberapa node menyepakati satu nilai, status, atau urutan operasi, walaupun ada kegagalan dan keterlambatan jaringan. Coordination adalah pengelolaan interaksi antar node yang lebih luas agar bekerja sama dengan konsisten, dan biasanya melibatkan consensus. Contohnya leader election pada kluster basis data dengan algoritma seperti Raft, update file di cloud storage seperti Google Drive yang dianggap final setelah disetujui mayoritas server, serta validasi transaksi pada blockchain.

## Retry dengan tenacity

Koneksi ke komponen di luar aplikasi selalu mungkin gagal. Library `tenacity` di Python dipakai untuk mengatur percobaan ulang (retry) pada proses seperti itu. Pada contoh `check-retry.py`, program mencoba mengakses `http://localhost:44777` beberapa kali. Kalau server hidup, langsung berhasil. Kalau server mati, setelah beberapa kali percobaan program berhenti dengan pesan gagal akses.

## Circuit Breaker

Circuit breaker adalah pola desain pada microservices untuk mencegah kegagalan berantai dengan memblokir sementara panggilan ke layanan yang sedang gagal. Cara kerjanya seperti state machine dengan tiga status. Pada status closed, permintaan diizinkan dan dipantau. Kalau jumlah kegagalan mencapai batas, statusnya berpindah ke open, sehingga permintaan berikutnya langsung ditolak dan layanan punya waktu untuk pulih. Setelah waktu tunggu habis, statusnya menjadi half-open, yaitu hanya beberapa permintaan uji yang diizinkan. Kalau uji berhasil, status kembali closed, dan kalau gagal, status kembali open. Pada contoh `check-circuit-breaker.py` dipakai library `pybreaker` dengan `fail_max=4` dan `reset_timeout=20`, dan fungsi yang dilindungi diberi dekorator `@breaker`.

## Actor Model untuk Supervisi Proses

Pada actor model, program terdiri dari banyak actor yang masing-masing punya mailbox dan hanya berkomunikasi lewat pengiriman pesan. Untuk fault tolerance, actor disusun berjenjang dalam pola supervisor-worker. Supervisor memeriksa apakah actor di bawahnya masih hidup dan akan me-restart actor yang gagal, sedangkan supervisor sendiri diawasi oleh top level supervisor di atasnya.

## Implementasi pada Berbagai Bahasa Pemrograman

Erlang dan Elixir sudah punya mekanisme bawaan untuk actor model dan supervisor-worker. Pony adalah bahasa yang memang dikhususkan untuk actor model. Bahasa lain biasanya memakai library pihak ketiga. Di Python ada Xoscar, tenacity, dan library circuit breaker. Di Java ada Apache Pekko untuk actor model dan supervisor-worker, serta Resilience4j untuk fault tolerance. Library untuk bahasa lain bisa dicari lewat npm untuk JavaScript, crates.io untuk Rust, dan sebagainya.

# Praktik Load Balancing dengan nginx

Praktik ini menunjukkan teknik load balancing secara nyata. Aplikasinya adalah aplikasi web ASGI Python yang dibuat dengan framework BlackSheep, hanya aplikasi bawaan untuk memahami load balancing. Software yang diperlukan adalah Docker dan Docker Compose. Untuk Python disarankan memakai env manager seperti uv, Micromamba, atau Miniconda.

Langkah pembuatan aplikasi: install `blacksheep` dan `blacksheep-cli`, lalu jalankan `blacksheep create` dengan nama proyek `blacksheep_lb` dan template mvc. Install dependensi dengan `uv pip install -r requirements.txt`, lalu jalankan `python dev.py`. Aplikasi bisa diakses di `http://localhost:44777`. Proyek yang dihasilkan sudah punya Dockerfile bawaan yang membuka port 80, bukan 44777. Ini penting diingat karena port akan dipakai di Docker Compose. Perubahan yang dilakukan pada Dockerfile hanya versi Python menjadi `3.14.4-slim`.

Susunan proyeknya ada dua komponen utama, yaitu aplikasi BlackSheep yang di-scale menjadi 2 instance dan proxy nginx sebagai load balancer. Strukturnya terdiri dari direktori `blacksheep_lb`, direktori `nginx` (berisi `Dockerfile` dan `nginx.conf`), file `docker-compose.yml`, dan `env.sh` yang opsional untuk alias perintah di Linux Bash.

Dockerfile nginx memakai image `nginx`, menghapus `default.conf`, lalu menyalin `nginx.conf` buatan sendiri. Pada `nginx.conf`, blok `upstream backend` berisi `server bs_app:80`, dan blok `server` mendengarkan port 80 dengan `proxy_pass http://backend`. Nama `bs_app` harus sama dengan nama service di `docker-compose.yml`, dan nama `backend` harus sama dengan nama upstream.

File `docker-compose.yml` mendefinisikan dua service. Service `bs_app` dibangun dari `./blacksheep_lb`. Service `nginx` dibangun dari `./nginx`, memetakan port `80:80`, dan `depends_on` ke `bs_app`. Jumlah instance ditentukan saat menjalankan Compose dengan opsi `--scale bs_app=2`. Alias di `env.sh` antara lain `dcu` untuk menjalankan, `dcd` untuk mematikan, `dps` untuk melihat container, dan `dcln` untuk melihat log nginx.

Sebelum menjalankan Docker, service `dockerd` harus aktif, yaitu dengan `sudo dinitctl start dockerd` pada sistem dinit atau `sudo systemctl start dockerd` pada Linux dengan Systemd. Setelah `dcu`, aplikasi diakses lewat `http://localhost` tanpa port, sehingga memakai port 80. Perintah `docker ps` menunjukkan dua instance, yaitu `load-balancing-bs_app-1` dan `load-balancing-bs_app-2`, ditambah container nginx. Awalan nama berasal dari nama direktori.

Untuk membuktikan load balancing berjalan, lihat log masing-masing instance dengan `docker logs`. Permintaan dari browser ternyata terbagi ke kedua instance, jadi nginx meneruskan request ke salah satu instance. Setelah selesai, semua container dimatikan dengan `dcd`.

**yang harus di ingat yah itu:** fault tolerance membuat sistem tetap berjalan walaupun ada komponen yang gagal, dan dicapai lewat redundansi, load balancing, checkpointing, heartbeat, serta consensus. Retry dan circuit breaker melindungi aplikasi dari kegagalan komponen luar, sedangkan nginx sebagai load balancer membagi request ke beberapa instance sehingga tidak ada single point of failure.
