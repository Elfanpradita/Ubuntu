Ubuntu 22.04 dengan cloud-init.

# Panduan Setup IP Statis Permanen di Ubuntu 22.04 (Cloud-init)

Dokumen ini menjelaskan langkah-langkah untuk **mengatur IP statis permanen** di Ubuntu 22.04, termasuk menonaktifkan cloud-init agar konfigurasi tidak hilang setelah reboot. Panduan ini juga memastikan IP lama (secondary) dinonaktifkan.

---

## **1. Cek IP saat ini**

```bash
ip addr show ens18
````

Periksa apakah ada IP primary dan secondary. Misal:

```
inet 192.168.0.158/24  # Primary
inet 192.168.0.88/24   # Secondary (lama)
```

---

## **2. Nonaktifkan cloud-init untuk network**

Buat file disable network cloud-init:

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

Isi:

```yaml
network: {config: disabled}
```

Simpan (`Ctrl+O`, `Enter`) dan keluar (`Ctrl+X`).

---

## **3. Bersihkan konfigurasi network cloud-init lama**

Hapus file netplan lama dan cache cloud-init:

```bash
sudo rm -f /etc/netplan/50-cloud-init.yaml
sudo cloud-init clean --logs
sudo cloud-init clean --network
```

---

## **4. Buat file netplan permanen**

Buat file netplan baru:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Isi file:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.0.158/24
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: 0.0.0.0/0
          via: 192.168.0.1
```

> ⚠️ Pastikan **IP lama (192.168.0.88)** dihapus dari konfigurasi.

---

## **5. Set permission file netplan**

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo chown root:root /etc/netplan/01-netcfg.yaml
```

Ini mencegah warning permission saat `netplan apply`.

---

## **6. Terapkan netplan**

```bash
sudo netplan generate
sudo netplan apply
```

Cek IP dan routing:

```bash
ip addr show ens18
ip route
```

* Pastikan hanya **192.168.0.158** yang aktif
* Default route via **192.168.0.1**

---

## **7. Nonaktifkan IP lama sementara (jika masih muncul)**

```bash
sudo ip addr del 192.168.0.88/24 dev ens18
```

---

## **8. Reboot untuk memastikan permanen**

```bash
sudo reboot
```

Setelah boot, cek lagi:

```bash
ip addr show ens18
ip route
```

* IP lama **192.168.0.88** tidak muncul
* IP **192.168.0.158** jadi primary dan permanen

---

## **9. Tes konektivitas**

```bash
ping 8.8.8.8
ping google.com
```

Jika lancar, server sudah **stabil dengan IP statis permanen**.

---

## **Catatan**

* Pastikan **cloud-init network sudah dinonaktifkan**, kalau tidak, reboot berikutnya bisa menimpa IP statis.
* Gunakan `networkd` sebagai renderer untuk server stabil.
* Untuk multiple IP atau routing kompleks, gunakan **routing-policy** di netplan sesuai dokumentasi resmi.

```

---

Kalau mau, aku bisa buatkan **versi script shell siap run 1x**, yang otomatis:

1. Nonaktifkan cloud-init network  
2. Bersihkan cache lama  
3. Buat netplan permanen untuk IP 158  
4. Terapkan dan cek  

Sehingga tinggal copy-paste di server baru, langsung permanen.  

Apakah mau aku buatkan versi script itu juga?
```
