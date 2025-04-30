# Root Erişimi Sağlamak (rd.break ile)

Sanal makinede root şifresi bilinmiyor. İpucu olarak “erişim bilgilerini bilmediğiniz bir sunucuya nasıl erişirsiniz” denildiği için GRUB üzerinden `rd.break` komutunu kullanarak root şifresini sıfırlayabiliriz.

## 🔐 Root Şifresini Sıfırlama Adımları (CentOS)

1. VM’i başlat.
2. GRUB ekranı geldiğinde klavyeden `e` tuşuna bas.
3. `linux16` veya `linux` ile başlayan satırı bul.
4. Satırın sonuna şu komutu ekle:

``` 
rd.break
```
https://github.com/nihatbayramm/staj-vm-inceleme/blob/main/image/Screenshot%20from%202025-04-30%2013-13-23.png?raw=true

