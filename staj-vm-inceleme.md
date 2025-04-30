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
![image](https://github.com/user-attachments/assets/13e54ba3-f5bb-4378-bc59-a5bd08f49a8e)

