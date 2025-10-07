# Ubuntu

nano /etc/ssh/sshd_config

#PermitRootLogin prohibit-password

PermitRootLogin yes

passwd

1️⃣ Disable konfigurasi network cloud-init
Buat file baru:
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
Isi file:
network: {config: disabled}
Simpan dan keluar (Ctrl+O, Enter, Ctrl+X).
Ini supaya cloud-init tidak lagi menimpa file netplan saat reboot.
2️⃣ Buat file netplan baru untuk IP statis
Buat file baru, misal 01-netcfg.yaml:
sudo nano /etc/netplan/01-netcfg.yaml
Isi:
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.0.88/24
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: 0.0.0.0/0
          via: 192.168.0.1
⚠️ Catatan:
renderer: networkd → lebih stabil untuk server.
Gunakan routes alih-alih gateway4.
Jangan ada konfigurasi IP ganda atau default route lain.
3️⃣ Perbaiki permission file netplan
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo chown root:root /etc/netplan/01-netcfg.yaml
Ini supaya netplan tidak mengeluh lagi tentang permissions.
4️⃣ Terapkan konfigurasi
sudo netplan generate
sudo netplan apply
Cek IP:
ip addr show ens18
ip route
Seharusnya sekarang IP statis 192.168.0.88 aktif dan default route ke 192.168.0.1 sudah benar.
5️⃣ Reboot untuk memastikan permanen
sudo reboot

