# Sanal Makine İncelemesi ve Güvenlik Analizi

Bu çalışmada verilen `.ova` uzantılı sanal makine imajı incelenmiş, sistemde tespit edilen sorunlar ve eksiklikler adım adım analiz edilmiştir. Aynı zamanda web servisi veren bir sunucuda bulunmaması gereken yapılandırmalar belirlenmiş ve gerekli düzeltmeler yapılmıştır.

## 🧩 Başlangıç

Sanal makine VirtualBox ortamına import edilerek başlatılmıştır.

### İlk Adım: Erişim Sağlama
- Sanal makine başlatıldıktan sonra kullanıcı adı ve parola ile giriş yapıldı.
- root kullanıcısının ev dizininde bulunan `README` dosyası incelendi.
- SSH, sudo ve root yetkileri test edildi.

## 🔍 İnceleme Adımları

1. **Ağ Yapılandırması Kontrolü**
   - `ip a` ve `netstat -tuln` komutları ile sistemin dinlediği portlar ve IP yapılandırması kontrol edildi.
   - Gereksiz açık portlar tespit edildi.

2. **Kullanıcı ve Yetki Analizi**
   - `sudo`, `su`, `/etc/passwd` ve `/etc/shadow` dosyaları kontrol edildi.
   - Şüpheli kullanıcılar ve zayıf parola ihtimalleri analiz edildi.

3. **Servis ve Daemon Kontrolleri**
   - `systemctl list-units --type=service` komutu ile çalışan servisler listelendi.
   - Gereksiz servisler belirlendi ve devre dışı bırakıldı.

4. **Güncelleme ve Paket Yönetimi**
   - `apt update && apt upgrade` komutları ile sistemin güncel olup olmadığı kontrol edildi.
   - Gereksiz veya potansiyel risk taşıyan paketler temizlendi.

5. **Güvenlik Duvarı ve Ağ Güvenliği**
   - `ufw status` ve `iptables -L` komutları ile güvenlik duvarı yapılandırması kontrol edildi.
   - Gerekli kurallar yazılarak dışa açık servisler sınırlandırıldı.

6. **Web Servisi Analizi**
   - `apache2` veya `nginx` gibi servislerin kurulu olup olmadığı ve yapılandırmaları incelendi.
   - Gerekliyse varsayılan sayfalar ve açıklayıcı bilgiler kaldırıldı.

## ✅ Sonuç ve Düzeltmeler

- Sistem üzerindeki temel güvenlik açıkları tespit edilip kapatıldı.
- Giriş bilgilerinin güvenliği artırıldı.
- Gereksiz servisler ve kullanıcılar sistemden kaldırıldı.
- Sistem daha güvenli, sade ve kontrol altında bir hale getirildi.

## 📎 Notlar

- Tüm yapılan işlemler terminal çıktıları ve ekran görüntüleri ile desteklenerek ayrı dosyalarda belge halinde sunulacaktır.
