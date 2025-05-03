# CentOS 7 Sistem Düzeltme Raporu

**SİSTEME ERİŞİM**

Root Erişimi Sağlamak (rd.break ile)

Sanal makinede root şifresi bilinmiyordu. İpucu olarak “erişim bilgilerini bilmediğiniz bir sunucuya nasıl erişirsiniz” denildiği için GRUB üzerinden rd.break komutunu kullanarak root şifresini sıfırladık ve sisteme erişim sağladık.

Root Şifresini Sıfırlamak için İzlenen Adımlar:

## Root Şifresini Sıfırlamak için Aşağıdaki  Adımları Takip Edecez : 

**1. VM’i başlatıyoruz.**
![image](https://github.com/user-attachments/assets/fffe6e73-6765-4361-a78c-362c09fb5073)

**3. GRUB ekranı geldiğinde klavyeden `e` tuşuna basıyoruz.**
**5. `linux16` veya `linux` ile başlayan satırını buluyoruz.**
![image](https://github.com/user-attachments/assets/f48ddeb8-b427-4c59-8c46-11cf0a5fde17)

**7. Satırın sonuna şu komutu ekliyoruz:**

``` 
rd.break
```
![image](https://github.com/user-attachments/assets/13e54ba3-f5bb-4378-bc59-a5bd08f49a8e)

**5. `Ctrl + X` tuşlarına basarak sistemi bu modda başlatıyoruz.**

## Dracut Shell Üzerinden Root Erişimi

**6. Sistem `switch_root:/#` gibi bir satırla açılacaktır.**
**7. Root dosya sistemini yeniden bağlıyoruz:**
```
mount -o remount,rw /sysroot
chroot /sysroot
```
![image](https://github.com/user-attachments/assets/b2adec0b-1f33-4b31-986a-7e1398c8dd89)

![image](https://github.com/user-attachments/assets/dd85fa16-59d5-4c03-a820-75bce2d6dfc6)

**8.Root şifresini sıfırlıyoruz:**

```
passwd
```
![image](https://github.com/user-attachments/assets/6bfb995b-b484-45fa-aacd-0ccf83a0a8df)

**9.SELinux tekrar etiketleme yapabilsin diye aşağıdaki komutu giriyoruz :**

```
touch /.autorelabel

```

![image](https://github.com/user-attachments/assets/d576c063-55e6-4031-9f39-3597eaa4f22a)

**10.exit yaparak giriş ekranına geliyoruz sonrasında kullanııcı adı olarak root yazıp belirlediğimiz şifreyi girip sisteme giriş yapıyoruz:**

![image](https://github.com/user-attachments/assets/bd194adb-ec03-4df0-9b51-f1bbc40e7696)

![image](https://github.com/user-attachments/assets/46d11ac0-f2af-4bfa-aa2a-d6e10e0b4d01)


# Karşılaşılan Sorunlar ve Çözüm Süreci
**1. Ağ Bağlantı Sorunları**

Sorun: Reboot sonrası internet bağlantısı kayboluyordu. enp0s3 arayüzü IP alamıyordu.

Çözüm Adımları:

-ip a ve ip addr show enp0s3 ile ağ arayüzü durumu kontrol edildi.

-cat /etc/resolv.conf ile DNS ayarları incelendi, ancak bağlantı sağlanamadı.

-chattr -i /etc/resolv.conf ile dosya korumasını kaldırıp nano /etc/resolv.conf ile manuel DNS eklendi (neden: otomatik DNS yapılandırması çalışmıyordu).

-dhclient -r enp0s3 ve dhclient enp0s3 ile DHCP üzerinden IP alındı.

-nano /etc/sysconfig/network-scripts/ifcfg-enp0s3 ile yapılandırma düzenlendi (BOOTPROTO=dhcp, ONBOOT=yes).

-systemctl restart network ile ağ servisi yeniden başlatıldı.

-nmcli con mod enp0s3 ipv4.method manual ipv4.addresses 192.168.56.1 ipv4.dns "8.8.8.8 8.8.4.4" denendi, ancak DHCP tercih edildi.

-systemctl stop NetworkManager ile çakışan NetworkManager devre dışı bırakıldı.

-systemctl enable network ve chkconfig network on ile ağ servisi önyüklemede aktif hale getirildi.

-dracut -f ile initramfs güncellendi (neden: önyükleme sırasında ağ yapılandırmalarının doğru yüklenmesini sağlamak).

-reboot sonrası ping -c 2 8.8.8.8 ve ping google.com ile bağlantı doğrulandı (0% paket kaybı).

**Kullanılan Komutlar :**

```
ip a
ip addr show enp0s3
cat /etc/resolv.conf
chattr -i /etc/resolv.conf
nano /etc/resolv.conf
dhclient -r enp0s3
dhclient enp0s3
nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
systemctl restart network
nmcli con mod enp0s3 ipv4.method manual ipv4.addresses 192.168.56.1 ipv4.dns "8.8.8.8 8.8.4.4"
systemctl stop NetworkManager
systemctl enable network
chkconfig network on
dracut -f
reboot
ping -c 2 8.8.8.8
ping google.com

```

![image](https://github.com/user-attachments/assets/6800ffd7-134d-4e91-9e80-30491aad61ea)

![image](https://github.com/user-attachments/assets/a725f311-0b4f-44bb-8dd8-4b239e20bb01)



























