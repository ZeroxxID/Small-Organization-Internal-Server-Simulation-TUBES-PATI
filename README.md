# **MINI SERVER GUIDE**
## **Pengenalan**
### Virtualisasi
- Virtualisasi adalah teknologi yang mensimulasikan perangkat keras, sistem operasi, ataupun jaringan di dalam satu mesin fisik 
- VirtualBox: https://www.virtualbox.org/wiki/Downloads
- VMWare Workstation: https://support.broadcom.com/group/ecx/downloads

---

## **PERSIAPAN**
### Ubuntu Server
Iso dapat diunduh di
- https://ubuntu.com/download/server
- https://releases.ubuntu.com/noble/
- https://mirror.unair.ac.id/ubuntu-cd/ 

### Tahap Instalasi
1. Virtual Machine Configuration
   - Nama: PATI Kelompok 4
   - Disk: 64 GB x 3 (1 OS, 2 RAID)
   - Memory: 8 GB
   - Procesor: 8 Cores
   - Network Adapter 
     - Bridge = DHCP
     - Custom VMnet2 = Static
     - Custom VMnet3 = Static
2. Virtual Machine Configuration Backup
   - Nama: PATI Kelompok 4 Backup
   - Disk: 64 GB x 3 (1 OS, 2 RAID)
   - Memory: 8 GB
   - Procesor: 8 Cores
   - Network Adapter 
     - Bridge = DHCP
     - Custom VMnet2 = Static
     - Custom VMnet3 = Static

3. Instalasi OS Ubuntu Server
   - Bahasa: English
   - Keyboard Layout: English (US) 
   - Mirror Address: https://mirror.unair.ac.id/ubuntu
   - Name: PATI Kelompok 7
   - Servers Name: ubuntu
   - User: william
   - Password: PATI_Kelompok#4#administrator


## **WORST CASE SETELAH INSTALL TANPA** ***source.list***
1. Tambahan list sumber repository secara manual di direktori Advance Package Tool (APT)
   ```
   sudo vi /etc/apt/source.list
   ```
2. Tambahkan repositori berikut pada `souce.list` dan berikan `#` pada bagian `deb cdrom:[Ubuntu-Server 24.04 _Noble Numbat_ - Release amd64 (20240423)]/ noble main restricted`
   ```
   deb https://mirror.unair.ac.id/ubuntu/ noble main restricted universe multiverse
   deb-src https://mirror.unair.ac.id/ubuntu/ noble main restricted universe multiverse

   deb https://mirror.unair.ac.id/ubuntu/ noble-updates main restricted universe multiverse
   deb-src https://mirror.unair.ac.id/ubuntu/ noble-updates main restricted universe multiverse

   deb https://mirror.unair.ac.id/ubuntu/ noble-security main restricted universe multiverse
   deb-src https://mirror.unair.ac.id/ubuntu/ noble-security main restricted universe multiverse

   deb https://mirror.unair.ac.id/ubuntu/ noble-backports main restricted universe multiverse
   deb-src https://mirror.unair.ac.id/ubuntu/ noble-backports main restricted universe multiverse

   deb https://mirror.unair.ac.id/ubuntu/ noble-proposed main restricted universe multiverse
   deb-src https://mirror.unair.ac.id/ubuntu/ noble-proposed main restricted universe multiverse
   ```

---

## **BASIC CONFIGURATION**
### SSH Server
1. Install SSH Server
   ```
   sudo apt install openssh-server
   ```
2. Aktifkan service `ssh`
   ```
   sudo systemctl enable ssh
   ```
3. Ubah sedikit konfigurasi `ssh`
   ```
   sudo vi /etc/ssh/sshd_config
   ```
4. Masukan konfigurasi berikut
   ```
   # This is the sshd server system-wide configuration file.  See
   # sshd_config(5) for more information.

   # This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

   # The strategy used for options in the default sshd_config shipped with
   # OpenSSH is to specify options with their default value where
   # possible, but leave them commented.  Uncommented options override the
   # default value.

   Include /etc/ssh/sshd_config.d/*.conf

   # When systemd socket activation is used (the default), the socket
   # configuration must be re-generated after changing Port, AddressFamily, or
   # ListenAddress.
   #
   # For changes to take effect, run:
   #
   #   systemctl daemon-reload
   #   systemctl restart ssh.socket
   #
   Port 22
   #AddressFamily any
   #ListenAddress 0.0.0.0
   #ListenAddress ::

   #HostKey /etc/ssh/ssh_host_rsa_key
   #HostKey /etc/ssh/ssh_host_ecdsa_key
   #HostKey /etc/ssh/ssh_host_ed25519_key

   # Ciphers and keying
   #RekeyLimit default none

   # Logging
   #SyslogFacility AUTH
   #LogLevel INFO

   # Authentication:

   LoginGraceTime 1m
   PermitRootLogin no
   StrictModes yes
   MaxAuthTries 3
   #MaxSessions 10

   #PubkeyAuthentication yes

   # Expect .ssh/authorized_keys2 to be disregarded by default in future.
   #AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

   #AuthorizedPrincipalsFile none

   #AuthorizedKeysCommand none
   #AuthorizedKeysCommandUser nobody

   # For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
   #HostbasedAuthentication no
   # Change to yes if you don't trust ~/.ssh/known_hosts for
   # HostbasedAuthentication
   #IgnoreUserKnownHosts no
   # Don't read the user's ~/.rhosts and ~/.shosts files
   #IgnoreRhosts yes

   # To disable tunneled clear text passwords, change to no here!
   #PasswordAuthentication yes
   #PermitEmptyPasswords no

   # Change to yes to enable challenge-response passwords (beware issues with
   # some PAM modules and threads)
   KbdInteractiveAuthentication no

   # Kerberos options
   #KerberosAuthentication no
   #KerberosOrLocalPasswd yes
   #KerberosTicketCleanup yes
   #KerberosGetAFSToken no

   # GSSAPI options
   #GSSAPIAuthentication no
   #GSSAPICleanupCredentials yes
   #GSSAPIStrictAcceptorCheck yes
   #GSSAPIKeyExchange no

   # Set this to 'yes' to enable PAM authentication, account processing,
   # and session processing. If this is enabled, PAM authentication will
   # be allowed through the KbdInteractiveAuthentication and
   # PasswordAuthentication.  Depending on your PAM configuration,
   # PAM authentication via KbdInteractiveAuthentication may bypass
   # the setting of "PermitRootLogin prohibit-password".
   # If you just want the PAM account and session checks to run without
   # PAM authentication, then enable this but set PasswordAuthentication
   # and KbdInteractiveAuthentication to 'no'.
   UsePAM yes

   #AllowAgentForwarding yes
   #AllowTcpForwarding yes
   #GatewayPorts no
   X11Forwarding yes
   #X11DisplayOffset 10
   #X11UseLocalhost yes
   #PermitTTY yes
   PrintMotd no
   #PrintLastLog yes
   #TCPKeepAlive yes
   #PermitUserEnvironment no
   #Compression delayed
   #ClientAliveInterval 0
   #ClientAliveCountMax 3
   #UseDNS no
   #PidFile /run/sshd.pid
   #MaxStartups 10:30:100
   #PermitTunnel no
   #ChrootDirectory none
   #VersionAddendum none

   # no default banner path
   #Banner none

   # Allow client to pass locale environment variables
   AcceptEnv LANG LC_*

   # override default of no subsystems
   Subsystem	sftp	/usr/lib/openssh/sftp-server

   # Example of overriding settings on a per-user basis
   #Match User anoncvs
   #	X11Forwarding no
   #	AllowTcpForwarding no
   #	PermitTTY no
   #	ForceCommand cvs server
   ```
5. Restart service SSH
   ```
   sudo systemctl restart ssh
   ```

### Configure Network Adapter
1. Pada Ubuntu Server menggunakan netplan sebagai service nya
   ```
   sudo vi /etc/netplan/50-cloud-init.yaml
   ```
2. Tambahkan konfigurasi berikut ke `50-cloud-init.yaml`
   ```
   network:
     version: 2
     ethernets:
       [Interface]: <- ens33:
         dhcp4: true
       [Interface]: <- ens34:
         dhcp4: false
         addresses:
           - IP/CIDR <- 2.2.2.2/8
       [Interface]: <- ens35:
         dhcp4: false
         addresses:
           - IP/CIDR <- 20.20.20.20/24 | 20.20.20.22/24
   ```
3. Aplikasikan konfigurasi `netplan`
   ```
   sudo netplan apply
   ```

### Tunneling SSH dengan Domain
1. Buka `cloudflare` dan login: https://cloudflare.com/
2. Klik `Zero Trust` pada menu Protect & Connect
3. Klik menu `Network` dan pilih submenu `Overview`
4. Klik `Manage Tunnels`, klik `Add a Tunnel`
5. Isi `Tunnel Name` <- PATI
6. Install dan Run Connector dengan Operating System `Debian`
7. Buat direktori `scripts`
   ```
   sudo mkdir -p /opt/scripts
   ```
8. Buat shell script agar memudahkan
   ```
   sudo vi /opt/scripts/cloudflare.sh
   ```
9. Tambahkan script berikut
   ```
   # Add cloudflare gpg key
   sudo mkdir -p --mode=0755 /usr/share/keyrings
   curl -fsSL https://pkg.cloudflare.com/cloudflare-public-v2.gpg | sudo tee /usr/share/keyrings/cloudflare-public-v2.gpg >/dev/null

   # Add this repo to your apt repositories
   echo 'deb [signed-by=/usr/share/keyrings/cloudflare-public-v2.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

   # install cloudflared
   sudo apt-get update && sudo apt-get install cloudflared

   # Install Token
   sudo cloudflared service install [Token]
   ```
10. Beri izin eksekusi
    ```
    sudo chmod +x /opt/scripts/cloudflare.sh
    ```
11. Jalankan scriptnya
    ```
    /opt/scripts/cloudflare.sh
    ```
12. Isi Hostname `Subdomain` <- ssh, `Domain` <- zeroxx.my.id
13. Isi Service `Type` <- ssh, `URL` <- localhost:22


### Network Time Protocol (NTP) Client
1. Tampilkan detail informasi mengenai waktu
   ```
   timedatectl
   ```
2. Jika time zone masih wilayah luar
   ```
   sudo timedatectl set-timezone Asia/Jakarta
   ```
3. Jika NTP Service tidak aktif
   ```
   sudo timedatectl set-ntp true
   ```
4. Lakukan konfigurasi NTP agar waktu sinkron dengan wilayah Indonesia di direktori `systemd`
   ```
   sudo vi /etc/systemd/timesyncd.conf
   ```
5. Hilangkan `#` pada bagian `NTP=` dan tambahakan pool seperti berikut
   ```
   NTP=id.pool.ntp.org
   ```
6. Lakukan restart service untuk mengaplikasikannya
   ```
   sudo systemctl restart systemd-timesyncd
   ```

---

## **CORE CONFIGURATION**
### Implementasi RAID 1
1. Install tools `mdadm`
   ```
   sudo apt install mdadm -y
   ```
2. Buat RAID antara `sdb` dan `sdc`
   ``` 
   sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
   ```
3. Untuk mengecek status sinkronisasi
   ```
   cat /proc/mdstat
   ```
4. Format `md0` menjadi file sistem bertipe `ext4`
   ```
   sudo mkfs.ext4 /dev/md0
   ```
5. Buat lokasi mount point
   ```
   sudo mkdir -p /srv/projects
   ```
6. Mount disk `md0` ke partisi `/srv/projects` 
   ```
   sudo mount /dev/md0 /srv/projects
   ```
6. Update konfigurasi di `mdadm`
   ```
   sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
   ```
7. Update `initramfs`
   ```
   sudo update-initramfs -u
   ```
8. Cek UUID `md0`
   ```
   sudo blkid /dev/md0
   ```
9. Masukan UUID tersebut ke `fstab`
   ```
   sudo vi /etc/fstab
   ```
10. Masukan konfigurasi berikut
    ```
    UUID=[UUID md0] <- UUID=ccee6545-9bed-44d4-ab8f-333c735eec7a /data  ext4  defaults  0  2
    ```
11. Reload `systemd` untuk mengaplikasikannya
    ```
    sudo systemctl daemon-reload
    ```

### Verifikasi RAID
1. Unmount partisi `/data`
   ```
   sudo umount /data
   ```
2. Test Auto-Mount dari `fstab`
   ```
   sudo mount -a
   ```
3. Cek status
   ```
   df -h | grep /data
   ```

### Membuat Service Startup Update
1. Buat service baru di direktori `system`
   ```
   sudo vi /etc/systemd/system/startup-update.service
   ```
2. Tambahkan konfigurasi berikut
   ```
   [Unit]
   Description=Auto Update and Upgrade on Startup
   After=network-online.target
   Wants=network-online.target

   [Service]
   Type=oneshot
   ExecStart=/usr/bin/apt update -y
   ExecStart=/usr/bin/apt upgrade -y
   StandardOutput=journal
   StandardError=journal

   [Install]
   WantedBy=multi-user.target
   ```
3. Lakukan daemon reload untuk mengaplikasikannya
   ```
   sudo systemctl daemon-reload
   ```
4. Aktifkan service yang telah dibuat
   ```
   sudo systemctl enable startup-update.service
   ```
5. Cek Status Service
   ```
   systemctl status [nama].service
   ```
6. Cek Log Service Spesifik
   ```
   journalctl -u [nama].service
   ```

### Otomasi Backup Data Pada 23.00
1. Buat direktori backup 
   ```
   sudo mkdir -p /home/administrator/backups
   ```
2. Buat script backup
   ```
   sudo vi /opt/scripts/daily_backup.sh
   ```
3. Tambahkan script berikut
   ```
   #!/bin/bash
   # Backup dari RAID ke folder backup di OS disk
   DEST="/home/administrator/backups/$(date +%Y-%m-%d)"
   mkdir -p $DEST
   rsync -av --delete /data/ $DEST
   chown -R administrator:member $DEST
   echo "Backup completed at $(date)" >> /var/log/backup.log
   ```
4. Berikan izin eksekusi
   ```
   sudo chmod +x /opt/scripts/daily_backup.sh
   ```
5. Ubah akses backup ke owner administrator group member
   ```
   sudo chown -R administrator:member /home/administrator/backups
   ```
6. Edit jadwal di `crontab`
   ```
   sudo crontab -e
   ```
7. Tambahkan konfigurasi berikut
   ```
   0 23 * * * /opt/scripts/daily_backup.sh
   ```

### Otomasi Reboot Pada 00.00
1. Edit jadwal di `crontab`
   ```
   sudo crontab -e
   ```
2. Tambahkan konfigurasi berikut
   ```
   0 0 * * * /usr/bin/systemctl reboot
   ```
3. Mengecek jadwal yang berjalan
   ```
   sudo crontab -l

### Otomasi Monitoring Setiap 10 Menit



### Install Apache
1. Install Apache
   ```
   sudo apt install apache2 -y
   ```
2. Buat folder web di RAID
   ```
   sudo mkdir -p /data/www/html
   ```
3. Edit konfigurasi default Apache
   ```
   sudo vi /etc/apache2/sites-available/000-default.conf
   ```
4. Ubah konfigurasi sebagai berikut
   ```
   DocumentRoot /data/www/html
   ```
5. Edit konfigurasi utama Apache
   ```
   sudo vi /etc/apache2/apache2.conf
   ```
6. Ubah konfigurasi sebagai berikut
   ```
   <Directory /data/www/>
      Options Indexes FollowSymLinks
      AllowOverride None
      Require all granted
   </Directory>
   ```
7. Restart service Apache
   ```
   sudo systemctl restart apache2
   ```

## NFtable
1. Install `nftables`
   ```
   sudo apt install nftables -y
   ```
2. Aktifkan service `nftables`
   ```
   sudo systemctl enable nftables
   ```
3. Konfigurasi Rule Utama:
   ```
   sudo vi /etc/nftables.conf
   ```
4. Tambahkan rule berikut
   ```
   #!/usr/sbin/nft -f

   flush ruleset

   table inet filter {
       chain input {
           type filter hook input priority 0; policy drop;
   
           # 1. Allow loopback (internal komunikasi)
           iif "lo" accept
   
           # 2. Allow traffic yang sudah terjalin (Established/Related)
           ct state established,related accept

           # --- BAGIAN HARDENING: DENY TELNET ---
           # Kita taruh di atas biar dicek duluan (Explicit Deny)
           tcp dport 23 reject with tcp reset
           # -------------------------------------
   
           # 3. Allow SSH (Port 22) - Krusial buat Remote
           tcp dport 22 accept
   
           # 4. Allow HTTP (Port 80) buat Web Server lo
           tcp dport 80 accept
   
           # 5. Allow Samba (Port 139, 445), NetBIOS (137, 138), Cloudflared (443, 7844)
           tcp dport { 139, 443, 445 } accept
           udp dport { 137, 138, 7844 } accept
   
           # 6. Allow ICMP (Ping) buat diagnosa jaringan
           ip protocol icmp accept
       }
   
       chain forward {
           type filter hook forward priority 0; policy drop;
       }
   
       chain output {
           type filter hook output priority 0; policy accept;
       }
   }
   ```
5. Terapkan rules
   ```
   sudo nft -f /etc/nftables.conf
   ```
6. Untuk cek status dapat menggunakan
   ```
   sudo nft list ruleset
   ```

### Access Control List
1. User
   - Tambahkan grup `public`
     ```
     sudo groupadd public
     ```
   - Tambahkan user ke group `public`, buatkan home direktorinya, berikan shell, dan berikan password
     ```
     sudo useradd -m -g public -s /bin/bash user
     echo "user:user" | sudo chpasswd
     ```

2. Administrator
   - Tambahkan grup `member`
     ```
     sudo groupadd member
     ```
   - Tambahkan user ke group `member`, buatkan home direktorinya, berikan shell, dan berikan password
     ```
     sudo useradd -m -g member -s /bin/bash rava
     echo "rava:rava" | sudo chpasswd

     sudo useradd -m -g member -s /bin/bash ariiq
     echo "ariiq:ariiq" | sudo chpasswd

     sudo useradd -m -g member -s /bin/bash fathan
     echo "fathan:fathan" | sudo chpasswd

     sudo useradd -m -g member -s /bin/bash rahmat
     echo "rahmat:rahmat" | sudo chpasswd
     ```
3. Tambahin permission sudo
   ```
   sudo usermod -aG sudo rava
   sudo usermod -aG sudo ariiq
   sudo usermod -aG sudo fathan
   sudo usermod -aG sudo rahmat
   sudo usermod -aG member administrator
   sudo usermod -aG member www-data
   ```
4. Tambahin hak akses ke `/data`
   ```
   sudo chown -R root:member /data
   sudo chmod -R 2775 /data
   sudo chmod -R 770 /data/www
   ```

## File Sharing
### Samba Server
- Install Samba
  ```
  sudo apt install samba -y
  ```
- Konfigurasi folder sharing di /srv/projects (RAID):
  ```
  sudo vi /etc/samba/smb.conf
  ```
- Masukan konfigurasi berikut
  ```
  [Data-Center]
  path = /data
  browseable = yes
  read only = no
  guest ok = no
  valid users = administrator @member @public
     
  # Admin bisa nulis, user biasa cuma bisa baca
  write list = administrator @member

  # Access Control
  create mask = 0664
  directory mask = 0775
  force create mode = 0664
  force directory mode = 0775
  ```
- Daftarin password Samba buat user
  ```
  sudo smbpasswd -a administrator
  sudo smbpasswd -a user
  sudo smbpasswd -a rava
  sudo smbpasswd -a ariiq
  sudo smbpasswd -a fathan
  sudo smbpasswd -a rahmat
  ```
- Restart service Samba
  ```
  sudo systemctl restart smbd
  ```

---

## **OPSIONAL**
### Install Desktop Environment
1. Install Desktop Environment XFCE
   ```
   sudo apt install xubuntu-desktop task-xfce-desktop -y
   ```
2. Lakukan restart untuk mengaplikasikannya.
   ```
   reboot
   ```

### Copy File on SSH
1. Copy file dari server ke lokal
   ```
   scp [Username]@[IP/Domain]:[Path File Server] [Path File Lokal]
   ```
2. Copy file dari lokal ke server
   ```
   scp [Path File Lokal] [Username]@[IP/Domain]:[Path File Server] 
   ```

### SSH Client
1. Windows
   - Install
     - https://github.com/cloudflare/cloudflared/releases/download/2026.3.0/cloudflared-windows-amd64.exe
     - https://github.com/cloudflare/cloudflared/releases/download/2026.3.0/cloudflared-windows-amd64.msi
     - Connect
       ```
       ssh -o "ProxyCommand=cloudflared access ssh --hostname ssh.zeroxx.my.id" [Username]@ssh.zeroxx.my.id
       ```
2. Linux Debian
   - Install
     ```
     wget https://github.com/cloudflare/cloudflared/releases/download/2026.3.0/cloudflared-linux-amd64.deb
     sudo dpkg -i cloudflared-linux-amd64.deb
     ```
   - Connect
     ```
     ssh -o "ProxyCommand=cloudflared access ssh --hostname ssh.zeroxx.my.id" [Username]@ssh.zeroxx.my.id
     ```
3. Mac Intel
   - Install melalui package manager
     ```
     brew install cloudflared
     ```
   - Install melalui package binary
     ```
     curl -O https://github.com/cloudflare/cloudflared/releases/download/2026.3.0/cloudflared-darwin-amd64.tgz
     tar -xvzf cloudflared-darwin-amd64.tgz
     sudo mv cloudflared /usr/local/bin/
     sudo chmod +x /usr/local/bin/cloudflared
     ```
   - Connect
     ```
     ssh -o "ProxyCommand=/usr/local/bin/cloudflared access ssh --hostname ssh.zeroxx.my.id" [Username]@ssh.zeroxx.my.id
     ```
4. Mac M1/M2/M3 (Apple Silicon)
   - Install melalui package manager
     ```
     brew install cloudflared
     ```
   - Install melalui package binary
     ```
     curl -O https://github.com/cloudflare/cloudflared/releases/download/2026.3.0/cloudflared-darwin-arm64.tgz
     tar -xvzf cloudflared-darwin-arm64.tgz
     sudo mv cloudflared /usr/local/bin/
     sudo chmod +x /usr/local/bin/cloudflared
     ```
   - Connect
     ```
     ssh -o "ProxyCommand=/usr/local/bin/cloudflared access ssh --hostname ssh.zeroxx.my.id" [Username]@ssh.zeroxx.my.id
     ```

---

# **TUGAS PRAKTIKUM & PRESENTASI KELOMPOK**
### Boot & Service Management
1. Mengidentifikasi:
   - Boot loader yang digunakan
     - Dapat dilihat pada direktori `boot`
       ```
       ls -la /boot
       ```
     - Dapat menggunakan tool `bootinfoscript` 
       - Install `boot-info-script`
         ```
         sudo apt install boot-info-script -y
         ```
       - Jalankan `bootinfoscript`
         ```
         sudo bootinfoscript [nama file]
         ```
       - Cek informasi yang tersimpan
         ```
         cat [nama file]
         ```
   - Kernel system
     - Dapat menggunakan `hostnamectl`
       ```
       hostnamectl
       ```
     - Dapat menggunakan tool `neofetch` atau `fastfetch`
       ```
       sudo apt install neofetch/fastfetch -y
       neofetch/fastfetch
       ```
   - Menampilkan daftar service yang aktif saat startup
     - Untuk menampilkan service saat startup
       ```
       systemctl list-unit-files --type=service/-t service --state=enabled 
       ```
     - Untuk menampilkan service yang sedang berjalan
       ```
       systemctl list-units -t service
       ```
   - Menonaktifkan minimal 1 service yang tidak penting
     ```
     sudo systemctl disable [nama service] <- ufw
     ```
   - Membuat 1 custom startup service -> Sudah dibuat pada `Core Configuration`

2. Output wajib:
   - Screenshot / demo systemctl	
   - Penjelasan fungsi service

---

## Manajemen Disk & Partisi
1. Menggunakan minimal 2 disk
   - 64 GB  -> sda
   - 64 GB -> sdb
   - 64 GB -> sdc
2. Membuat:
   - Partisi sistem
     - **sda (64 GB):** Harddisk utama yang digunakan untuk instalasi sistem operasi
       - **sda1 (1 MB)** -> **BIOS Boot Partition:** Berisi kode bootloader (GRUB) untuk inisiasi proses booting pada sistem berbasis GPT.
       - **sda2 (2 GB)** -> `/boot`**:** Partisi khusus untuk menyimpan Kernel Linux dan file konfigurasi awal booting.
       - **sda3 (62 GB)** -> **LVM Container:** Berisi Physical Volume (PV) yang menjadi wadah bagi Volume Group (VG) Ubuntu.
          - `ubuntu--vg-ubuntu--lv` **(31 GB)** -> `/` **(Root)**
          - **Note:** Di dalam root ini juga terdapat file `/swap.img` sebagai memori virtual sistem.
          - **Unallocated Space (31 GB)**: Sisa kapasitas di dalam LVM yang disiapkan untuk ekspansi penyimpanan sistem secara on-the-fly tanpa perlu restart (skalabilitas masa depan).
   - Partisi data (/data)
     - **sdb & sdc (64 GB):** Harddisk identik yang digunakan sebagai anggota array 
       - md0 (63,9GB) -> RAID1 (Menggabungkan `sdb` dan `sdc` sehingga data terduplikasi secara otomatis (mirroring). Jika salah satu disk rusak, data tetap aman.) `/data`: Digunakan sebagai direktori utama untuk data layanan kritis (seperti direktori web Apache).
   - Menjelaskan fungsi masing-masing partisi -> Sudah dijelaskan di `Partisi sistem` dan `Partisi data (/data)`
3. Output :
   - Screenshot lsblk
   - Struktur partisi
     ```
     NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
     loop0                       7:0    0  48.4M  1 loop  /snap/snapd/26382
     loop1                       7:1    0  66.8M  1 loop  /snap/core24/1587
     loop2                       7:2    0     4K  1 loop  /snap/bare/5
     loop3                       7:3    0  91.7M  1 loop  /snap/gtk-common-themes/1535
     loop4                       7:4    0 273.7M  1 loop  /snap/firefox/8107
     loop5                       7:5    0   395M  1 loop  /snap/mesa-2404/1165
     loop6                       7:6    0 606.1M  1 loop  /snap/gnome-46-2404/153
     loop7                       7:7    0 227.6M  1 loop  /snap/thunderbird/1057
     sda                         8:0    0    64G  0 disk
     ├─sda1                      8:1    0     1M  0 part
     ├─sda2                      8:2    0     2G  0 part  /boot
     └─sda3                      8:3    0    62G  0 part
       └─ubuntu--vg-ubuntu--lv 252:0    0    31G  0 lvm   /
     sdb                         8:16   0    64G  0 disk
     └─md0                       9:0    0  63.9G  0 raid1 /data
     sdc                         8:32   0    64G  0 disk
     └─md0                       9:0    0  63.9G  0 raid1 /data
     sr0                        11:0    1   3.2G  0 rom
     ```

---

## Keamanan Data (RAID / Failure Handling)
1. Kelompok harus melakukan salah satu:
   - Mengimplementasikan RAID 1 (mirroring), atau -> Sudah diimplementasikan pada `Core Configuration`
   - Menyusun mekanisme cadangan data manual -> Sudah diimplementasikan pada `Core Configuration`
   - Mensimulasikan kegagalan disk
     - Bikin salah satu disk (misal sdb) jadi status 'fail' (rusak)
       ```
       sudo mdadm --manage /dev/md0 --fail /dev/sdb
       ```
    - Tunjukin ke dosen kalau statusnya jadi degraded [U_]
      ```
      cat /proc/mdstat
      ```
    - Copot disk yang rusak dari RAID
      ```
      sudo mdadm --manage /dev/md0 --remove /dev/sdb
      ```
   - Membuktikan sistem atau data tetap dapat diakses

2. Output wajib:
   - Demo kondisi normal & gagal
   - Penjelasan skenario pemulihan

---

## Monitoring Sistem
1. Kelompok harus:
   - Melakukan monitoring: CPU, RAM, Disk
     - Dapat menggunakan top
       ```
       top
       ```
     - Dapat menggunakan htop
       ```
       htop
       ```
     - Dapat menggunakan glances
       - Install `glances`
         ```
         sudo apt install glances -y
         ```
       - Jalankan `glances`
         ```
         glances
         ```
   -  Mensimulasikan beban tinggi
      - Stress-ng (The All-Rounder)
        - Install `stress-ng`
          ```
          sudo apt install stress-ng -y
          ```
        - Jalankan `stress-ng`
          ```
          stress-ng --cpu 4 --vm 1 --vm-bytes 1G --timeout 60s
          ```
      - Apache Benchmark / ab (Khusus Web Server)
        - Install `apache2-utils`
          ```
          sudo apt install apache2-utils -y
          ```
        - Jalankan `apache2-utils`
          ```
          ab -n 1000 -c 100 https://[Domain]/ <- https://web.zeroxx.my.id/
          ```
      - FIO - Flexible I/O Tester (Khusus RAID & Disk)
        - Install `fio`
          ```
          sudo apt install fio -y
          ```
        - Jalankan `fio`
          ```
          fio --name=test --directory=/data --rw=write --bs=4k --size=1G --numjobs=4 --runtime=30 --group_reporting
          ```
   -  Mencatat perubahan performa -> Catat semua perubahan peforma

2. Output wajib:
   - Screenshot monitoring -> Screenshot `top`, `htop`, `glances` sembari simulasi beban tinggi.
   - Analisis singkat (bukan hanya hasil) -> Analisa perubahan yang terjadi setelah simulai beban

---

## Update Sistem & Keamanan Dasar
1. Kelompok harus: 
   - Melakukan update sistem operasi -> Otomatis berjalan setelah startup
   - Mengaktifkan firewall -> Menggunakan nftables
   - Menutup minimal 1 port -> Menutup port telnet (port 23)

2. Output wajib:
   - Screenshot status update
     ```
     journalctl -u startup-update.service
     ``` 
   - Status firewall aktif

---

## Layanan Tambahan
1. File Sharing / Server Service
   - Membuat file sharing sederhana
   - Mengatur hak akses user
   - Menguji akses antar user

2. Output wajib:
   - Demo layanan berjalan
   - Penjelasan arsitektur

--

# NOTES
- [Panduan Praktikum Sistem Operasi Mendalam](https://gemini.google.com/share/13ee7358670b)
- [Linux file permissions explained](https://www.redhat.com/en/blog/linux-file-permissions-explained)
- [Linux permissions: SUID, SGID, and sticky bit](https://www.redhat.com/en/blog/suid-sgid-sticky-bit)