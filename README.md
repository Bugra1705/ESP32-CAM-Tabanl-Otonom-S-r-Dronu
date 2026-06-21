# ESP32-CAM Tabanlı Otonom Sürü Dronu — Uçuş Kontrol Kartı (Flight Controller)

> Sürü (swarm) algoritmalarıyla entegre çalışabilecek, görüntü işleme yetenekli, ultra hafif bir uçuş kontrol kartının **uçtan uca şematik ve çift katmanlı PCB tasarımı**.
> Altium Designer ile donanım tasarımı, ESP32 için Gömülü C ile yazılım geliştirme.

**Durum:** 🟡 Devam ediyor — PCB üretimden teslim bekleniyor, eş zamanlı olarak yazılım geliştirme ve bileşen tedarik süreçleri sürüyor.

---

## 🛠️ Kullanılan Araçlar
`Altium Designer` · `Gömülü C` · `VS Code + PlatformIO` · `ESP32-CAM`

---

## 📸 Tasarım Görselleri

> Not: Aşağıdaki dosyaları `images/` klasörüne ekledikten sonra bu görseller otomatik görünür.

**3B Render (Üstten)**
![3B Render Üstten](images/3d_render_top.jpeg)

**3B Render (İzometrik)**
![3B Render İzometrik](images/3d_render_iso.jpeg)

**Şematik**
![Şematik](images/schematic.jpeg)

**PCB — Üst Katman / Alt Katman**
![PCB Üst Katman](images/pcb_top.jpeg)
![PCB Alt Katman](images/pcb_bottom.jpeg)

---

## 🎯 Öne Çıkan Mühendislik Kararları

Bu projede sadece bağlantı çizmek yerine, her bileşeni belirli bir mühendislik gerekçesiyle seçtim:

- **ADC2 / WiFi çakışmasının çözümü:** ESP32 mimarisinde, WiFi aktifken dahili ADC2 birimi devre dışı kalıyor. Sürü algoritmaları için kritik olan haberleşmeyi aksatmamak adına, batarya izlemeyi **16-bit harici ADS1115 ADC** ile I2C hattı üzerinden gerçekleştirdim. Böylece WiFi kesintisiz çalışırken pil gerilimini yüksek hassasiyetle dijitalleştirdim.

- **Güç yönetimi ve brown-out koruması:** 1S LiPo gerilimi (3.7V–4.2V) kritik seviyelere düştüğünde dahi sistemin kilitlenmemesi için, **360 mV gibi çok düşük dropout** değerine sahip ve 1A kapasiteli **AP7361C LDO** regülatörünü tercih ettim.

- **Motor sürücü katı ve ters-EMF koruması:** 8520 coreless motorların kontrolü için **AO3400A lojik-seviye N-Kanal MOSFET'lerle low-side anahtarlama** kurdum. MOSFET'in düşük eşik gerilimi (Vgs ≈ 0.7–1.45V) sayesinde ESP32'nin 3.3V lojik sinyalleriyle motorlar tam verimle sürülüyor. Motorların duruş anındaki **ters-EMF (Back-EMF) sıçramalarına karşı SS14 Schottky diyotlar** ve kararlı başlatma için **gate pull-down dirençleri** ekledim.

- **Ağırlık optimizasyonu:** Dronun toplam ağırlığını düşürmek için standart pin header yerine, programlama arayüzünde **Pogo Pin (yaylı mandal) test pad'leri** (TP1–TP4) kullandım. Programlama, USB-TTL dönüştürücü + Pogo Pin aparatıyla ek ağırlık olmadan yapılabiliyor.

- **I2C adres çakışmalarının yönetimi:** Aynı I2C hattındaki cihazları çakışmadan çalıştırmak için IMU adresini `0x68`, ADS1115 adresini `0x48` olarak konfigüre ettim; hatta 4.7K pull-up dirençleri ekledim.

---

## 📐 Teknik Özellikler

| Özellik | Detay |
|---|---|
| Ana işlemci | ESP32-CAM (çift çekirdek) + OV2640 kamera |
| IMU sensörü | MPU6050 (GY-521) |
| Güç kaynağı | 1S 650mAh LiPo (3.7V–4.2V) |
| Regülasyon | AP7361C LDO → 3.3V (360mV dropout, 1A) |
| Batarya izleme | ADS1115 16-bit ADC (I2C), gerilim bölücü 4.2V → 2.1V |
| Motor sürücü | 4 × AO3400A N-Kanal MOSFET (low-side), 4 × SS14 Schottky |
| Motorlar | 4 × 8520 coreless |
| Motor PWM | GPIO 14, 15, 13, 12 (M1–M4) |
| Haberleşme | I2C (IMU + ADC), UART/TTL (programlama) |
| Gövde | X-tipi, ~100 mm çapraz mesafe, PCB ile bütünleşik |
| Programlama | Pogo Pin test pad'leri (TP1–TP4) + USB-TTL |

---

## 🔋 Güç & Koruma Mimarisi Özeti

- 1S LiPo girişi → AP7361C LDO ile 3.3V sistem rayı
- Düşük dropout sayesinde pil zayıfladığında bile kararlı çalışma
- Her motor hattında: AO3400A MOSFET + SS14 Schottky + 100K gate pull-down + 220Ω gate direnci
- Batarya gerilimi gerilim bölücüyle güvenli seviyeye çekilip ADS1115 üzerinden okunuyor

---

## 💻 Yazılım & Geliştirme Ortamı

- **Ortam:** Visual Studio Code + PlatformIO
- ESP32 için uçuş ve haberleşme kodları modüler yapıda geliştiriliyor

---

## 🗺️ Yol Haritası

- [x] Şematik tasarımı (Altium Designer)
- [x] Çift katmanlı PCB tasarımı ve üretime gönderim
- [x] Üretimden gelen kartların bileşen lehimleme/montajı
- [x] Test kodlarıyla ilk uçuş ve temel stabilite denemeleri
- [ ] Dronun dinamik davranışının modellenmesi (transfer fonksiyonunun çıkarılması)

---

## 📂 Bileşen Listesi (BOM — Özet)

| Bileşen | Adet | Açıklama |
|---|---|---|
| ESP32-CAM | 1 | Ana kontrolcü + kamera |
| MPU6050 (GY-521) | 1 | IMU sensörü |
| ADS1115 | 1 | 16-bit harici ADC |
| AP7361C | 1 | 3.3V LDO regülatör |
| AO3400A | 4 | N-Kanal MOSFET (motor sürücü) |
| SS14 | 4 | Schottky diyot (ters-EMF koruma) |
| 8520 Motor | 4 | Coreless motor |
| Pasif bileşenler | — | Kapasitör, direnç, JST konnektör, test pad'leri |

*Tam BOM repoda `docs/` klasöründe yer almaktadır.*

---

*Bu proje, Marmara Üniversitesi Elektrik-Elektronik Mühendisliği bitirme projesi kapsamında geliştirilmektedir.*
