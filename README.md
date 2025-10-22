## Deskripsi Proyek dan Tugas
Modul ini berdurasi 3 jam. Perusahaan Speaks adalah sebuah perusahaan yang menyediakan layanan survei dan pemungutan suara (voting). Aplikasi saat ini sudah dikontainerisasi (containerised) dan berjalan pada Elastic Container Service (ECS).
Perusahaan ini ingin melangkah lebih jauh dengan mengadopsi metodologi DevOps.
Semua lingkungan aplikasi milik perusahaan Speaks berjalan di atas Amazon Web Services (AWS).
Mereka menginginkan kamu sebagai DevOps Engineer untuk membangun arsitektur Proof of Concept (PoC) untuk arsitektur mereka.

## Tugas
1. Pastikan pengguna dapat login ke AWS Academy Console.
2. Baca dokumentasi dengan saksama (tercantum di bawah).
3. Lihat arsitektur di bagian **Architecture**.
4. Baca setiap item di bagian **Technical Detail** dengan hati-hati.
5. Baca setiap item di bagian **Application Detail** dengan hati-hati.
6. Siapkan akun GitHub.
7. Masuk ke AWS Academy Learner Lab dan Launch Lab.
8. Unduh key pair di bagian AWS Details dan catat kredensial AWS CLI.
9. Kamu tidak perlu membuat user atau role baru, cukup gunakan role yang sudah ada yaitu LabRole.
10. Siapkan VPC dengan konfigurasi subnet dan security group masing-masing.
11. Siapkan layanan pendukung awal seperti Cloud9, Load Balancing, Database, Shared Storage, dan Memory Data Store.
12. Konfigurasikan semua dependensi server sesuai dengan yang dijelaskan di bagian **Technical Details*.
13. Siapkan dan konfigurasikan Elastic Container Service (ECS) untuk lingkungan produksi. Detail konfigurasi ECS dijelaskan di bagian **Elastic Container Service – Service Details**.
14. Buat Pipeline sesuai instruksi di bagian **Deployment Process Details** dan lakukan deployment aplikasi.
15. Konfigurasikan pemantauan dan metrik aplikasi di CloudWatch.

## Detail Teknis (Technical Details)
1. Tujuan proyek ini adalah membangun otomatisasi deployment untuk aplikasi berbasis microservice.
2. Aplikasi bernama **Voting App** dan kode sumbernya ada di repositori yang sama. Kamu harus melakukan fork dari repositori GitHub sumber ke akun GitHub kamu sendiri. Baca detailnya di bagian **Application Details**.
3. Aplikasi Voting memiliki tiga layanan utama: vote service sebagai halaman pemungutan suara, result service sebagai halaman untuk menampilkan hasil pemungutan suara, dan worker service sebagai layanan yang menulis data dari Redis ke database.
4. Aplikasi akan dideploy di dua lingkungan, yaitu Development dan Production. Lingkungan Development akan menggunakan Docker Swarm Cluster, sedangkan Production akan menggunakan Elastic Container Service (ECS).
5. Aplikasi harus dideploy secara otomatis dan dipicu oleh perubahan kode di GitHub — baik untuk cluster Development maupun Production.
6. Kamu harus membuat security group tambahan sesuai dengan bagian Security Group Service Details.
7. Sistem operasi dasar (base OS) yang digunakan adalah **Amazon Linux** (https://aws.amazon.com/amazon-linux-ami/), dipilih karena dukungan industri yang luas, kestabilan, ketersediaan dukungan, dan integrasi yang baik dengan AWS.
9. Load Balancer akan mengatur dan mengelola beban aplikasi.
10. Developer dan Operator hanya boleh mengakses instance atau cluster dari Cloud9.
11. Pastikan semua resource dibuat dan dikonfigurasi di region **us-east-1**.

## Arsitektur 
![Infr Diaggram](https://github.com/Ramdan241004/Soal-lks-2-2024/blob/main/Aaa.png) 

## Detail Aplikasi (Application Details)
Aplikasi Voting App dikembangkan oleh dockersamples. Voting App terdiri dari 3 aplikasi utama dan 2 komponen pendukung. Tiga aplikasi utama: **Vote App** — dikembangkan menggunakan Python. **Result App** — dikembangkan menggunakan NodeJS. **Worker App** — dikembangkan menggunakan .NET.

![Infr Diaggram](https://github.com/Ramdan241004/Soal-lks-2-2024/blob/main/UUID%E2%80%99s.png) 

Agar dapat berjalan dengan baik, Voting App memerlukan dua komponen pendukung, yaitu: PostgreSQL (sebagai database), dan Redis (sebagai in-memory data store). Dalam proyek ini, Voting App akan dijalankan di dua lingkungan, yaitu: Development, Production. Untuk Development, aplikasi Voting dapat diakses melalui Load Balancer bernama lks-lb-dev pada port 80 dan 81. Sedangkan untuk Production, dapat diakses melalui Load Balancer bernama lks-lb-prod dengan port yang sama (80 dan 81).

## Siklus Hidup Deployment (Deployment LifeCycle)

![Infr Diaggram](https://github.com/Ramdan241004/Soal-lks-2-2024/blob/main/Aaaa.png) 

Semua layanan harus dideploy secara berkelanjutan mengikuti versi dari layanan masing-masing. Seluruh layanan harus dideploy secara otomatis ke: Docker Swarm Cluster di EC2 instances untuk lingkungan Development, dan ECS (Elastic Container Service) untuk lingkungan Production. Kamu harus menggunakan GitHub Actions untuk mengotomatisasi proses deployment. Pipeline akan berjalan secara otomatis ketika ada perubahan kode di: **branch dev** → untuk Development, **branch prod** → untuk Production Repositori publik yang digunakan adalah GitHub Service, dan setiap repositori memiliki Dockerfile untuk membuat image Docker. Kamu dapat membuat Docker image menggunakan file tersebut — baca deskripsi di setiap repositori untuk memahami cara menginstal dan membuat image. Setiap image harus diunggah ke private registry dengan tag versi terbaru serta nomor versi kodemisalnya:

**ACCOUNT_ID**.dkr.ecr.us-east-1.amazonaws.com/lks-voting-image:**vote-{dev/prod}-latest**
**ACCOUNT_ID**.dkr.ecr.us-east-1.amazonaws.com/lks-voting-image:**result-{dev/prod}-latest**
**ACCOUNT_ID**.dkr.ecr.us-east-1.amazonaws.com/lks-voting-image:**worker-{dev/prod}-latest**

Kamu dapat menggunakan Elastic Container Registry (ECR) sebagai private Docker registry. Image yang telah kamu simpan akan digunakan untuk deployment ke: Docker Swarm, dan Elastic Container Service (ECS). Seluruh proses ini harus dijalankan di dalam AWS CodePipeline.

## Prasyarat (Pre-Requirement)
Sebelum kamu mulai membuat pipeline, kamu harus mengkloning repositori layanan ke dalam AWS Cloud9. Berikut adalah repositori untuk Voting App:

Voting Service: https://github.com/sulaimantok/lks-voting-app.git

Langkah-langkah: Buat Cloud9 environment dengan nama **lks-cloud9**, dan pilih **SSH** sebagai tipe koneksi. Lakukan fork repositori tersebut ke akun GitHub kamu sendiri, lalu clone sumber kode ke Cloud9. Beri nama repositorinya **lks-voting-app**. Baca panduan instalasi aplikasi di file README.md. Siapkan repository registry dengan nama **lks-voting-image**, lalu letakkan image di private repositories. Tambahkan Secrets ke dalam repositori GitHub kamu. Kredensial yang perlu ditambahkan ke repository secrets adalah:

```env
AWS_ACCESS_KEY=YOUR_AWS_ACCESS_KEY
AWS_SECRET_KEY=YOUR_AWS_SECRET_KEY
AWS_SESSION_TOKEN=YOUR_AWS_SESSION_TOKEN
AWS_SNS_ARN=YOUR_AWS_SNS_ARN
```

Referensi:
https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions-creating-secrets-for-a-repository

## Code Pipeline (GitHub Actions)
Buat dua workflow GitHub Actions dengan nama: **lks-dev-pipeline.yaml**, dan **lks-prod-pipeline.yaml**. Kamu dapat merujuk ke contoh workflow yang dapat digunakan pada folder **.github/workflow/** dalam repositori. Kamu harus membuat 3 job untuk lks-dev-pipeline dan 2 job untuk lks-prod-pipeline di dalam CodePipeline dengan rincian berikut:

1. lks-dev-pipeline
- **Build** Job ini berisi beberapa langkah: Memperbarui versi Docker pada runner, Melakukan checkout repositori, Menyiapkan kredensial AWS, Login ke ECR, Build dan push image ke ECR.
- **Pull Image on master** Job ini akan melakukan pull image terbaru pada instance control plane.
- **Pull Image on node1** Job ini akan melakukan pull image terbaru pada instance node1.
- **Pull Image on node2** Job ini akan melakukan pull image terbaru pada instance node2.
- **Deploy** Job ini berisi proses Deploy Application ke dalam Docker Swarm Cluster.

2. lks-prod-pipeline
- **Build** Langkah-langkah di dalam job ini sama seperti pada dev pipeline: Update Docker version, Checkout repository, Setup AWS credentials, Login ke ECR, Build dan push image ke ECR.
- **Deploy** Job ini akan: Memperbarui konten di dalam task definition, Melakukan deploy application ke dalam ECS Cluster, Mengirim notifikasi (SNS) apabila deployment berhasil.

Catatan: Pastikan di repositori kamu hanya ada dua file workflow ini: lks-dev-pipeline.yaml lks-prod-pipeline.yaml Hapus file workflow lain yang tidak digunakan. Semua job harus berjalan secara berurutan (sequentially). Nantinya GitHub Actions akan menampilkan visualisasi pipeline seperti pada contoh gambar (dalam dokumen asli).

## Detail Layanan (Service Details)

### Virtual Private Cloud (VPC)
Pada bagian ini, kamu memiliki tugas untuk membuat dan mengatur custom VPC dengan nama **speaks-vpc** menggunakan CIDR 192.168.10.0/24. Pastikan DNS hostnames diaktifkan, dan CIDR dibagi ke dalam tiga Availability Zone dengan detail sebagai berikut:

1. VPC harus memiliki dua public subnet dan dua private subnet.
2. Public subnet pertama bernama speaks-pubsubnet-a, berisi 64 host dengan IPv6 diaktifkan. Subnet ini harus berada di zona us-east-1a.
3. Public subnet kedua bernama speaks-pubsubnet-b, berisi 64 host setelah prefix subnet pertama. IPv6 juga harus diaktifkan dan subnet ini berada di zona us-east-1b.
4. Dua private subnet masing-masing berisi 64 host: speaks-privsubnet-a di us-east-1a speaks-privsubnet-b di us-east-1b
5. Konfigurasikan agar semua resource di public subnet dapat terhubung ke internet melalui Internet Gateway bernama speaks-igw, dan arahkan ke 0.0.0.0/0 melalui Route Table bernama speaks-rtpublic.
6. Konfigurasikan agar semua resource di private subnet dapat terhubung ke internet melalui NAT Gateway bernama speaks-ngw, dan arahkan ke 0.0.0.0/0 melalui Route Table bernama speaks-rtprivate.

### Security Groups
Security Group (SG) adalah fitur di Amazon EC2 yang digunakan untuk mengontrol lalu lintas inbound dan outbound dari instance yang terhubung dengan SG tersebut. Dalam bagian ini, kamu harus membuat beberapa Security Group dengan ketentuan berikut:

1. **lks-lb-sg** Inbound: TCP port 80 dan 81 dari 0.0.0.0/0
2. **lks-container-sg** Inbound: semua port TCP dari **lks-lb-sg**
3. **lks-docker-sg** Inbound: TCP port 22, 5000, 5001, 2376, 2377, 7946, 4789, serta UDP port 7946
4. **lks-db-sg** Inbound: Akses untuk PostgreSQL dan Redis dari **speaks-vpc**

### Docker Swarm (Cluster Development)
Docker Swarm adalah alat manajemen orkestrasi yang digunakan untuk menjalankan aplikasi berbasis Docker. Fungsinya membantu pengguna dalam membangun dan menerapkan cluster node Docker. Setiap node di dalam Docker Swarm adalah Docker daemon, dan seluruh daemon tersebut berkomunikasi menggunakan Docker API. Dalam proyek ini, Docker Swarm akan dibangun dan dijalankan di AWS EC2. Tugasmu adalah membangun aplikasi microservice di atas Docker Swarm untuk melakukan pengujian aplikasi sebelum dijalankan di produksi, dengan rincian berikut:

1. Luncurkan 3 instance EC2: 1 instance untuk control-plane, 2 instance untuk worker. Beri nama: **lks-master-node** untuk control-plane, **lks-node1** dan **lks-node2** untuk dua instance lainnya.
2. Gunakan **vockey** sebagai key pair, dan pastikan instance dapat diakses dari luar.
3. Gunakan tipe instance t2.medium untuk semua instance.
4. Beri public IP hanya pada instance control-plane.
5. Gunakan **lks-docker-sg** sebagai security group.
6. Jalankan perintah untuk menginisialisasi Docker Swarm Cluster di **lks-master-node**.

```sh
sudo yum install docker dotnet-sdk-6.0 -y
sudo usermod -aG docker ec2-user
exit
## re login

docker swarm init
### copy docker swarm join and paste in node1 and node2
```

8. Jalankan perintah untuk menambahkan worker node (**lks-node1** dan **lks-node2**) ke cluster Swarm.

```sh
sudo yum install docker dotnet-sdk-6.0 -y
sudo usermod -aG docker ec2-user
exit
## re login

docker swarm join --token <your_token> <ip-address>:2377
### comand from control plane
```

10. Daftarkan semua instance sebagai GitHub Self-Hosted Runner, dengan label: **master** → untuk control-plane, **node1** → untuk lks-node1, **node2** → untuk lks-node2.
Referensi: https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners

### Amazon Elastic Container Service (ECS)
Elastic Container Service (ECS) digunakan untuk lingkungan Production. Aplikasi Voting App harus dideploy secara otomatis ke ECS melalui CodePipeline. Kamu harus membuat ECS Cluster dengan nama lks-voting-cluster, menggunakan Fargate sebagai node instance, dan VPC yang berisi private subnet (lihat bagian VPC Service untuk detail jaringan). Kamu juga harus membuat ECS Service dan Task Definition untuk setiap komponen aplikasi Voting. Berikut detail yang diperlukan:

1. Vote Container: Host port: 80 Container port: 80
2. Result Container: Host port: 80 Container port: 80
3. Worker Container: Hapus port mapping (tidak diperlukan port publik).
4. Gunakan base image yang telah diunggah ke ECR.
5. Gunakan nama task definition berikut: lks-td-vote lks-td-result lks-td-worker
6. Tambahkan container di dalam task definition dengan nama: lks-vote-container untuk vote service lks-result-container untuk result service lks-worker-container untuk worker service
7. Aktifkan log collection.
8. Salin konten JSON dari Task Definition ke clipboard, lalu simpan di dalam folder masing-masing service di repositori, dengan nama file: task-definition.json (contoh: vote/task-definition.json).
9. Buat ECS Cluster dengan nama lks-voting-cluster.
10. Buat ECS Service untuk setiap komponen: lks-vote-service lks-result-service lks-worker-service Gunakan Launch Type = Fargate dan Application Type = Service.
11. Gunakan Load Balancer bernama lks-lb-prod dengan protokol HTTP.
12. Setiap service menggunakan path / sebagai Health Check.
13. Buat Target Group: lks-tg-vote untuk vote service lks-tg-result untuk result service
14. Buat listener port 80 → diarahkan ke vote container.
15. Buat listener port 81 → diarahkan ke result container.

### Microservices Application
Dalam bagian ini, tugasmu adalah membangun dan men-deploy aplikasi microservices di atas Docker Swarm (Development) dan Amazon ECS (Production). Aplikasi ini adalah Vote Application yang terhubung ke database (PostgreSQL) dan Redis. Ikuti langkah-langkah berikut:

1. Buka AWS Cloud9 dan ikuti panduan di repositori: https://github.com/sulaimantok/lks-voting-app.git
2. Baca file README.md dengan teliti sebelum memulai.
3. Aplikasi akan dideploy secara otomatis setiap kali ada perubahan pada repositori GitHub: branch dev → untuk Development branch prod → untuk Production
4. Aplikasi dapat diakses melalui: Load Balancer lks-lb-dev pada port 80 dan 81 untuk Development, Load Balancer lks-lb-prod pada port 80 dan 81 untuk Production.
5. Pastikan semua objek berhasil dideploy dengan benar.

### Amazon RDS (Relational Database Service)
Amazon RDS (Relational Database Service) adalah layanan web yang mempermudah pembuatan, pengoperasian, dan skala database relasional di cloud AWS. Dalam bagian ini, kamu memiliki tugas untuk menyiapkan dan mengonfigurasi database di Amazon RDS dengan detail berikut:

1. Buat cluster database menggunakan private subnet di dua Availability Zone.
2. Gunakan PostgreSQL sebagai database engine dengan versi terbaru.
3. Untuk template, gunakan Free tier.
4. Gunakan identifier dan kredensial berikut: DB Instance Identifier: lks-rds Username: admin Password: LKSNCC2024
5. Gunakan VPC speaks-vpc dan pastikan Public Access = No.
6. Gunakan Security Group lks-db-sg. Matikan Performance Insight.
7. Buat Custom Parameter Group dengan detail: Name: custompostgres Engine Type: PostgreSQL Parameter Group Family: versi PostgreSQL terbaru
8. Edit parameter berikut: **rds.force_ssl** → ubah nilainya menjadi 0 **password_encryption** → ubah nilainya menjadi md5
9. Ubah konfigurasi parameter group milik lks-rds untuk menggunakan custompostgres.

### Amazon ElastiCache
Amazon ElastiCache adalah layanan in-memory caching yang dikelola sepenuhnya oleh AWS. Layanan ini memungkinkan kamu untuk men-deploy, mengoperasikan, dan menskalakan cache terdistribusi di cloud, yang dapat meningkatkan kinerja dan skalabilitas aplikasi dengan mengurangi beban database backend. Dalam bagian ini, kamu memiliki tugas untuk membuat sistem cache Redis di dalam Amazon ElastiCache untuk menyimpan data cache dari aplikasi yang berjalan di Amazon ECS, dengan detail berikut:

1. Buat Redis OSS Cache dengan memilih Design your own cache pada opsi Deployment Option.
2. Pilih Cluster Cache sebagai Creation Method.
3. Pilih Cluster Mode: Disabled.
4. Beri nama cluster: **lks-redis**.
5. Nonaktifkan Multi-AZ, dan aktifkan Auto Failover.
6. Gunakan versi engine terbaru, dengan konfigurasi: Node Type: cache.m7g.large Number of Replicas: 2
7. Buat Subnet Group baru dengan nama redis-sb, pilih VPC kamu, dan gunakan dua public subnet.
8. Nonaktifkan Automatic Backup dan Auto Upgrade Minor Versions.

### Amazon Simple Notification Service (SNS)
Amazon SNS (Simple Notification Service) adalah layanan terkelola sepenuhnya yang memungkinkan pengiriman pesan menggunakan model publish/subscribe ke banyak penerima sekaligus. SNS mendukung berbagai protokol, seperti HTTP/S, SMS, dan Email. Dalam bagian ini, kamu harus mengintegrasikan SNS dengan GitHub Actions, agar setiap kali ada publish ke topik SNS, para subscriber menerima notifikasi. Langkah-langkahnya sebagai berikut:

1. Buat Topic dengan nama lks-vote-topic dan pilih Standard Type.
2. Buat Subscription menggunakan ARN dari topic tersebut: Protocol: Email Endpoint: personallabs4@gmail.com Setelah itu, konfirmasi ke juri jika status subscription kamu masih Pending Confirmation.
3. Publikasikan pesan berikut ke dalam topik:
**Subject**: Hello from <Provinsi>!
**Message Body** : New version of The Voting App from <Provinsi> is Released !!
