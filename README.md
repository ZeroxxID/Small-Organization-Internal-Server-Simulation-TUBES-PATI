# **SMALL ORGANIZATION INTERNAL SERVER SIMULATION GUIDE**
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
   - Name: PATI Kelompok 4
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

### AUTOMATION CONFIGURATION
#### Membuat Service Startup Update
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

#### Otomasi Reboot Pada 00.00
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
