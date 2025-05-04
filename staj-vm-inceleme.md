# CentOS 7 Sistem Düzeltme Raporu

**SİSTEME ERİŞİM**

Root Erişimi Sağlamak (rd.break ile)

Sanal makinede root şifresi bilinmiyordu. İpucu olarak “erişim bilgilerini bilmediğiniz bir sunucuya nasıl erişirsiniz” denildiği için GRUB üzerinden rd.break komutunu kullanarak root şifresini sıfırladık ve sisteme erişim sağladık.

Root Şifresini Sıfırlamak için İzlenen Adımlar:

## Root Şifresini Sıfırlamak için Aşağıdaki  Adımları Takip Edecez : 

**1. VM’i başlatıyoruz.**
![image](https://github.com/user-attachments/assets/fffe6e73-6765-4361-a78c-362c09fb5073)

**2. GRUB ekranı geldiğinde klavyeden `e` tuşuna basıyoruz.**

**3. `linux16` veya `linux` ile başlayan satırını buluyoruz.**

![image](https://github.com/user-attachments/assets/f48ddeb8-b427-4c59-8c46-11cf0a5fde17)

**4. Satırın sonuna şu komutu ekliyoruz:**

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

1)ip a ve ip addr show enp0s3 ile ağ arayüzü durumu kontrol edildi.

2)cat /etc/resolv.conf ile DNS ayarları incelendi, ancak bağlantı sağlanamadı.

3)chattr -i /etc/resolv.conf ile dosya korumasını kaldırıp nano /etc/resolv.conf ile manuel DNS eklendi (neden: otomatik DNS yapılandırması çalışmıyordu).

4)dhclient -r enp0s3 ve dhclient enp0s3 ile DHCP üzerinden IP alındı.

5)nano /etc/sysconfig/network-scripts/ifcfg-enp0s3 ile yapılandırma düzenlendi (BOOTPROTO=dhcp, ONBOOT=yes).

6)systemctl restart network ile ağ servisi yeniden başlatıldı.

7)nmcli con mod enp0s3 ipv4.method manual ipv4.addresses 192.168.56.1 ipv4.dns "8.8.8.8 8.8.4.4" denendi, ancak DHCP tercih edildi.

8)systemctl stop NetworkManager ile çakışan NetworkManager devre dışı bırakıldı.

9)systemctl enable network ve chkconfig network on ile ağ servisi önyüklemede aktif hale getirildi.

10)dracut -f ile initramfs güncellendi (neden: önyükleme sırasında ağ yapılandırmalarının doğru yüklenmesini sağlamak).

11)reboot sonrası ping -c 2 8.8.8.8 ve ping google.com ile bağlantı doğrulandı (0% paket kaybı).

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
Not : Disk boyutuna başta ekleme yaptım sonrasında diskleri birleştirip hataları düzelttim

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
![image](https://github.com/user-attachments/assets/cc4c42a8-d490-4878-91fc-1d35e69cb628)

![image](https://github.com/user-attachments/assets/de20cc6a-084b-4d89-b68b-d0bc9dee79eb)


**Not: Arka planın değişme sebebi, işlemleri kendi lokal bilgisayarımda gerçekleştirmem ve bu sırada sunucuya bağlanmamdır. İlerleyen adımlarda sunucuya nasıl bağlandığımı detaylı olarak açıklayacağım**

# 4 .Yum Hatası ve Paket Kurulum Problemlerinin Giderilmesi

**Sorun:** Sistemde yum komutu çalıştırıldığında kilitlenme, repo erişim sorunları ve eksik paket problemleri (örneğin wget olmaması) tespit edilmiştir. Bu nedenle hem yum süreçlerinin düzgün çalışması hem de eksik paketlerin kurulumu için iki aşamalı bir çözüm süreci uygulanmıştır.



### Aşama 1: Yum Hatası ve Repo Problemlerinin Çözülmesi : 

1)ps aux | grep yum komutuyla çalışan yum süreçleri tespit edildi.

2)kill -9 <PID> ve rm -f /var/run/yum.pid ile kilit çözüldü (neden: yum işlemini serbest bırakmak).

3)yum check ile tutarlılık kontrol edildi.

4)mkdir /etc/yum.repos.d/backup ve mv /etc/yum.repos.d/CentOS-* /etc/yum.repos.d/backup/ ile eski repolar yedeklendi.

5)nano /etc/yum.repos.d/CentOS-Vault.repo ile vault.centos.org tabanlı bir repo oluşturuldu.

6)wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirror.centos.org/centos/7/os/x86_64/CentOS-Base.repo ile alternatif repo denendi.

7)yum clean all ve yum makecache ile önbellek güncellendi.

8)yum-config-manager --save --setopt=base.skip_if_unavailable=true ile erişilemeyen repolar atlandı (neden: hata vermesini önlemek).

**Kullanılan Komutlar :**

```
ps aux | grep yum
kill -9 6846 7104 7552
rm -f /var/run/yum.pid
yum check
mkdir /etc/yum.repos.d/backup
mv /etc/yum.repos.d/CentOS-* /etc/yum.repos.d/backup/
nano /etc/yum.repos.d/CentOS-Vault.repo
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirror.centos.org/centos/7/os/x86_64/CentOS-Base.repo
yum clean all
yum makecache
yum-config-manager --save --setopt=base.skip_if_unavailable=true
yum-config-manager --save --setopt=updates.skip_if_unavailable=true
yum-config-manager --save --setopt=extras.skip_if_unavailable=true
```

![image](https://github.com/user-attachments/assets/39ff45d8-9386-4815-8159-079a604830ec)


### Aşama 2: Wget Eksikliği ve EPEL Deposu Sorunlarının Giderilmesi

**Not: Gerekli bazı paketler standart CentOS repolarında bulunamadığı veya hatalı olduğu için,
https://vault.centos.org/7.9.2009/isos/x86_64/ adresinden CentOS 7.9.2009 ISO dosyası indirilmiş ve eksik paketler buradan manuel olarak alınmıştır.**


### Çözüm Adımları : 

1)mount -o loop /dev/sr1 /mnt/iso ile ISO bağlandı.

2)find /mnt/iso -CentOS-7-x86_64-DVD-2009.iso "wget*.rpm" ile wget-1.14-18.el7_6.1.x86_64.rpm bulundu.

3)rpm -ivh --force /mnt/iso/Packages/wget-1.14-18.el7_6.1.x86_64.rpm ile kuruldu.

4)EPEL için wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm denendi, ancak 404 hatası alındı.

5)echo "precedence ::ffff:0:0/96 100" >> /etc/gai.conf ile IPv4 önceliği eklendi (neden: IPv6 kaynaklı sorunları çözmek).

6)wget http://mirror.centos.org/centos/7/extras/x86_64/Packages/epel-release-7-11.noarch.rpm de başarısız oldu

7)Yerel kaynaktan rpm -ivh epel-release-7-11.noarch.rpm ile EPEL manuel kuruldu.

8)yum --enablerepo=epel makecache ile önbellek güncellendi


# 5. Logrotate ve RPM Veritabanı

**Sorun:** Log rotasyon hataları ve RPM veritabanı bozulması.

### Çözüm Adımları:

1)touch /var/log/boot.log, chmod 600, chown root:root ile log dosyası oluşturuldu.

2)nano /etc/logrotate.d/syslog ve nano /etc/logrotate.conf ile yapılandırma düzenlendi.

3)logrotate -f /etc/logrotate.conf ile rotasyon test edildi.

4)rm -f /var/log/boot.log-* ile eski loglar silindi.

5)rm -f /var/lib/rpm/__db* ve rpm --initdb ile RPM veritabanı onarıldı.

6)find /mnt/iso -CentOS-7-x86_64-DVD-2009.iso "*db*.rpm" ve rpm -ivh --force /mnt/iso/Packages/libdb-5.3.21-25.el7.x86_64.rpm ile eksik bağımlılıklar kuruldu.

7)find /mnt/iso -CentOS-7-x86_64-DVD-2009.iso "glibc*.rpm" ile glibc paketleri bulundu, rpm -ivh --force ile kuruldu.

**Kullanılan Komutlar :**

```
touch /var/log/boot.log
chmod 600 /var/log/boot.log
chown root:root /var/log/boot.log
nano /etc/logrotate.d/syslog
nano /etc/logrotate.conf
logrotate -f /etc/logrotate.conf
rm -f /var/log/boot.log-*
rm -f /var/lib/rpm/__db*
rpm --initdb
find /mnt/iso -CentOS-7-x86_64-DVD-2009.iso "*db*.rpm"
rpm -ivh --force /mnt/iso/Packages/libdb-5.3.21-25.el7.x86_64.rpm
find /mnt/iso -CentOS-7-x86_64-DVD-2009.iso "glibc*.rpm"
rpm -ivh --force /mnt/iso/Packages/glibc-2.17-317.el7.x86_64.rpm /mnt/iso/Packages/glibc-common-2.17-317.el7.x86_64.rpm

```

![image](https://github.com/user-attachments/assets/5ba71327-c7f0-479c-b4a8-41d8bc75dc03)

# 6. Apache Kurulum ve Test

**Sorun:** Apache servisi çalışmıyor, web sayfası hedefi tamamlanmamıştı.

### Çözüm Adımları:

1)yum install -y httpd ile Apache kuruldu.

2)systemctl start httpd ve systemctl enable httpd ile servis başlatıldı ve kalıcı hale getirildi.

3)chown apache:apache /var/www/html/index.html ve chmod 644 /var/www/html/index.html ile izinler düzenlendi.

4)firewall-cmd --permanent --add-service=http ve --add-service=https ile HTTP/HTTPS portları açıldı.

5)firewall-cmd --reload ile firewall güncellendi.

6)Kendi bilgisayardan http://192.168.227.72 ile bağlanıldı, mesaj doğrulandı.

7)Sunucu yarım saat çalıştırıldı, reboot sonrası tekrar kontrol edildi, sayfa hala görünüyordu.

**Kullanılan Komutlar :**

```
yum install -y httpd
systemctl start httpd
systemctl enable httpd
chown apache:apache /var/www/html/index.html
chmod 644 /var/www/html/index.html
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
systemctl restart httpd
reboot

```

![image](https://github.com/user-attachments/assets/3af45529-4ef3-493f-a974-dbf9fde0a685)

![image](https://github.com/user-attachments/assets/4526c91d-42ef-40ae-bb7a-2b4fdc06ac0b)


# 8. Güvenlik ve Sistem Kontrolü

**Adımlar:**

1)yum install -y rkhunter ve yum install -y chkrootkit ile güvenlik araçları kuruldu.

2)rkhunter --update, rkhunter --propupd, rkhunter --check ile sistem tarandı.

3)top ile sistem kaynakları izlendi.

**Kullanılan Komutlar :**

```
yum install -y rkhunter
yum install -y chkrootkit
rkhunter --update
rkhunter --propupd
rkhunter --check
top

```

![image](https://github.com/user-attachments/assets/6f169db4-f648-4ff2-934e-40bf655ba555)

![image](https://github.com/user-attachments/assets/84e5ae80-dc88-4970-b47e-1c5826a4d776)


# Son Durum

**Ağ Bağlantısı: Reboot sonrası kalıcı hale geldi.**

**Disk Kullanımı: %7, temizlik ile kontrol altında.**

**Apache: Çalışıyor, mevcut /var/www/html/index.html "Apache çalışıyor, hem de son sürümde :)" gösteriyor.**

**EPEL: Yerel kaynaktan kuruldu.**

**Yedekleme: /root/backup dizininde tamamlandı.**

**Güvenlik: rkhunter ve chkrootkit ile tarama yapıldı, /opt/.script1 silindi, cron temizlendi, /opt sıkılaştırıldı.**


