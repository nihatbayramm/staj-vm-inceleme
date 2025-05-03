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


**2. Disk Doluluğu ve Temizlik**

Sorun: Disk %100 doluydu (/dev/mapper/centos-root), cron betikleri ve büyük dosyalar nedeniyle.

### Çözüm Adımları:

1)df -h ile disk durumu kontrol edildi.

2)du -ah / | sort -rh | head -n 20 ve du -sh /var/tmp/* ile büyük dosyalar (/var/tmp/*.bin) tespit edildi.

3)rm -f /var/tmp/*.bin ve rm -rf /var/tmp/* ile gereksiz dosyalar silindi.

4)rm -rf /var/log/*.log.* ve rm -rf /var/cache/* ile eski loglar ve önbellek temizlendi.

5)lsof +D /var/tmp ile açık dosyalar kontrol edildi (neden: disk alanını tamamen serbest bırakmak).


### Kullanılan Komutlar:

```
df -h
du -ah / | sort -rh | head -n 20
du -sh /var/tmp/*
rm -f /var/tmp/*.bin
rm -rf /var/tmp/*
truncate -s 0 /var/log/messages
truncate -s 0 /var/log/secure
rm -rf /var/log/*.log.*
rm -rf /var/cache/*
find /tmp -type f -exec rm -f {} \;
lsof +D /var/tmp

```

Disk kullanımı %100'den %7'e düşürüldü 

![image](https://github.com/user-attachments/assets/1a0abbfd-df44-4ca6-afef-d74d17109af7)


**3. stress.service ve Cron Betikleri**

Sorun: stress.service ve cron betikleri (/opt/.script1, /opt/.script2) kaynakları zorluyordu, disk doluluğuna neden oluyordu.

### Çözüm Adımları:

1)ps aux | grep stress ile çalışan süreçler kontrol edildi, pkill -9 stress ile sonlandırıldı.

2)systemctl stop stress.service ve systemctl disable stress.service ile servis devre dışı bırakıldı.

3)rm -f /etc/systemd/system/stress.service ve rm -f /usr/lib/systemd/system/stress.service ile servis dosyaları silindi.

4)crontab -l ve cat /etc/crontab ile cron betikleri kontrol edildi.

5)crontab -r ile kullanıcı cronları silindi, sed -i '/script1/d' /etc/crontab ile sistem cronundan betik kaldırıldı.

6)rm -f /opt/.script1 ile betik dosyası silindi (neden: zararlı script’i kaldırmak).

7)systemctl restart crond ile cron servisi yeniden başlatıldı (neden: değişikliklerin etkili olmasını sağlamak).

8)chmod 700 /opt ve chattr +i /etc/crontab ile güvenlik artırıldı (neden: yetkisiz erişimi ve tekrar oluşmasını önlemek).

**Kullanılan Komutlar :**
```
ps aux | grep stress
pkill -9 stress
systemctl stop stress.service
systemctl disable stress.service
rm -f /etc/systemd/system/stress.service
rm -f /usr/lib/systemd/system/stress.service
systemctl daemon-reload
crontab -l
cat /etc/crontab
crontab -r
sed -i '/script1/d' /etc/crontab
rm -f /opt/.script1
systemctl restart crond
chmod 700 /opt
chattr +i /etc/crontab
```

![image](https://github.com/user-attachments/assets/24bfa376-f626-413f-86e7-ff6d33f1e502)

### Not: Arka planın değişme sebebi, işlemleri kendi lokal bilgisayarımda gerçekleştirmem ve bu sırada sunucuya bağlanmamdır. İlerleyen adımlarda sunucuya nasıl bağlandığımı detaylı olarak açıklayacağım
