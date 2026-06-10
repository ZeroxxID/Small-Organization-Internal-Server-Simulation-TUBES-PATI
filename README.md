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
