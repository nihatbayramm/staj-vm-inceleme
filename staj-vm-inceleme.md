### CentOS 7 Sistem Düzeltme Raporu

**SİSTEME ERİŞİM**

# Root Erişimi Sağlamak (rd.break ile)

Sanal makinede root şifresi bilinmiyor. İpucu olarak “erişim bilgilerini bilmediğiniz bir sunucuya nasıl erişirsiniz” denildiği için GRUB üzerinden `rd.break` komutunu kullanarak root şifresini sıfırlayabiliriz.

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


# Sunucu Sorunları 

# İnternet erişim sorunu

### Geçici ve Kalıcı IP Ayarları

İnternete bağlanmak için geçici olarak IP ataması yapılabilir. Ancak, sunucu her yeniden başlatıldığında internet bağlantısı kaybolur ve komutların tekrar girilmesi gerekir. Ayarların kalıcı olabilmesi için şu adımları izlemeliyiz:

1. **Geçici Ayarlar**  
   İlk adımda, geçici yani RAM'de saklanan ayarları yapacağım. Bu ayarlarla sunucuyu geçici olarak internete bağlayacağız.

2. **Sunucu Depolama Alanı Genişletme**  
   Ayarların kalıcı olabilmesi için, sunucunun depolama alanını genişletmemiz gerekiyor. Eğer depolama alanı yeterli değilse, gereksiz dosyaları silerek alan yaratabiliriz.

3. **Kalıcı Ayarların Yapılması**  
   Depolama alanını genişlettikten sonra, kalıcı ayarları yaparak IP atamalarını ve diğer ağ ayarlarını sunucunun diskine kaydedeceğiz.

Bu adımları takip ederek, sunucunun her yeniden başlatılmasında internet bağlantısının kaybolmasını engelleyebiliriz.



# Geçici Ayarlar

**ip adresi var fakat ping atıldığı zaman internet erişiminin olmadığı görülüyor bunun için aşağıdaki adımlar izlenmelidir :** 

![image](https://github.com/user-attachments/assets/a89f2bdf-1f4d-4387-92e6-2756ca6546fb)

**1. DNS sunucusu kontrolü**
```
cat /etc/resolv.conf
```
- nameserver 8.8.8.7 yanlış bir DNS adresi — Google'ın doğru DNS adresi 8.8.8.8 veya 8.8.4.4 olmalı. 8.8.8.7 kullanılmaz ve internet erişimini bozuyordur.
- Dns adresini değiştirirken resolv.conf dosyası değiştirelemez olarak ayarlanmış bunu değiştirmek için resolv.conf Dosyasından Immutable Özelliğini Kaldıracaz.
![image](https://github.com/user-attachments/assets/079bfba7-0d4e-4623-a7a4-dc43987d165c)

**İmmutable özelliğini kaldırmak için aşağı komutu girecez :**

```
chattr -i /etc/resolv.conf

```
![image](https://github.com/user-attachments/assets/69996b5f-2cac-4e62-8926-0c6fd6e403ae)

**Dns sunucusunu nano ile değiştirip  tekrar konrtol ediyoruz :**

![image](https://github.com/user-attachments/assets/84e5b546-b073-4f2d-b6cb-aac15e176a4b)

**-Ping attığımızda hala sorun olduğu gözüküyor :**

![image](https://github.com/user-attachments/assets/072f0c03-1d92-4269-b61e-0e5224c02554)

**Sorunu çözmek için mevcut ip ardesini serbest bırakıp yeni bir ip talep edecez :**

```
dhclient -r enp0s3  # Mevcut IP'yi serbest bırak
dhclient enp0s3    # Yeni ip talep et
```
![image](https://github.com/user-attachments/assets/851e9123-5898-4cf7-94d6-4bdce31234fc)

**Tekrar ping attığımız zaman internete bağlandığımızı görebiliriz :**

![image](https://github.com/user-attachments/assets/84328b7c-a351-4489-ac25-89caf761ab29)


### Sunucu Depolama Alanı Genişletme :

**VirtualBox Üzerinde Disk Alanını Genişletme**

1) VirtualBox → Settings → Storage → .vdi dosyasına tıkla

2) Sağ alt köşede “Size” kısmından kaydırarak artırabiliriz

![image](https://github.com/user-attachments/assets/86a41d6e-5283-4a63-b467-06f34fd43332)

![image](https://github.com/user-attachments/assets/fa7a2a61-c97c-4c88-adee-78a1ef1621a3)

![image](https://github.com/user-attachments/assets/2a376086-0a32-4588-91ee-26542d144309)

![image](https://github.com/user-attachments/assets/1325b805-44a4-4247-8230-e7df46b1c847)

# CentOS Root Alanını Genişletme (LVM Kullanarak)
Sanal disk büyütüldükten sonra CentOS’ta aşağıdaki işlemleri yaparak root alanını genişletebiliriz:

**1. Yeni Alanı Tanımlama ve VG’ye Eklemek için aşağıdaki komutları kullanacaz :**

```
lsblk          # Yeni diski/bölümü tespit etmek için (örneğin /dev/sdb)
pvcreate /dev/sdb
vgextend centos /dev/sdb

```
![image](https://github.com/user-attachments/assets/5fa26d28-4550-44af-9354-cd4048023796)


**2. Root Logical Volume Genişletme**

**Tüm boş alanı eklemek için aşağıdaki komutları kullanıyoruz :**

```
lvextend -l +100%FREE /dev/centos/root
```
![image](https://github.com/user-attachments/assets/40b8c10b-def8-4a3f-96ce-8c4e8b3cee03)

**3. Dosya Sistemini Genişletme (XFS için) aşağıdaki kodu kullanmalıyız :**

```
xfs_growfs /dev/centos/root
```

![image](https://github.com/user-attachments/assets/de61505d-bcde-4a84-a510-389ef175c5bf)

**4. Son Kontrolü yapmak içi aşağıdaki komutu yazığ kontrol edecez :**
```
df -h
```

![image](https://github.com/user-attachments/assets/7247c26d-a958-4520-8e0e-e8e972f55111)

**Artık / bölümü genişledi.**

### HATALAR VE DUZELTMELER :

**1)Ram'ı zorlayan stress paketi kapatıldı




























