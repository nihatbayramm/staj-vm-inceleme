# CentOS 7 Sanal Makine Düzeltme ve Yedekleme Projesi

## Proje Amacı
Bu proje, bir CentOS 7 sanal makinesinde karşılaşılan çeşitli sistem sorunlarını çözmeyi ve ardından sistemin son halindeki `/etc` ve `/root` dizinlerini paketleyerek paylaşmayı hedeflemektedir. Apache web sunucusunun çalışır hale getirilmesi, sistemin stabil bir duruma getirilmesi ve tüm işlemlerin belgelenmesi amaçlanmıştır.

## Karşılaşılan Sorunlar ve Çözümler

### 1. Root Erişimi
- **Sorun**: Sanal makineye root erişimi için şifre bilinmiyordu.
- **Çözüm**: GRUB üzerinden `rd.break` kullanılarak root şifresi sıfırlandı.

### 2. Ağ Bağlantı Sorunları
- **Sorun**: Reboot sonrası ağ bağlantısı kayboluyordu (`enp0s3` arayüzü IP alamıyordu).
- **Çözüm**: 
  - `/etc/sysconfig/network-scripts/ifcfg-enp0s3` dosyasında `BOOTPROTO=dhcp` ve `ONBOOT=yes` ayarları yapılarak ağ bağlantısı kalıcı hale getirildi.
  - `NetworkManager` devre dışı bırakıldı, `dracut -f` ile `initramfs` güncellendi.
- **Sonuç**: Reboot sonrası ağ bağlantısı stabil hale geldi.

### 3. Disk Doluluğu
- **Sorun**: Disk %100 doluydu (`/dev/mapper/centos-root`).
- **Çözüm**:
  - `/var/tmp/*.bin` dosyaları ve eski loglar (`/var/log/`) temizlendi.
  - Disk kullanımı %7'e düşürüldü.

### 4. Yum Sorunları
- **Sorun**: Yum kilitlenmişti ve CentOS depolarına erişim sağlanamıyordu (EOL nedeniyle).
- **Çözüm**:
  - Yum kilidi çözüldü (`kill -9` ve `rm -f /var/run/yum.pid`).
  - `vault.centos.org` tabanlı repolar eklendi (`/etc/yum.repos.d/CentOS-Vault.repo`).
  - `skip_if_unavailable=true` ayarı yapılarak erişim sorunları giderildi.
- **Sonuç**: Yum çalışır hale getirildi.

### 5. Cron ve Script Temizliği
- **Sorun**: `stress.service` ve `/opt/.script1` gibi zararlı script’ler sistemi zorluyordu.
- **Çözüm**:
  - `stress.service` devre dışı bırakıldı, `/opt/.script1` silindi.
  - `/etc/crontab` temizlendi ve `chattr +i` ile korundu.
  - `/opt` dizini `chmod 700` ile sıkılaştırıldı.
- **Sonuç**: Sistem kaynakları serbest bırakıldı.

### 6. Apache Kurulumu ve Test
- **Sorun**: Apache servisi çalışmıyordu.
- **Çözüm**:
  - `httpd` paketi kuruldu, servis başlatıldı ve kalıcı hale getirildi.
  - Mevcut `/var/www/html/index.html` dosyası doğrulandı ("Apache çalışıyor, hem de son sürümde :)").
  - Firewall’da HTTP/HTTPS portları açıldı, reboot sonrası erişim doğrulandı.
- **Sonuç**: Apache stabil bir şekilde çalışıyor.

### 7. Yedekleme
- **İşlem**:
  - `/etc` ve `/root` dizinleri, sanal makinenin son halini yansıtacak şekilde `final_backup.tar.gz` dosyasına paketlendi.
  - Bu dosya, GitHub reposuna yüklenmiştir.

## Son Durum
- **Ağ Bağlantısı**: Reboot sonrası stabil.
- **Disk Kullanımı**: %51, kontrol altında.
- **Yum**: Çalışır durumda.
- **Apache**: `/var/www/html/index.html` hedef mesajı gösteriyor, erişim stabil.
- **Yedekleme**: `/etc` ve `/root` dizinleri `final_backup.tar.gz` dosyasında mevcut.

## Ekler
- **final_backup.tar.gz**: Sanal makinenin son halindeki `/etc` ve `/root` dizinlerini içerir.
  - Örnek dosyalar:
    - `/etc/sysconfig/network-scripts/ifcfg-enp0s3`: Ağ yapılandırması.
    - `/etc/yum.conf` ve `/etc/yum.repos.d/`: Yum ayarları.
    - `/root/backup/index.html`: Apache durumu.

## Notlar
- Tüm işlemler, sistemin stabil hale getirilmesi ve hedeflerin tamamlanması için yapılmıştır.
- Rapor, yalnızca işlemleri özetler ve hassas bilgiler paylaşmaz.

Hazırlayan: **Nihat Bayram**  
Tarih: **03 Mayıs 2025**
