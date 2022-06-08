---
layout: post
title:  "Arch Linux'da bir garip Bluetooth & Wi-Fi problemi"
date:   2022-06-08 23:06:35 +0200
author: Cem Özden
---
İş dışında kullandığım kişisel bilgisayarımda yaklaşık 7 senedir Linux kullanıyorum. Bu süreçte bir çok tuhaf problemlerle karşılaşmıştım ama karşılaştığım en ilginç sorunlardan birisinide sanırım geçen hafta karşılaşmış oldum. :=)

Geçen hafta sonu yaptığım system upgrade ile Linux Kernel 5.18'e geçtim. Genelde bu tür geçişlerde içim ürperiyor çünkü kullandığım Intel WLAN kartı görece yeni bir kart ve WLAN driver görece oldukça yeni sayılabilecek driverlardan birisi. Özellikle linux upgrade yaptıktan sonra driverda bazı sapıtmalar ile karşılaşabiliyorum ve upgrade yaptıktan sonrasında da korktuğum şey başıma geldi. Bir anda internet bağlantımda ciddi bir packet drop sorunları ve bağlantı kopmaları yaşamaya başladım.

Tabiki hemen incelemeye başladım ve öncellikle hemen *pacman* ile kernel downgrade yapıp 5.17'ye geçtim fakat sorunun düzelmediğini aslında sorunun kernel upgrade ile alakasının olmadığını gördüm. Daha sonrasında sorunun kendince düzeldiğini farkedince bir "ne oluyor kardeşim?" tepkisi doğal olarak verdim. Bir gün sonrasında sorunun tekrar ettiğini Spotify üzerinden müzik dinlerken farkettim ve yine sistemi incelemeye karar verdim ve kernel module durumlarına bakarken bir çözüm yolu bulamadım.

1 gün sonrasına sorunun tekrardan kendi kendine düzeldiğini gördüm. 1-2 saat sonrasına YouTube üzerinden bir videoyu Bluetooth kulaklığım ile dinlemeye karar verince internet bağlantımda yine sıkıntılar yaşadığımı farkedince "acaba Bluetooth benim internetimi mi etkiliyor?" diye düşünmeye başladım. İlk başta herhangi bir kolerasyon kuramadım her iki bağlantı arasında fakat daha sonrasında İnternette araştırma yapmaya başlayınca aslında Bluetooth ve Intel Wi-Fi'ın kendi aralarında ***Radio Conflict*** yaşadıklarını öğrendim. (Conflict ile alakalı detaylı bilgileri bu [link](https://superuser.com/a/1312013)'te bulabilirsiniz.)

Çözüm olarak **iwlwifi** driver için aşağıdaki konfigurasyonu `/etc/modprobe.d/iwlwifi.conf` dosyasına yazıp, iwlwifi kernel modülünü yeniden yükleyip veya makinenizi yeniden başlatıp sorunu çözebilirsiniz.

```config
options iwlwifi bt_coex_active=0 swcrypto=1 11n_disable=8
```