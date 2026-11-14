---
Pertemuan 06: Distributed File System, HDFS, dan Cloud Storage"
mata_kuliah: "Sistem Terdistribusi dan Terdesentralisasi"
kode: FIF25002T
nama: "Usat Jalung"
nim: 255410017
dosen: "Dr. Bambang Purnomosidi D. P."
from: tambahakan teks yang kurang dan buat dalam bentuk format README.md
rakuman  di buat : awalan di buat manual laul di lenkapi dengan ia dan di bantu membuat format README.md
---
# File System

File system (FS) adalah metode yang dipakai sistem operasi untuk mengelola dan mengatur file pada media penyimpan seperti harddisk, flashdisk, dan SSD. Contohnya ext4, XFS, btrfs, dan ReiserFS di Linux, FAT16, FAT32, dan NTFS di DOS/Windows, serta APFS di Mac.

Sebuah FS diakses lewat driver file system yang dikelola kernel. Kernel menyediakan virtual file system yang dipakai aplikasi di user space. Alurnya adalah aplikasi, lalu system call interface, lalu virtual FS, lalu driver kernel, dan terakhir hardware atau media penyimpan.

FS berbeda dengan hirarki FS. Hirarki FS adalah susunan file dan direktori dalam sebuah sistem operasi, misalnya `/bin`, `/etc`, `/home`, dan `/usr` pada Linux yang diatur oleh Filesystem Hierarchy Standard. Pengguna biasanya hanya melihat hirarkinya dan tidak peduli FS apa yang ada di belakangnya.

# Distributed File System (DFS)

DFS tidak sekadar FS yang tersebar di lebih dari satu mesin. Secara konsep, DFS adalah abstraksi terkoordinasi dari sekumpulan media penyimpan yang menutupi heterogenitas, kegagalan, dan konkurensi, sehingga menjadi jaringan penyimpanan yang terintegrasi, andal, dan berkinerja tinggi. Seperti sistem terdistribusi lainnya, medianya terdiri dari banyak, tetapi bagi pengguna terlihat sebagai satu media penyimpan.

## Perbedaan DAS, NAS, SAN, dan DFS

Direct-Attached Storage (DAS) adalah penyimpanan yang terpasang langsung di komputer, misalnya hasil perintah `df -h` yang menampilkan disk NVMe lokal. Network-Attached Storage (NAS) adalah penyimpanan bersama lewat jaringan biasa, berbasis file system, dan lebih mudah dikelola. Storage Area Network (SAN) adalah penyimpanan bersama lewat jaringan khusus, berbentuk block storage, cepat, tetapi mahal. DFS sendiri adalah abstraksi di level software yang mengelola data dalam bentuk file dan direktori yang dapat disebar ke banyak node.

SAN menyajikan penyimpanan sebagai block device yang disambungkan langsung ke server dan diformat dengan FS sendiri. SAN secara konsep bisa dipakai sebagai storage untuk DFS, tetapi tidak disarankan. Penggunaan umumnya sebagai backend database skala besar, editing video dan audio, serta virtualisasi skala besar. DFS lebih condong memakai DAS di setiap node.

## DFS yang Umum

DFS yang umum dikenal antara lain Google File System, Colossus (pengganti Google File System yang dipakai di YouTube, Gmail, dan Google Docs), CephFS, Gluster, HDFS, dan Lustre.

## Cara Kerja DFS

Pengguna dan aplikasi mengakses data lewat satu global namespace. Di bawah namespace itu ada DFS yang mengumpulkan beberapa namespace dari sumber berbeda, seperti NAS di data center, Linux NFS dan Windows SMB di lokasi remote, serta public cloud. Semuanya terlihat sebagai satu hirarki.

# Studi Kasus: HDFS

HDFS (Hadoop Distributed File System) adalah FS open source, scalable, terdistribusi, dan fault-tolerant untuk menyimpan dataset besar di klaster commodity hardware. Commodity hardware adalah perangkat keras yang diproduksi massal, murah, dan mudah dipertukarkan. HDFS adalah komponen utama Apache Hadoop. File dipecah menjadi blok-blok, disebar ke beberapa server, dan direplikasi untuk mencapai high availability. Apache Hadoop 3.5.0 dikompilasi dengan JDK 17 dan berjalan di Java 17 untuk server, serta Java 17 atau 21 untuk client.

## Arsitektur HDFS

Arsitektur HDFS terdiri dari satu master dan banyak slave. Master berisi NameNode yang menyimpan metadata, yaitu nama file, jumlah replika, dan lokasi blok. Di sisi YARN ada Resource Manager. Setiap slave berisi DataNode yang menyimpan blok data, dan Node Manager yang menjalankan proses Map dan Reduce.

Saat client membaca atau menulis, client lebih dulu melakukan operasi metadata ke NameNode. NameNode lalu mengarahkan operasi blok ke DataNode. Data ditulis ke DataNode dan direplikasi ke DataNode lain, termasuk yang berada di rack berbeda, supaya data tetap aman bila satu rack atau node bermasalah.

## Praktik Menggunakan HDFS

Persyaratan software adalah JDK 17 atau 21 (gunakan 17 bila akan mengompilasi Hadoop), Apache Hadoop 3.5.0, pdsh, ssh, dan sshd yang sudah berjalan. Langkahnya sebagai berikut.

1. **Unduh Hadoop** dari halaman releases Apache Hadoop (`hadoop.apache.org/releases.html`).
2. **Instalasi.** Ekstrak file `hadoop-3.5.0.tar.gz`, buat symbolic link `hadoop` ke `hadoop-3.5.0`, lalu atur PATH agar mencakup `$HADOOP_HOME/bin` dan `$HADOOP_HOME/sbin`. `$HADOOP_HOME` di sini adalah `$HOME/software/storage/hadoop`. Jalankan `source` pada file env sekali di tiap terminal, lalu cek dengan perintah `hadoop`.
3. **Instalasi pdsh.** Di Artix Linux pdsh dikompilasi dari AUR memakai trizen. Pastikan Rcmd type adalah ssh (atau atur `PDSH_RCMD_TYPE=ssh`). Atur juga agar ssh ke localhost tidak minta password, dengan `ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa`, menambahkan kunci publik ke `authorized_keys`, dan `chmod 0600` pada file itu.
4. **Konfigurasi.** `hdfs-site.xml` mengatur `dfs.replication` bernilai 1 karena hanya satu node. `core-site.xml` mengatur `fs.defaultFS` ke `hdfs://localhost:9000`. `mapred-site.xml` mengatur `mapreduce.framework.name` ke yarn. `yarn-site.xml` mengatur aux-services `mapreduce_shuffle` dan batas memori.
5. **Format NameNode** dengan `hdfs namenode -format`. Ini hanya dilakukan sekali, karena kalau diulang akan menghapus file yang sudah diproses di HDFS.
6. **Atur JAVA_HOME** di `$HADOOP_HOME/etc/hadoop/hadoop-env.sh` sesuai lokasi JDK di komputer.
7. **Jalankan daemon** dengan `start-dfs.sh`. Yang aktif adalah namenode, datanode, dan secondary namenode. Pemantauan bisa lewat web di `http://localhost:9870`.
8. **Selesai.** Coba operasi file, misalnya `hdfs dfs -ls /` dan `hdfs dfs -mkdir -p /user/nama`. Untuk menghentikan HDFS pakai `stop-dfs.sh`.

# Cloud Storage

Cloud storage adalah layanan online yang memungkinkan pengguna menyimpan dan mengakses data di server jarak jauh lewat internet, bukan di harddisk lokal. Penyimpanan fisiknya bisa terdiri dari banyak server di beberapa lokasi, dan lingkungan fisiknya dimiliki serta dikelola penyedia cloud. Contohnya Amazon S3, Azure Blob Storage, dan Google Cloud Storage.

## Relasi Cloud Storage dengan DFS

Cloud storage adalah tempat menyimpan data, sedangkan pengorganisasiannya dilakukan oleh DFS. Contohnya S3 bisa dipakai HDFS: S3 sebagai cloud storage dan HDFS yang melakukan operasi FS. Ini diatur di `core-site.xml` lewat properti seperti `fs.s3a.access.key`, `fs.s3a.secret.key`, dan `fs.s3a.endpoint`.

## Tipe Cloud Storage

Ada tiga tipe. Block storage memakai transport FC atau iSCSI dengan antarmuka Direct Attached atau SAN, latensinya rendah dan cocok untuk data terstruktur. File storage memakai TCP/IP dengan antarmuka NFS dan SMB, cocok untuk file sharing dan global file locking. Object storage memakai TCP/IP dengan antarmuka HTTP dan REST, mudah di-scale tanpa batas dan bisa diakses lewat LAN maupun WAN.

## Studi Kasus Cloud Storage

Beberapa contoh yang ditampilkan di slide adalah sebagai berikut.

- **Ceph:** sistem penyimpanan terdistribusi open source.
- **CubeFS:** penyimpanan cloud-native open source untuk data tak terstruktur yang mendukung protokol S3, HDFS, dan POSIX. Arsitekturnya terdiri dari master, metadata subsystem (meta node), data subsystem (data node dan blob node dengan erasure code), serta object subsystem.
- **RustFS:** object storage yang diposisikan sebagai pengganti MinIO dan S3.
- **PGSTY SILO:** fork MinIO yang kompatibel dengan S3 dan dikelola oleh PGSTY.
- **Rook:** penyimpanan cloud-native open source untuk Kubernetes yang mengelola file, block, dan object storage.

**yang harus di ingat yah itu:** DFS membuat banyak media penyimpan terlihat sebagai satu FS, dengan data dipecah menjadi blok dan direplikasi ke banyak node. HDFS memisahkan metadata di NameNode dari blok data di DataNode. Cloud storage adalah tempat menyimpan data, dan DFS yang mengatur cara data itu disimpan dan diakses.
