# Deskripsi Proyek dan Tugas
Modul ini berdurasi 3 jam.
Speaks Company adalah sebuah perusahaan yang membagikan survei dan pemungutan suara. Aplikasi saat ini sudah dikontainerisasi dan berjalan di Elastic Container Service (ECS). Perusahaan ini ingin melangkah lebih jauh dengan mengadopsi metodologi DevOps. Semua lingkungan aplikasi Speaks Company berjalan di Amazon Web Services (AWS). Mereka menginginkan Anda sebagai DevOps Engineer untuk membangun arsitektur Proof of Concept (PoC) untuk arsitektur tersebut.

# Tugas
1. Pastikan pengguna dapat login ke konsol AWS Academy.
2. Baca dokumentasi dengan saksama (tercantum di bawah).
3. Lihat arsitektur di bagian **arsitektur**.
4. Baca setiap item di bagian **detail teknis** dengan cermat.
5. Baca setiap item di bagian **detail aplikasi** dengan cermat.
6. Siapkan akun GitHub.
7. Buka AWS Academy Learner Lab dan Luncurkan Lab.
8. Unduh key pair di AWS Details dan catat kredensial AWS CLI.
9. Anda tidak perlu membuat pengguna atau role baru, cukup gunakan role yang sudah ada, yaitu LabRole.
10. Siapkan VPC dengan konfigurasi subnet dan security group yang sesuai.
11. Siapkan layanan pendukung awal seperti Cloud9, load balancing, database, shared storage, dan memory data store.
12. Konfigurasikan semua dependensi server seperti yang dijelaskan di **detail teknis**.
13. Siapkan dan konfigurasikan platform Elastic Container Service untuk lingkungan produksi. Detail konfigurasi ECS terdapat di bagian **Elastic Container Service – Service details**.
14. Buat pipeline sesuai instruksi yang diberikan di bagian **Deployment Process Details** dan lakukan deployment aplikasi.
15. Konfigurasikan pemantauan dan metrik aplikasi yang diperlukan di CloudWatch.

# Detail Teknis
1. Tujuan dari proyek ini adalah membangun otomatisasi deployment untuk aplikasi microservice.
2. Aplikasi yang digunakan bernama **Voting App**, dan source code-nya ada di repositori yang sama; Anda harus melakukan fork dari sumber GitHub ke akun GitHub Anda. Bacalah **detail bagian Aplikasi**.
3. Voting App memiliki tiga layanan utama, yaitu: vote service sebagai halaman pemungutan suara, result service sebagai halaman yang menampilkan hasil voting, worker service sebagai layanan yang menulis data dari Redis ke database.
4. Aplikasi akan dideploy pada dua lingkungan: Development dan Production. Lingkungan Development akan menggunakan Docker Swarm Cluster. Lingkungan Production akan menggunakan Elastic Container Service (ECS).
5. Aplikasi harus ter-deploy secara otomatis dan dipicu oleh perubahan di GitHub — baik pada cluster development maupun production.
6. Anda harus membuat Security Group tambahan sesuai dengan detail di bagian Security Group Service details.
7. Sistem operasi dasar yang dipilih adalah **Amazon Linux** (https://aws.amazon.com/amazon-linux-ami/). Distribusi ini dipilih karena dukungan industri yang luas, stabilitas, ketersediaan dukungan, dan integrasi yang sangat baik dengan AWS.
8. Load balancer akan mengatur dan membagi beban aplikasi.
9. Developer dan operator hanya dapat mengakses instance atau cluster melalui Cloud9.
10. Pastikan semua sumber daya dibuat dan dikonfigurasi di region **us-east-1**.

# Detail Aplikasi
Voting Apps dikembangkan oleh dockersamples. Voting Apps terdiri dari 3 aplikasi utama dan 2 komponen pendukung. Tiga aplikasi utama adalah: **vote app** dikembangkan dengan Python, **result app** dikembangkan dengan NodeJS, **worker app** dikembangkan dengan .NET. Agar berjalan dengan baik, Voting Apps membutuhkan dua komponen pendukung, yaitu PostgreSQL dan Redis. Dalam proyek ini, Voting Apps akan berjalan di dua lingkungan: Development dan Production. Aplikasi di lingkungan development dapat diakses melalui load balancer bernama **lks-lb-dev** dengan port 80 dan 81. Aplikasi di lingkungan production dapat diakses melalui load balancer bernama **lks-lb-prod** dengan port 80 dan 81 yang terbuka.

# Siklus Hidup Deployment
Semua layanan harus terus menerus dideploy sesuai dengan versi layanan. Seluruh layanan harus secara otomatis dideploy ke cluster Docker Swarm di EC2 untuk development, dan ke ECS untuk Production, menggunakan GitHub Actions untuk otomatisasi proses deployment. Pipeline akan berjalan secara otomatis ketika ada perubahan kode di: **branch dev** untuk environment Development **branch prod** untuk environment Production Repository publik yang digunakan adalah GitHub, yang memiliki Dockerfile di dalamnya. Anda dapat membuat image Docker menggunakan Dockerfile yang disediakan — bacalah deskripsi pada setiap repository untuk mengetahui cara instalasi dan pembuatan image.
Setiap image harus diunggah ke Private Registry dengan tag versi terbaru dan nomor versi, contoh:

**ACCOUNT_ID**.dkr.ecr.us-east-1.amazonaws.com/lks-voting-image:**vote-{dev/prod}-latest** 
**ACCOUNT_ID**.dkr.ecr.us-east-1.amazonaws.com/lks-voting-image:**result-{dev/prod}-latest**
**ACCOUNT_ID**.dkr.ecr.us-east-1.amazonaws.com/lks-voting-image:**worker-{dev/prod}-latest**

Gunakan Elastic Container Registry (ECR) sebagai private Docker registry. Image yang telah disimpan akan dideploy ke Docker Swarm dan ECS. Semua proses ini harus dijalankan melalui Code Pipeline.

# Prasyarat
Sebelum membuat pipeline, Anda harus mengkloning repository layanan ke Cloud9 Anda.
Berikut repository untuk

Voting App: https://github.com/sulaimantok/lks-voting-app.git

Buat Cloud9 sebagai lingkungan pengkodean dengan nama **lks-cloud9** dan jenis koneksi **SSH**. Lalu, lakukan fork repository ke akun GitHub Anda dan clone source code tersebut di sana. Gunakan nama repository **lks-voting-app**. Bacalah instruksi instalasi aplikasi di README.md. Anda perlu menyiapkan repository registry bernama **lks-voting-image** di private registry.

Tambahkan secrets pada repository GitHub Anda. Kredensial yang harus ditambahkan adalah:

```env
AWS_ACCESS_KEY=YOUR_AWS_ACCESS_KEY
AWS_SECRET_KEY=YOUR_AWS_SECRET_KEY
AWS_SESSION_TOKEN=YOUR_AWS_SESSION_TOKEN
AWS_SNS_ARN=YOUR_AWS_SNS_ARN
```

Referensi: https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions-creating-secrets-for-a-repository

# Code Pipeline (GitHub Actions)
Buat GitHub Actions workflow dengan nama: **lks-dev-pipeline.yaml** dan **lks-prod-pipeline.yaml** Gunakan referensi di folder **.github/workflow/** sebagai contoh workflow. Anda harus membuat 3 job untuk lks-dev-pipeline dan 2 job untuk lks-prod-pipeline seperti berikut:

1. lks-dev-pipeline
- **Build**: berisi langkah-langkah memperbarui versi Docker pada runner, checkout repository, setup kredensial AWS, login ke ECR, build dan push image ke ECR.
- **Pull Image on master**: menarik image terbaru pada instance control plane.
- **Pull Image on node1**: menarik image terbaru pada instance node1.
- **Pull Image on node2**: menarik image terbaru pada instance node2.
- **Deploy**: melakukan deployment aplikasi ke cluster Docker Swarm.

2. lks-prod-pipeline
- **Build**: memperbarui Docker, checkout repository, setup kredensial AWS, login ke ECR, build, dan push image ke ECR.
- **Deploy**: memperbarui konten di task definition dan mendeply aplikasi ke ECS Cluster, serta mengirim notifikasi ke SNS jika deployment berhasil.

Catatan: Pastikan hanya terdapat file lks-dev-pipeline.yaml dan lks-prod-pipeline.yaml di repository Anda. Hapus file workflow lain yang tidak digunakan. Semua job harus dijalankan secara berurutan.

# Detail Layanan
## Virtual Private Cloud (VPC)
Anda harus membuat dan mengatur custom VPC dengan nama **speaks-vpc** dan CIDR 192.168.10.0/24. Pastikan DNS hostnames diaktifkan dan VPC memiliki tiga availability zones dengan rincian:
1. Dua public subnet dan dua private subnet.
2. speaks-pubsubnet-a (64 host, IPv6 aktif, zona us-east-1a).
3. speaks-pubsubnet-b (64 host setelah prefix subnet-a, IPv6 aktif, zona us-east-1b).
4. Dua private subnet masing-masing 64 host: speaks-privsubnet-a di us-east-1a dan speaks-privsubnet-b di us-east-1b
5. Semua resource di public subnet harus bisa mengakses internet melalui Internet Gateway (speaks-igw) dan diarahkan ke 0.0.0.0/0 melalui speaks-rtpublic route table.
6. Semua resource di private subnet harus bisa mengakses internet melalui NAT Gateway (speaks-ngw) dan diarahkan ke 0.0.0.0/0 melalui speaks-rtprivate route table.

# Security Groups
Anda harus membuat beberapa security group berikut:
1. **lks-lb-sg**: inbound port 80 dan 81 TCP dari 0.0.0.0/0
2. **lks-container-sg**: inbound semua TCP dari **lks-lb-sg**
3. **lks-docker-sg**: inbound port 22, 5000, 5001, 2376, 2377, 7946, 4789 TCP dan 7946 UDP
4. **lks-db-sg**: inbound PostgreSQL dan Redis dari speaks-vpc

# Docker Swarm (Cluster Development)
Anda harus membuat Docker Swarm Cluster di EC2 dengan rincian:
1. Luncurkan 3 instance EC2: 1 control-plane (**lks-master-node**) 2 worker (**lks-node1**, **lks-node2**)
2. Gunakan key pair vockey.
3. Gunakan tipe instance t2.medium untuk semua instance.
4. Hanya control plane yang diberi public IP.
5. Gunakan security group **lks-docker-sg**.
6. Deploy cluster Docker Swarm di **lks-master-node**

```sh
sudo yum install docker dotnet-sdk-6.0 -y
sudo user mode -aG docker ec2-user
exit
## region
docker swarm ini

### copy docker swarm join and paste in node 1 and node 2
```

### command from. control panel
8. Deploy cluster Docker Swarm di **lks-node1**, **lks-node2**.

```sh
sudo yum install docker dotnet-sdk-6.0 -y
sudo user mode -aG docker ec2-user
exit
## region
docker Swarm join --token <token> <ip-address>:2377

### command from control panel
```

10. Daftarkan semua instance sebagai GitHub self-hosted runner: Label **master** untuk control plane, Label **node1** untuk lks-node1, dan Label **node2** untuk lks-node2.

Referensi:
https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners

# Amazon Elastic Container Service (ECS)
Gunakan ECS untuk Production.
Voting App harus dideploy otomatis ke ECS melalui pipeline.
1. Buat ECS Cluster bernama lks-voting-cluster menggunakan Fargate.
2. Gunakan VPC dan subnet privat.
3. Task Definition:
lks-td-vote, lks-td-result, lks-td-worker
Container:
lks-vote-container (port 80)
lks-result-container (port 80)
lks-worker-container (tanpa port mapping)
Gunakan image dari ECR.
Aktifkan log collection.
Salin JSON task definition dan simpan di masing-masing foldyanan dengan nama task-definition.json.
4. Buat ECS Service:
lks-vote-service, lks-result-service, lks-worker-service
Launch type: Application
Load balancer: lks-lb-prod, protokol HTTP
Health check path: /
5. Target group:
lks-tg-vote untuk port 80
lks-tg-result untuk port 81

# Aplikasi Microservices
1. Buka AWS Cloud9 dan ikuti panduan di repository:
https://github.com/sulaimantok/lks-voting-app.git
2. Baca README dengan seksama.
3. Aplikasi akan otomatis dideploy jika ada perubahan di GitHub repo:
branch dev untuk Development
branch prod untuk Production
4. Akses aplikasi melalui:
lks-lb-dev (port 80 dan 81) untuk Development
lks-lb-prod (port 80 dan 81) untuk Production
5. Pastikan semua objek berhasil dideploy.

# Amazon RDS
1. Buat database cluster di private subnets dua AZ.
2. Gunakan PostgreSQL versi terbaru.
3. Template: Free Tier.
4. Instance identifier: lks-rds
Username: admin
Password: LKSNCC2024
5. Gunakan speaks-vpc, public access No.
6. Security group: lks-db-sg, matikan Performance Insights.
7. Buat parameter group:
Nama: custompostgres
Engine: PostgreSQL
Versi terbaru
8. Ubah parameter
rds.force_ssl = 0
password_encryption = md5
9. Modifikasi parameter group lks-rds agar menggunakan custompostgres.

# Amazon ElastiCache
1. Buat Redis OSS cache (Design your own).
2. Cluster cache sebagai metode pembuatan.
3. Cluster Mode Disabled.
4. Nama: lks-redis.
5. Nonaktifkan Multi-AZ, aktifkan Auto Failover.
6. Engine terbaru, cache.m7g.large, jumlah replica 2.
7. Buat subnet group redis-sb, pilih VPC dan 2 public subnet.
8. Nonaktifkan automatic backup dan auto minor upgrade.

# Amazon Simple Notification Service (SNS)
1. Buat topic bernama lks-vote-topic dengan tipe Standard.
2. Buat subscription dengan ARN topic di atas, Protocol: Email Endpoint: personallabs4@gmail.com Konfirmasi ke juri jika status subscription masih pending.
3. Publikasikan pesan berikut ke topic: **Subject**: Hello from <Provinsi>! **Message body**: New version The Voting App from <Provinsi> is Released!!
