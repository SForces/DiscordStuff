
# ⚖️ Mergen Roleplay - Adalet, Güven Skoru ve Yaptırım Sistemi (V2.0)

## 📖 1. Sisteme Genel Bakış

Yeni sistem, oyuncuları sunucudan banlayıp uzaklaştırmak yerine **"Oyun İçi Süründürme" (Tematik Kısıtlamalar)** ve **"Güven Skoru" (Trust Factor)** üzerine kuruludur.

### 🛡️ Güven Skoru (Trust Factor - TF)
Her oyuncu sisteme **100 TF** ile başlar. İşlenen her ihlal, kuralın ağırlığına göre TF puanını düşürür.
* **90 - 100:** Temiz Sicil
* **70 - 89:** Gözetim Altında
* **50 - 69:** Sabıkalı
* **30 - 49:** Tehlikeli Madde
* **0 - 29:** Sürgün (Kritik Sicil)

*Sistem, Discord rollerini kişinin anlık TF puanına göre otomatik günceller.*

### 🔄 İyileşme Serisi (Clean Streak)
Cezasını çeken ve uslu duran oyuncular sonsuza kadar damgalı kalmaz. 
* Her gece `00:00`'da çalışan **Cron Job**, son cezasının üzerinden 24 saat geçmiş oyuncuların `clean_streak` (temiz kalma) serisini 1 gün artırır.
* Uslu durulan gün sayısına göre oyuncuya günlük **+1, +2 veya +3 TF** iade edilir.
* TF'si yükselen oyuncunun rolleri sistem tarafından otomatik geri verilir.

### 🚫 Tematik Yaptırımlar (Oyun İçi Kısıtlamalar)
Oyuncular TF puanı kaybederken aynı zamanda işledikleri suça özel kısıtlamalar (Saat/Miktar bazlı) alırlar:
* **COMMS:** OOC Sohbet ve Tweet yasağı (Discord Rolü ile entegre).
* **WEAPON (GunRP Yasağı):** Silah kullanım yasağı. Oyuncu birini vurursa sistem saniyesinde **PERMA BAN** atar.
* **DRIVE (Araç Yasağı):** Oyuncu araca binmeye çalıştığında sistem otomatik olarak araçtan atar, `5DK Jail` verir ve uyarır.
* **LABOR (Zorunlu Kamu Hizmeti):** Oyuncu madene gönderilir. Kalan cezası bitene kadar taş kırar ancak **asla XP, para veya eşya kazanamaz**. Sadece ceza sayacı düşer.

---

## 💻 2. Yeni Komutlar ve Özellikleri

* **`/uyari` (İhlal Kontrol Paneli):** Yetkilinin 58 farklı kural arasından seçim yaptığı, kanıt eklediği paneli açar. Sistem matematik hesaplarını (toplam düşecek TF, eklenecek saat) kendi yapar.
* **`/uyarılarım` (Sicil Dosyası):** Bir oyuncunun kendi güncel itibarını, kalan ceza sürelerini (dinamik geri sayım ile) ve geçmiş arşivini sayfa sayfa görebildiği komuttur.
* **`/ceza-sil` (İnsiyatif Paneli):** Yetkililerin aktif cezaları azaltmasını veya sıfırlamasını sağlar.
* **`;rp+` (Oyun İçi Ödül Sistemi):** Yüksek itibarlı oyuncuların (`TF >= 70`) çevrelerindeki (Max 30 Stud) en yakın 10 kişiye "Rol Pası" atarak onlara gizlice **+2 TF** kazandırdığı otonom in-game komutudur.

---

## 🧪 3. QA TEST DİREKTİFLERİ (EDGE CASES)

Test ekibinin sistemi kırmak için aşağıdaki ekstrem (Edge Case) senaryolarını harfiyen uygulaması gerekmektedir.

### 🛑 A. `/uyari` Komutu Testleri
1. **Zaman Aşımı Testi:** Paneli açın, hiçbir şeye basmadan 5 dakika (300 saniye) bekleyin. Panelin zaman aşımına uğrayıp kilitlendiğini doğrulayın.
2. **Kanıtsız İşlem Testi:** Menüden ihlal seçin ancak `Kanıt Ekle` butonunu kullanmadan `Onayla` butonuna basın. Botun hata vermesini bekleyin.
3. **Çoklu Menü Seçimi:** 1. Menüden 1 kural, 2. Menüden 1 kural ve 3. Menüden 1 kural seçip onaylayın. Sistem üçünü de birleştirip tek bir cezada toplamalıdır.
4. **Özel İnfaz Uyarıları:** `İzinsiz ERP` (WIPE gerektiren) veya `İzinsiz Res` (CLEAR gerektiren) kurallarını seçin. Sonuç ekranında **"🚨 SİSTEM UYARISI: ENVANTER SİLİNMESİ GEREKİYOR"** bildiriminin çıktığını teyit edin.

### 🛡️ B. `/ceza-sil` Komutu Testleri
1. **Olmayan Cezayı Silme:** Hiç cezası olmayan veya süresi geçmiş cezası olan birine komutu atın. Botun işlemi reddettiğini doğrulayın.
2. **Değer Yükseltme Girişimi (Kritik):** Bir oyuncunun "Maden Cezası: 20" ise, ceza sil menüsüne girip yeni değer olarak `50` yazın. Sistemin **bunu kesinlikle reddetmesi** ve "Cezayı uzatamazsınız" hatası vermesi gerekir.
3. **Negatif/Geçersiz Veri:** Yeni değere `-5` veya `abc` yazmayı deneyin. Sistem engellemelidir.
4. **Tamamen Silme (Discord Senkronizasyonu):** İletişim (COMMS) yasağı olan birinin yasağını `0` olarak güncelleyin. Discord sunucusundaki Mute/Comms yasaklı rolünün **anında** o kişiden alındığını gözlemleyin.

### ⛏️ C. Oyun İçi (In-Game) Yaptırım Testleri
1. **Sıfır Tolerans (GunRP):** Oyuncuya `/uyari` üzerinden "Revenge RP (GunRP Yasağı)" atın. Oyuncu oyuna girip birini vursun (Kill Log'a düşsün). Sistem oyuncuyu saniyesinde **PERMA BAN** atmalı ve Discord Ban Log'a `🚨 SİFIR TOLERANS İHLALİ` mesajı düşmelidir.
2. **Araç Binme Bloku:** "Problock (Garaj Engeli)" cezası almış bir oyuncuyu Exotic veya normal bir araca bindirin. Sistemin kişiyi hemen arabadan atıp `5 Dakika Jail` ve PM uyarısı verdiğini görün.
3. **Maden (Kamu Cezası) Testi:** Maden cezası olan biriyle gidip taş kırın.
   - XP gelmemeli.
   - Envantere taş/maden gelmemeli.
   - Kazma ASLA kırılmamalı (Şans faktörü kapalı olmalı).
   - "Kalan Ceza: X Adet" yazmalı. Sayı `0` olduğunda **"Tebrikler, cezan bitti, TF iade edildi"** yazısını görün.

### 🌟 D. `;rp+` (Oyun İçi Ödül) Komutu Testleri
1. **Spam ve Limit Testi:**
   - **TF'si 95** olan bir yetkili/oyuncu hesabı ile komutu 4 kez arka arkaya yazın. 4. kullanım çalışmamalıdır.
   - **TF'si 80** olan bir hesapla 2. kez yazın. Çalışmamalıdır.
   - **TF'si 50** olan bir hesapla yazın. Hiçbir tepki vermemelidir (Yetkisiz).
2. **Öz-Puan Kontrolü (Exploit Testi):** Tek başınıza ıssız bir yerde komutu kullanın. Size asla puan vermemelidir.



---
**Geliştirici Notu:** Loglardaki `Event Loop Blocking` sorununu aşmak için veritabanı başlangıç RAM yüklemesi son 1.5 gün ile sınırlandırılmıştır. Sistemi yormamak adına aylık periyotlarla `node db_temizlik.js` komutunu kullanarak eski logları SQLite üzerinden silebilirsiniz.

*Başarılar ve Kolay Gelsin,*
**Mergen Roleplay Sistem Yönetimi**
