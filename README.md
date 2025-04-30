# Sanal Makine İncelemesi ve Güvenlik Analizi (CentOS)

Bu projede verilen `.ova` uzantılı CentOS sanal makinesi incelenmiş; sistem yapılandırmaları, kullanıcı yetkileri ve servis durumları detaylı olarak analiz edilmiştir. Amaç, sunucu ortamında bulunmaması gereken yapılandırmaları tespit edip düzeltmektir.

## 🧩 Başlangıç

Sanal makine VirtualBox ortamına import edilip başlatıldıktan sonra kullanıcı girişleri başarıyla gerçekleştirildi. `root` dizininde yer alan `README` dosyası incelendi.

## 🔍 İnceleme ve Düzeltme Adımları

### 1. Ağ ve Port Kontrolü
- `ip a` ile ağ arayüzleri kontrol edildi.
- `ss -tuln` ve `netstat -tuln` komutları ile sistemde dinlenen portlar listelendi.
- Gereksiz açık portlar ve dışa açık servisler belirlendi.

### 2. Kullanıcı ve Yetki Analizi
- `/etc/passwd` ve `/etc/shadow` dosyaları incelenerek sistem kullanıcıları gözden geçirildi.
- `sudo` yetkileri ve `wheel` grubuna ait kullanıcılar kontrol edildi.
- Şüpheli kullanıcılar sistemden kaldırıldı.

### 3. Servis Yönetimi
- `systemctl list-units --type=service` ile aktif servisler görüntülendi.
- Gereksiz servisler `systemctl disable` ve `systemctl stop` komutlarıyla devre dışı bırakıldı.
- Özellikle dışa açık web, FTP ya da SSH servislerinin yapılandırmaları denetlendi.

### 4. Paket ve Güncelleme Yönetimi
- Sistem güncellemeleri `yum update -y` komutu ile yapıldı.
- Kullanılmayan veya riskli paketler `yum remove` ile sistemden temizlendi.

### 5. Güvenlik Duvarı (firewalld)
- `firewall-cmd --list-all` komutu ile mevcut kurallar incelendi.
- Gereksiz açık portlar kapatıldı, sadece gerekli servisler açık bırakıldı.
- firewalld aktif hale getirildi ve yeniden yüklendi:
  ```bash
  systemctl enable firewalld
  systemctl start firewalld
  firewall-cmd --reload
