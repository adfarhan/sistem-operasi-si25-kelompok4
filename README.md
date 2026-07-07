# Laporan Tugas Kelompok: Instalasi Debian 13 Headless Web Server
Mata Kuliah: Sistem Operasi (SI-25)
Program Studi: Sistem Informasi, Universitas Galuh

## 👥 Anggota Kelompok 4 (Kelas SI-2025A)
1. Yayan Farhan - 7020250001
2. Taufiqurrahman Hakim - 7020250020
3. Ade Farhan Gunawan - 7020250015
4. Dewi Clarisa Supardi - 7020250045
5. Inayah Subekti - 7020250006
6. Putri Zahra - 7020250007

## 🎯 Spesifikasi Lingkungan Server
* **Hypervisor:** VMware Workstation
* **ISO:** `debian-13.4.0-amd64-DVD-1.iso`
* **Sistem Operasi:** Debian 13 (Bookworm) - Headless (CLI / Tanpa GUI, tidak ada Desktop Environment)
* **RAM:** 2048 MB (untuk perangkat host dengan RAM < 8 GB) / 4096 MB (untuk perangkat host dengan RAM > 8 GB)
* **Storage:** 20 GB
* **Network:** NAT (default)
* **Partisi:** Guided - use entire disk, All files in one partition
* **IP Address VM (Guest):** `192.168.x.x` (Gunakan perintah `ip a` pada antarmuka ens33 untuk melihat IP VM Anda)
* **Port Forwarding:** Host Port `8080` -> VM Port `80` (HTTP)

## 🛠️ Langkah-Langkah & Dokumentasi Praktikum

### Langkah 1 - Instalasi Sistem Dasar

* Boot dari ISO *netinstall*, lalu pada tahap Software Selection hanya mencentang **Standard system utilities** dan **SSH server**, tanpa memilih Desktop Environment apa pun.

  **Penjelasan:** Tahap ini menentukan sifat "headless" dari server. **SSH server** dipilih agar server bisa diakses dan dikelola dari jarak jauh melalui terminal (tanpa perlu membuka konsol VMware terus-menerus). **Standard system utilities** menyediakan tools dasar Linux (`nano`, `less`, `wget`, dll). Semua paket Desktop Environment (GNOME, dsb) sengaja tidak dicentang supaya sistem tetap ringan dan murni berjalan lewat CLI, sesuai kebutuhan server produksi pada umumnya.

* Mengatur hostname: `server-SI2025[A/B]` (disesuaikan dengan kelas kelompok).

  **Penjelasan:** Hostname adalah nama identitas server di jaringan. Nama ini akan tampil di prompt terminal (`user@server-SI2025A:~$`) dan memudahkan identifikasi server bila ada banyak VM/server lain yang berjalan bersamaan.

* Melakukan partisi dengan skema *Guided - use entire disk* → *All files in one partition*, serta memasang bootloader GRUB ke `/dev/sda`.

  **Penjelasan:** Skema *Guided - use entire disk* membuat installer otomatis membagi ruang disk, lebih sederhana dibanding partisi manual. *All files in one partition* berarti seluruh direktori sistem (`/`, `/home`, `/var`, dsb) digabung jadi satu partisi saja — cukup untuk skala praktikum. **GRUB** sendiri adalah bootloader, yaitu program pertama yang dijalankan saat VM dinyalakan untuk memuat kernel Linux; tanpa GRUB terpasang di `/dev/sda`, VM tidak akan bisa booting ke sistem yang baru diinstal.

* *[Tambahkan screenshot proses menu software selection di bawah ini]*
  ![Software Selection](images/01-software-selection.png)

* Setelah reboot, login sebagai `root`, membuat user biasa, lalu menambahkannya ke grup sudo:
  ```bash
  apt update && apt upgrade -y
  apt install sudo -y
  usermod -aG sudo kelompok4
  # logout & login kembali
  ```

  **Penjelasan tiap perintah:**
  - `apt update` menyinkronkan daftar paket terbaru dari mirror repository (belum menginstal apa pun, hanya memperbarui "katalog"), sedangkan `apt upgrade -y` meng-upgrade seluruh paket yang sudah terpasang ke versi terbarunya; `-y` otomatis menjawab "yes" pada setiap konfirmasi.
  - `apt install sudo -y` memasang paket `sudo`, yang dibutuhkan agar user biasa nantinya bisa menjalankan perintah administratif tanpa harus login penuh sebagai `root` — praktik ini lebih aman karena akses root hanya dipinjam sementara per-perintah.
  - `usermod -aG sudo kelompok4` menambahkan user `kelompok4` ke grup `sudo` (`-a` = *append*, tidak menghapus keanggotaan grup lain; `-G` = grup tambahan). Anggota grup ini diizinkan menjalankan `sudo` sesuai aturan di `/etc/sudoers`.
  - Logout & login ulang diperlukan karena keanggotaan grup baru hanya terbaca pada sesi login baru, bukan sesi yang sedang berjalan.

* *[Tambahkan screenshot tampilan login terminal Debian pertama kali di bawah ini]*
  ![Login Terminal](images/02-debian-login.png)

### Langkah 2 - Tools Dasar & Web Server Nginx

* Menginstal paket pendukung:
  ```bash
  sudo apt install net-tools curl git -y
  ```

  **Penjelasan tiap paket:**
  - `net-tools` — menyediakan perintah jaringan klasik seperti `ifconfig`, `netstat` untuk membantu diagnosa jaringan.
  - `curl` — dipakai untuk mengirim request HTTP langsung dari terminal, berguna untuk menguji web server tanpa perlu browser (dipakai di langkah berikutnya).
  - `git` — sistem kontrol versi, berguna bila nanti perlu mengambil/mengelola kode dari repository.
  
  Semua perintah diawali `sudo` karena user biasa perlu meminjam hak akses administratif untuk menginstal paket ke sistem.

* Menginstal, menjalankan, dan mengaktifkan Nginx agar berjalan otomatis saat booting:
  ```bash
  sudo apt install nginx -y
  sudo systemctl start nginx
  sudo systemctl enable nginx
  sudo systemctl status nginx --no-pager
  curl http://localhost
  ```

  **Penjelasan tiap perintah:**
  - `sudo apt install nginx -y` memasang perangkat lunak web server **Nginx**, yang bertugas melayani (menyajikan) halaman web ke browser yang mengaksesnya.
  - `sudo systemctl start nginx` menjalankan service Nginx untuk sesi saat ini.
  - `sudo systemctl enable nginx` membuat Nginx otomatis menyala setiap kali sistem melakukan boot, tanpa perlu dijalankan manual tiap kali — penting untuk server yang harus selalu online.
  - `sudo systemctl status nginx --no-pager` menampilkan status terkini service (`active (running)`, `inactive`, atau `failed`) beserta beberapa baris log terakhir untuk membantu diagnosis jika ada masalah.
  - `curl http://localhost` mengirim request HTTP ke web server yang berjalan di VM itu sendiri (`localhost`). Ini adalah cara cepat memverifikasi Nginx sudah aktif dan menyajikan halaman, sebelum diuji lagi lewat browser di komputer host.

* Halaman *Welcome to nginx!* berhasil tampil melalui `curl`, menandakan service Nginx berjalan dengan baik.

  **Penjelasan:** Output HTML default ini adalah halaman bawaan Nginx yang tersimpan di `/var/www/html/index.html`. Keberhasilan menampilkannya membuktikan seluruh rantai proses — instalasi paket, aktivasi service, hingga konfigurasi default — berjalan tanpa masalah.

* *[Tambahkan screenshot status active running dari Nginx]*
  ![Nginx Service Status](images/03-nginx-status.png)

### Langkah 3 - Halaman HTML Statis (Profil Kelompok)

* Mengubah dokumen default Nginx pada `/var/www/html/index.html` dengan HTML profil anggota Kelompok 4:
  ```bash
  sudo nano /var/www/html/index.html
  ```

  **Penjelasan:** `/var/www/html/` adalah direktori root default Nginx — seluruh isi file di sini yang akan disajikan ke browser saat diakses. `nano` adalah text editor sederhana berbasis terminal, cocok dipakai pada server headless karena tidak membutuhkan tampilan grafis seperti Notepad. Perintah diawali `sudo` karena file ini dimiliki oleh sistem, bukan user biasa, sehingga butuh hak akses administratif untuk mengeditnya.

* Melakukan restart web server dan verifikasi ulang setelah perubahan disimpan:
  ```bash
  sudo systemctl restart nginx
  curl http://localhost
  ```

  **Penjelasan:** `systemctl restart nginx` memuat ulang service Nginx agar perubahan konfigurasi/isi file diterapkan secara bersih (menghindari kemungkinan cache atau proses lama yang masih tertinggal). `curl http://localhost` kemudian dipakai kembali untuk memastikan isi HTML yang baru sudah benar-benar tersaji sebelum diuji dari browser di komputer host pada langkah selanjutnya.

* *[Tambahkan screenshot pengeditan index.html menggunakan nano editor]*
  ![Edit index.html](images/04-edit-html.png)

## 🌐 Port Forwarding - Akses dari Host

* Mengecek IP Address Debian (Guest) dengan perintah `ip a`, mencatat alamat IP pada antarmuka *ens33*.

  **Penjelasan:** Karena VM berjalan dalam mode **NAT**, VM memiliki IP tersendiri yang berbeda dari IP komputer host. IP inilah yang perlu dicatat agar bisa didaftarkan pada aturan port forwarding di langkah berikutnya — tanpa IP yang benar, VMware tidak tahu VM mana yang harus dituju.

* Melakukan pemetaan **Host Port 8080** ke **VM Port 80** melalui menu `Edit` → `Virtual Network Editor` → pilih **VMnet8 (NAT)** → **NAT Settings** → **Add** pada bagian Port Forwarding, dengan mengisi alamat IP Guest yang sudah dicatat.

  **Penjelasan:** Mode NAT menempatkan VM di balik "router virtual" milik VMware (`VMnet8`), sehingga komputer host tidak bisa langsung mengakses VM tanpa aturan tambahan. Aturan port forwarding ini secara sederhana berarti: *"setiap request yang masuk ke port 8080 pada komputer host, teruskan ke port 80 pada VM dengan IP tersebut"*. Port 80 dipilih karena itu adalah port standar HTTP tempat Nginx melayani permintaan web.

* *[Tambahkan screenshot pengaturan NAT Settings VMware]*
  ![NAT Settings VMware](images/05-nat-settings.png)

* Menguji akses web server Debian melalui browser di sistem operasi host pada alamat `http://localhost:8080`.

  **Penjelasan:** `localhost` di sini merujuk ke komputer host itu sendiri. Berkat aturan port forwarding yang sudah dibuat, request ke port 8080 pada host otomatis diteruskan ke port 80 pada VM, sehingga browser di host bisa mengakses halaman web yang berjalan di dalam VM tanpa perlu mengetahui IP VM secara langsung. Jika halaman profil kelompok berhasil tampil, ini membuktikan seluruh proses — mulai dari instalasi sistem, konfigurasi Nginx, hingga jaringan NAT — berjalan dengan benar secara menyeluruh.

* *[Tambahkan screenshot halaman profil kelompok yang berhasil diakses dari browser host di http://localhost:8080]*
  ![Akses Browser Host](images/06-browser-host.png)

## 🎥 Link Video Demo
[Tonton Video Demo Pengerjaan Tugas Kelompok di Google Drive](https://drive.google.com/file/d/1_fFGlP-1xCBUqy4K78ndMNQV9zMw8RMS/view?usp=drivesdk)

## 📄 Link Laporan
[Laporan Lengkap Instalasi Debian 13 Headless Web Server - Kelompok 4 (Google Drive)](https://drive.google.com/file/d/1GnltoVQskK7cOje7NgAXW8nG1EXXIjVM/view?usp=drivesdk)

*(Ganti tautan di atas dengan link Google Drive/OneDrive tempat file laporan `.docx` Kelompok 4 disimpan, dan pastikan akses berbagi diatur "Anyone with the link can view".)*

## 📝 Kesimpulan
Melalui praktikum ini, Kelompok 4 berhasil melakukan instalasi Debian 13 (Bookworm) dalam mode headless (CLI, tanpa Desktop Environment) di atas hypervisor VMware Workstation, mulai dari tahap partisi disk, pemilihan paket software minimal (SSH server dan standard system utilities), hingga pemasangan bootloader GRUB.

Setelah sistem dasar terpasang, kelompok mengonfigurasi user non-root dengan hak akses `sudo` sebagai praktik keamanan standar, sehingga aktivitas administratif tidak perlu dilakukan langsung sebagai `root`. Selanjutnya, web server **Nginx** berhasil diinstal, dijalankan, dan diaktifkan agar berjalan otomatis saat booting menggunakan `systemctl`, kemudian halaman default Nginx dimodifikasi menjadi halaman profil kelompok yang disajikan dalam format HTML statis.

Tantangan utama yang dihadapi adalah memahami konsep **jaringan NAT** pada VMware, di mana VM tidak dapat diakses langsung dari komputer host tanpa aturan **port forwarding**. Setelah port 8080 pada host dipetakan ke port 80 pada VM melalui Virtual Network Editor, halaman web profil kelompok berhasil diakses dari browser host melalui alamat `http://localhost:8080`, yang menandakan seluruh konfigurasi — mulai dari sistem operasi, service Nginx, hingga jaringan — telah berjalan dengan benar secara menyeluruh.

Poin-poin penting yang didapatkan dari praktikum ini antara lain:
1. Pemahaman instalasi sistem operasi Linux dalam mode teks/CLI tanpa antarmuka grafis.
2. Pentingnya prinsip *least privilege* melalui penggunaan user biasa dengan hak akses `sudo`, alih-alih login langsung sebagai `root`.
3. Kemampuan mengelola *service* di Linux menggunakan `systemctl` (start, enable, status, restart).
4. Pemahaman dasar administrasi web server Nginx, termasuk lokasi direktori root (`/var/www/html`) dan pengeditan halaman statis.
5. Pemahaman konsep jaringan virtual pada hypervisor (NAT) serta cara kerja port forwarding untuk menghubungkan jaringan host dan guest.
6. Pengalaman kerja sama tim dalam menyelesaikan tugas praktikum berbasis Linux server secara terstruktur dan terdokumentasi.
