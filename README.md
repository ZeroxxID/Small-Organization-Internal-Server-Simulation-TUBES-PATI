# SMALL ORGANIZATION INTERNAL SERVER SIMULATION GUIDE

## 👥 Tim Penyusun (Kelompok 4)
* **Administrator:** William
* **Anggota 1:** Farrel
* **Anggota 2:** Syahdat
* **Anggota 3:** Dimas
* **Anggota 4:** Keysha
  
---

## Pengenalan
### Virtualisasi
Virtualisasi adalah teknologi yang mensimulasikan perangkat keras, sistem operasi, ataupun jaringan di dalam satu mesin fisik. 
* **VirtualBox:** [Download di sini](https://www.virtualbox.org/wiki/Downloads)
* **VMWare Workstation:** [Download di sini](https://support.broadcom.com/group/ecx/downloads)

---

## PERSIAPAN

### Unduh Ubuntu Server
File ISO dapat diunduh melalui tautan berikut:
* **Global:** [Ubuntu Server](https://ubuntu.com/download/server)
* **Rilis Noble (24.04):** [Ubuntu Releases](https://releases.ubuntu.com/noble/)
* **Mirror Lokal (Lebih Cepat):** [Unair Mirror](https://mirror.unair.ac.id/ubuntu-cd/)

### Spesifikasi Virtual Machine (VM)
* **Nama VM:** PATI Kelompok 4
* **Disk:** 64 GB x 3 (1 OS Utama, 2 Disk untuk RAID 1)
* **Memory:** 8 GB
* **Processor:** 8 Cores
* **Network Adapter 1 (Bridge):** Mode DHCP (Akses Internet)
* **Network Adapter 2 (Custom VMnet2):** Mode Static (LAN Internal)


### Instalasi OS Ubuntu Server
Pastikan konfigurasi saat instalasi sesuai dengan parameter berikut:
* **Bahasa:** English
* **Keyboard Layout:** English (US) 
* **Mirror Address:** `https://mirror.unair.ac.id/ubuntu`
* **Name:** PATI Kelompok 4
* **Servers Name:** corp4
* **User:** william
* **Password:** `PATI_Kelompok#4#administrator`

---

## REPOSITORI LOKAL (Jika Tanpa source.list)
1. Edit konfigurasi repositori APT:
   ```bash
   sudo vi /etc/apt/sources.list.d/ubuntu.sources
   # Catatan: Ubuntu 24.04 (Noble) menggunakan format DEB822 di ubuntu.sources, bukan lagi sources.list lama.
   ```
2. Pastikan alamat URL mengarah ke mirror lokal agar proses instalasi lebih cepat:
   ```
   URIs: https://mirror.unair.ac.id/ubuntu/
   ```

---

## BASIC CONFIGURATION
### Pembuatan Hierarki Direktori
```bash
sudo mkdir -p /root/setup-scripts/     # Folder Script Installer
sudo mkdir -p /srv                     # Root Mount Point RAID
sudo mkdir -p /srv/projects            # Parent direktori Samba
sudo mkdir -p /srv/www/html            # DocumentRoot Apache Web Server
sudo mkdir -p /srv/projects/internal   # Ruang privat Administrator/Developer
sudo mkdir -p /srv/projects/eksternal  # Ruang publik untuk semua user
sudo mkdir -p /srv/backups             # Penyimpanan arsip otomatis
```

### Konfigurasi SSH Server (Hardened)
1. Install service SSH:
   ```bash
   sudo apt install openssh-server -y
   ```
2. Amankan konfigurasi daemon SSH:
   ```bash
   sudo vi /etc/ssh/sshd_config
   ```
3. Sesuaikan parameter keamanan berikut:
   ```plaintext
   Port 2222
   LoginGraceTime 1m
   PermitRootLogin no
   StrictModes yes
   MaxAuthTries 3
   PubkeyAuthentication yes
   PasswordAuthentication yes
   KbdInteractiveAuthentication no
   UsePAM yes
   X11Forwarding no
   PrintMotd no
   AcceptEnv LANG LC_*
   Subsystem sftp /usr/lib/openssh/sftp-server
   AllowGroups sudo developer
   ```
4. Matikan socket bawaan dan jalankan service SSH utama:
   ```bash
   sudo systemctl disable --now ssh.socket
   sudo systemctl enable ssh
   sudo systemctl restart ssh
   ```

### Konfigurasi Jaringan (Netplan)
1. Edit konfigurasi interface jaringan:
   ```bash
   sudo vi /etc/netplan/50-cloud-init.yaml
   ```
2. Tetapkan IP statis untuk antarmuka jaringan internal (VMnet2 / ens34):
   ```yaml
   network:
     version: 2
     ethernets:
       ens33:
         dhcp4: true
       ens34:
         dhcp4: false
         addresses:
           - 10.4.4.1/24
   ```
3. Aplikasikan perubahan jaringan:
   ```bash
   sudo netplan apply
   ```

### Tunneling Cloudflare Zero Trust
1. Login ke dasbor Cloudflare (https://cloudflare.com/).
2. Navigasi ke **Zero Trust** -> **Network** -> **Tunnels**.
3. Klik **Add a Tunnel**, beri nama `PATI`.
4. Pilih OS Debian dan salin token instalasinya.
5. Buat script instalasi lokal:
   ```bash
   sudo vi /root/setup-scripts/cloudflare.sh
   ```
6. Masukkan script instalasi:
   ```bash
   sudo mkdir -p --mode=0755 /usr/share/keyrings
   curl -fsSL https://pkg.cloudflare.com/cloudflare-public-v2.gpg | sudo tee /usr/share/keyrings/cloudflare-public-v2.gpg >/dev/null
   echo 'deb [signed-by=/usr/share/keyrings/cloudflare-public-v2.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
   sudo apt-get update && sudo apt-get install cloudflared
   sudo cloudflared service install [TOKEN]
   ```
7.  Eksekusi script tersebut:
    ```bash
    sudo bash /root/setup-scripts/cloudflare.sh
    ```
8.  Pada dasbor Cloudflare, set Public Hostname: `ssh.zeroxx.my.id` mengarah ke service `ssh://localhost:2222`.


### Sinkronisasi Waktu (NTP Client)
1. Atur zona waktu ke wilayah Indonesia:
   ```bash
   sudo timedatectl set-timezone Asia/Jakarta
   sudo timedatectl set-ntp true
   ```
2. Arahkan sinkronisasi ke server NTP lokal:
   ```bash
   sudo vi /etc/systemd/timesyncd.conf
   ```
3. Tambahkan konfigurasi pool Indonesia:
   ```plaintext
   NTP=id.pool.ntp.org
   FallbackNTP=ntp.ubuntu.com 0.asia.pool.ntp.org
   ```
4. Restart layanan dan verifikasi:
   ```bash
   sudo systemctl restart systemd-timesyncd
   timedatectl timesync-status
   ```

---

## MANAJEMEN PENYIMPANAN: RAID 1 (MIRRORING)
1. Install modul *mdadm*:
   ```bash
   sudo apt install mdadm -y
   ```
2. Ciptakan array RAID 1 menggunakan disk `sdb` dan `sdc`:
   ```bash 
   sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
   ```
3. Format disk virtual dengan *filesystem* ext4:
   ```bash
   sudo mkfs.ext4 /dev/md0
   ```
4. Daftarkan konfigurasi ke sistem secara permanen:
   ```bash
   sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
   sudo update-initramfs -u
   ```
5. Konfigurasi Auto-Mount saat Boot:
   Cek UUID disk menggunakan perintah `sudo blkid /dev/md0`.
   Masukkan nilai UUID tersebut ke file `/etc/fstab`:
   ```bash
   sudo vi /etc/fstab
   # Tambahkan baris (ganti UUID sesuai hasil blkid lu):
   UUID=xxx-xxx-xxx /srv ext4 defaults 0 2
   ```
6. Validasi file sistem:
   ```bash
   sudo systemctl daemon-reload
   sudo mount -a
   ```

## AUTOMATION CONFIGURATION
### Install Dependensi Skrip
```bash
sudo apt install bc libpam-pwquality -y
```

### 1. Monitoring System Health (Setiap 10 Menit)
1. Buat script monitoring:
   ```bash
   sudo vi /usr/local/sbin/monitor.sh
   ```
2. Masukkan logika *system check*:
   ```bash
   #!/bin/bash
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   LOG_FILE="/var/log/sysmon.log"
   TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

   CPU_USAGE=$(awk '{print $1*100/NR}' /proc/loadavg)
   RAM_USAGE=$(free -m | awk 'NR==2{printf "%.2f", $3*100/$2 }')
   DISK_OS=$(df -h / | awk 'NR==2{print $5}' | sed 's/%//')
   DISK_RAID=$(df -h /srv | awk 'NR==2{print $5}' | sed 's/%//')

   echo "[$TIMESTAMP] CPU: $CPU_USAGE% | RAM: $RAM_USAGE% | Disk OS: $DISK_OS% | Disk RAID: $DISK_RAID%" >> $LOG_FILE

   if (( $(echo "$CPU_USAGE > 70.0" | bc -l) )); then
      echo "[$TIMESTAMP] WARNING: CPU Usage tinggi! Terdeteksi $CPU_USAGE%" >> $LOG_FILE
   fi
   ```
3. Berikan izin eksekusi
   ```bash
   sudo chmod 700 /usr/local/sbin/monitor.sh
   ```

### 2. Log Audit Keamanan (Berjalan 23:55 WIB)
1. Buat script pencatatan audit:
   ```bash
   sudo vi /usr/local/sbin/audit.sh
   ```
2. Masukkan skrip forensik:
   ```bash
   #!/bin/bash
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   LOG_FILE="/var/log/audit_summary.log"
   TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

   echo "=== SYSTEM AUDIT & SECURITY REPORT - $TIMESTAMP ===" >> $LOG_FILE
   echo "[+] LAST LOGIN REPORT:" >> $LOG_FILE
   lastlog | grep -v "Never logged in" >> $LOG_FILE

   echo "[+] FAILED LOGIN ATTEMPTS (Last 24 Hours):" >> $LOG_FILE
   journalctl --since "24 hours ago" | grep "Failed password" >> $LOG_FILE

   if [ ${PIPESTATUS[1]} -ne 0 ]; then
      echo "Safe: Tidak ada indikasi brute-force hari ini." >> $LOG_FILE
   fi
   echo -e "\n" >> $LOG_FILE
   ```
3. Set hak akses:
   ```bash
   sudo chmod 700 /usr/local/sbin/audit.sh
   ```

### 3. Automasi Backup Server (Berjalan 23:30 WIB)
1. Buat script *archiving* direktori krusial:
   ```bash
   sudo vi /usr/local/sbin/daily_backup.sh
   ```
2. Masukkan logika backup:
   ```bash
   #!/bin/bash
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   TIMESTAMP=$(date +'%Y-%m-%d')
   BACKUP_DIR="/srv/backups"

   tar -czf "$BACKUP_DIR/sys_backup_$TIMESTAMP.tar.gz" /etc /var/log /srv/projects /srv/www /root/setup-scripts /usr/local/sbin 2>/dev/null
   find "$BACKUP_DIR" -type f -name "*.tar.gz" -mtime +7 -exec rm {} \;
   ```
3. Set hak akses eksekusi:
   ```bash
   sudo chmod 700 /usr/local/sbin/daily_backup.sh
   ```

### 4. Jadwalkan di Crontab & Konfigurasi Logrotate
1. Daftarkan seluruh otomatisasi ke dalam Cron:
   ```bash
   sudo crontab -e
   ```
2. Masukkan *rule* jadwal:
   ```plaintext
   */10 * * * * /usr/local/sbin/monitor.sh
   30 23 * * * /usr/local/sbin/daily_backup.sh
   55 23 * * * /usr/local/sbin/audit.sh
   ```
3. Buat file putaran log:
   ```bash
   sudo vi /etc/logrotate.d/sysadmin_logs
   ```
4. Konfigurasi batas file log:
   ```plaintext
   /var/log/sysmon.log /var/log/audit_summary.log {
      weekly
      rotate 4
      create 0600 root root
      compress
      missingok
      notifempty
   }
   ```

## USER MANAGEMENT & ACCESS CONTROL LIST (ACL)
### Install Dependensi Skrip
```bash
sudo apt install acl -y
```

1. Enkripsi standar kualitas *password*:
   ```bash
   sudo vi /etc/security/pwquality.conf
   # Set parameter berikut:
   minlen = 12
   minclass = 4
   usercheck = 1
   difok = 3
   ```
2. Buat skrip instalasi akun pengguna:
   ```bash
   sudo vi /root/setup-scripts/setup_user.sh
   ```
3. Masukkan arsitektur *Role-Based Access Control*:
   ```bash
   #!/bin/bash
   
   if [ "$EUID" -ne 0 ]; then
     echo "Jalankan script sebagai root (sudo)!"
     exit 1
   fi

   echo "[*] Memulai setup user dan ACL..."
   
   # 1. Bikin Grup Developer
   groupadd -f developer

   # 2. Pembuatan Akun
   useradd -m -s /bin/bash -G developer farrel    
   useradd -m -s /bin/bash -G developer syahdat
   useradd -M -s /usr/sbin/nologin dimas            
   useradd -M -s /usr/sbin/nologin keysha

   # 3. Set Default Password
   echo "farrel:PATI_Kelompok#4" | chpasswd
   echo "syahdat:PATI_Kelompok#4" | chpasswd
   echo "dimas:PATI_Kelompok#4" | chpasswd
   echo "keysha:PATI_Kelompok#4" | chpasswd

   # 4. Otomatisasi Sinkronisasi Database Samba
   echo -e "PATI_Kelompok#4#administrator\nPATI_Kelompok#4#administrator" | smbpasswd -a -s william
   echo -e "PATI_Kelompok#4\nPATI_Kelompok#4" | smbpasswd -a -s farrel
   echo -e "PATI_Kelompok#4\nPATI_Kelompok#4" | smbpasswd -a -s syahdat
   echo -e "PATI_Kelompok#4\nPATI_Kelompok#4" | smbpasswd -a -s dimas
   echo -e "PATI_Kelompok#4\nPATI_Kelompok#4" | smbpasswd -a -s keysha

   # 5. Enforce Password Expiration (Rotasi 5 Hari)
   chage -M 5 -W 1 farrel

   # 6. Setup Dasar Ownership Folder Samba
   chown -R root:developer /srv/projects
   chmod 2770 /srv/projects
   chmod 2770 /srv/projects/internal
   chmod 2770 /srv/projects/eksternal

   # 7. Konfigurasi Access Control List (ACL)
   # Grup developer diberikan hak akses penuh rwx
   setfacl -R -m g:developer:rwx,d:g:developer:rwx /srv/projects
   setfacl -R -m u:william:rwx /srv/projects

   # Blokir mutlak user nologin dari folder internal
   setfacl -m u:dimas:---,d:u:dimas:--- /srv/projects/internal
   setfacl -m u:keysha:---,d:u:keysha:--- /srv/projects/internal

   # Izinkan user nologin masuk dan baca folder eksternal
   setfacl -m u:dimas:rwx,d:u:dimas:rwx /srv/projects/eksternal
   setfacl -m u:keysha:rwx,d:u:keysha:rwx /srv/projects/eksternal

   # Implementasi file spesifik
   echo "Data Konfidensial Farrel" > /srv/projects/internal/dokumen_rahasia_farrel.txt
   echo "Farrel: tidur, Syahdat: Kerja" > /srv/projects/internal/pembagian_tugas.txt
   echo "Data Konfidensial Syahdat" > /srv/projects/internal/dokumen_rahasia_syahdat.txt
   
   setfacl -b /srv/projects/internal/dokumen_rahasia_farrel.txt
   setfacl -b /srv/projects/internal/pembagian_tugas.txt
   setfacl -b /srv/projects/internal/dokumen_rahasia_syahdat.txt

   chown farrel:developer /srv/projects/internal/dokumen_rahasia_farrel.txt
   chown farrel:developer /srv/projects/internal/pembagian_tugas.txt
   chown syahdat:developer /srv/projects/internal/dokumen_rahasia_syahdat.txt
   
   chmod 0600 /srv/projects/internal/dokumen_rahasia_farrel.txt
   sudo setfacl -m u:william:rw /srv/projects/internal/dokumen_rahasia_farrel.txt
   
   chmod 0640 /srv/projects/internal/pembagian_tugas.txt
   sudo setfacl -m u:william:rw /srv/projects/internal/pembagian_tugas.txt
   
   chmod 0660 /srv/projects/internal/dokumen_rahasia_syahdat.txt
   sudo setfacl -m u:william:rw /srv/projects/internal/dokumen_rahasia_syahdat.txt

   echo "[+] Setup Akun dan ACL Selesai!"
   ```
4. Eksekusi skrip:
   ```bash
   sudo bash /root/setup-scripts/setup_user.sh
   ```

## FILE SHARING (SAMBA SERVER)
1. Instalasi Samba:
   ```bash
   sudo apt install samba -y
   ```
2. Modifikasi konfigurasi sistem *sharing*:
   ```
   sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
   sudo vi /etc/samba/smb.conf
   ```
3. Sisipkan parameter jaringan *secure* di baris akhir:
   ```toml
   [Projects]
      comment = Internal Corporate Projects
      path = /srv/projects
      browseable = yes
      read only = no
      valid users = @sudo, @developer, dimas, keysha
      create mask = 0770
      directory mask = 0770
      force create mode = 0770
      force directory mode = 0770
      force group = developer
      vfs objects = acl_xattr
      map acl inherit = yes
      inherit acls = yes
   ```
4. Restart *daemon*:
   ```bash
   sudo systemctl restart smbd nmbd
   ```

## INTERNAL DNS (BIND9)
1. Instalasi modul resolusi nama domain:
   ```bash
   sudo apt install bind9 bind9utils bind9-doc dnsutils -y
   ```
2. Daftarkan zona lokal `corp4.local`:
   ```bash
   sudo vi /etc/bind/named.conf.local
   ```
   Tambahkan blok ini:
   ```plaintext
   zone "corp4.local" {
      type master;
      file "/etc/bind/db.corp4.local";
   };
   ```
3. Buat *database* DNS internal:
   ```bash
   sudo cp /etc/bind/db.local /etc/bind/db.corp4.local
   sudo vi /etc/bind/db.corp4.local
   ```
4. Konfigurasikan *record* A, MX, dan CNAME:
   ```plaintext
   $TTL    604800
   @       IN      SOA     ns1.corp4.local. admin.corp4.local. (
                                 2         ; Serial
                            604800         ; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                            604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      ns1.corp4.local.
   @       IN      MX  10  mail.corp4.local.

   ns1     IN      A       10.4.4.1
   mail    IN      A       10.4.4.1
   www     IN      A       10.4.4.1
   git     IN      A       127.0.0.1
   dev     IN      CNAME   git
   ```
5. Edit file `named.conf.options`
   ```
   sudo vi /etc/bind/named.conf.options
   # Ubah menjadi berikut
   forwarders {
      1.1.1.1;
      8.8.8.8;
   };
   ```
6. Edit file resolved:
   ```bash
   sudo vi /etc/systemd/resolved.conf
   ```
7. Hapus tanda pagar (#) dan ubah baris ini:
   ```plaintext
   DNS=127.0.0.1
   Domains=~corp4.local
   ```
8. Restart resolver-nya:
   ```bash
   sudo systemctl restart systemd-resolved
   ```
9. *Restart* layanan dan lakukan validasi resolusi DNS:
   ```bash
   sudo systemctl restart bind9
   ```
10. Pengujian
    ```
    dig git.corp4.local
    dig www.corp4.local
    host mail.corp4.local
    nslookup dev.corp4.local
    ```

## WEB SERVER (APACHE)
1. Instalasi modul Apache:
   ```bash
   sudo apt install apache2 -y
   ```
2. Isolasi *Directory Root* ke disk RAID:
   ```bash
   sudo vi /etc/apache2/sites-available/000-default.conf
   # Ubah DocumentRoot menjadi /srv/www/html
   ```
3. Aktifkan *permission* pada direktori khusus di file sentral:
   ```bash
   sudo vi /etc/apache2/apache2.conf
   # Tambahkan:
   <Directory /srv/www/>
      Options -Indexes +FollowSymLinks
      AllowOverride None
      Require all granted
   </Directory>
   ```
4. Tutup celah informasi versi OS (Security Hardening):
   ```bash
   sudo vi /etc/apache2/conf-available/security.conf
   # Pastikan bernilai:
   ServerTokens Prod
   ServerSignature Off
   ```
5. Terapkan konfigurasi:
   ```bash
   sudo systemctl restart apache2
   ```

## FIREWALL (NFTABLES ZERO-TRUST)
1. Pasang *nftables* dan matikan *firewall* lawas (UFW):
   ```bash
   sudo apt install nftables -y
   sudo systemctl disable --now ufw
   sudo systemctl mask ufw
   ```
2. Buat *ruleset* tingkat kernel:
   ```bash
   sudo vi /etc/nftables.conf
   ```
3. Masukkan postur penolakan *default* (Default Drop Policy):
   ```plaintext
   #!/usr/sbin/nft -f
   flush ruleset
   table inet filter {
       chain input {
           type filter hook input priority 0; policy drop;
   
           iif "lo" accept
           ct state established,related accept
   
           # Cloudflare & Web Access
           tcp dport 2222 accept
           tcp dport { 80, 443 } accept
           udp dport 7844 accept

           # Isolasi Jaringan Internal via ens34
           iifname "ens34" tcp dport 53 accept
           iifname "ens34" udp dport 53 accept
           iifname "ens34" tcp dport { 139, 445 } accept
           iifname "ens34" udp dport { 137, 138 } accept
   
           # Proteksi Ping Flood
           ip protocol icmp limit rate 10/second accept
       }
       chain forward {
           type filter hook forward priority 0; policy drop;
       }
       chain output {
           type filter hook output priority 0; policy accept;
       }
   }
   ```
4. Nyalakan dan aplikasikan ruleset:
   ```bash
   sudo systemctl enable nftables
   sudo nft -f /etc/nftables.conf
   ```

## OPSIONAL (CLIENT & GUI)
### Desktop Environment Ringan
```bash
sudo apt install xubuntu-desktop task-xfce-desktop -y
reboot
```

### Copy File on SSH
1. Copy file dari server ke lokal
   ```
   scp -P 2222 [Username]@[IP/Domain]:[Path File Server] [Path File Lokal]
   ```
2. Copy file dari lokal ke server
   ```
   scp -P 2222 [Path File Lokal] [Username]@[IP/Domain]:[Path File Server] 
   ```

### SSH Client via Cloudflare (Remote Work)
#### Windows
1. Unduh `.exe`:
   ```
   https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe
   ```
2. Modifikasi SSH config:
   ```dos
   ssh -o "ProxyCommand=cloudflared access ssh --hostname ssh.zeroxx.my.id" william@ssh.zeroxx.my.id
   ```
#### Linux (Debian/Ubuntu) / Mac (Homebrew)
1. Instal paket:
   ```bash
   brew install cloudflared # Untuk MacOS
   sudo apt install cloudflared # Untuk Linux
   ```
2. Hubungkan ke server:
     ```
     ssh -o "ProxyCommand=cloudflared access ssh --hostname ssh.zeroxx.my.id" william@ssh.zeroxx.my.id
     ```
