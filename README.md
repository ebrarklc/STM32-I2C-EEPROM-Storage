# 💾 STM32 I2C EEPROM Kalıcı Veri Depolama & 7-Segment HMI

Bu proje, gömülü sistemlerde enerji kesintilerine karşı verilerin korunmasını sağlayan **Kalıcı Bellek (EEPROM)** kullanımını ve **I2C (Inter-Integrated Circuit)** haberleşme protokolünün donanım seviyesinde nasıl entegre edileceğini göstermektedir. 

Sistem, kullanıcının fiziksel buton ile girdiği komutları işler, sayacı 7-Segment ekranda gösterir, LED parlaklığını PWM ile değiştirir ve en önemlisi; güncel durumu EEPROM'a kaydederek güç kesintisi (Power-Loss) sonrasında sistemin kaldığı yerden devam etmesini sağlar.

*(Geliştirme ve test süreçleri, STM32F407 işlemcili ArmApp-18 eğitim seti üzerinde gerçekleştirilmiştir.)*

## 🚀 Öne Çıkan Özellikler (Highlights)

* **I2C Tabanlı EEPROM Entegrasyonu:** İşlemci ile harici EEPROM çipi arasında I2C donanım modülü üzerinden çift yönlü haberleşme kurularak sayaç verisi kalıcı belleğe yazılıp okunmuştur.
* **Kalıcı Durum Belleği (Non-Volatile State):** Sistem her açılışta önce EEPROM'u okur. Eğer daha önceden kaydedilmiş bir veri varsa (örneğin 7 rakamı), açılışta sıfırdan başlamak yerine hafızadaki bu değerle sistemi başlatır.
* **PWM ve Dijital Çıkış Kombinasyonu:** Butona basıldığında sadece sayaç artmaz; aynı zamanda TIM4 birimi üzerinden üretilen PWM sinyali ile Power LED'in parlaklığı da senkronize olarak değiştirilir.
* **7-Segment Display Sürüşü:** 4 haneli 7-Segment ekranı sürmek için gerekli pin konfigürasyonları ve multiplexing (çoğullama) altyapısı kurulmuştur.

## 🛠️ Donanım ve Pin Yapılandırması

Sistemdeki dış birimlerin STM32F407 üzerindeki pin atamaları aşağıdaki gibidir:

| Bileşen | Pin Kodu | Kullanım Modu | Açıklama |
| :--- | :--- | :--- | :--- |
| **I2C SCL** | `PA8` | I2C3_SCL | EEPROM saat (Clock) senkronizasyon hattı. |
| **I2C SDA** | `PC9` | I2C3_SDA | EEPROM çift yönlü veri (Data) hattı. |
| **Yazma Butonu** | `PB8` | GPIO_Input | Sayacı artırma ve EEPROM'a yazma tetikleyicisi (Pull-down). |
| **Power LED (PWM)** | `PB6` | TIM4_CH1 | Parlaklık kontrolü için donanımsal PWM çıkışı. |
| **7-Segment Veri** | `PC0-PC6` | GPIO_Output | A, B, C, D, E, F, G segment pinleri. |
| **7-Segment Seçim**| `PB0-PB3` | GPIO_Output | Ortak katot/anot dijit seçim (Multiplexing) pinleri. |

## 📂 Yazılım Mimarisi (Algoritma)

1.  **Açılış (Boot) Dizisi:** Mikrodenetleyici başlatıldığında, I2C hattı üzerinden EEPROM'un ilgili bellek adresine okuma isteği gönderilir. Okunan veri `0-9` aralığındaysa sistem bu değerle, değilse `0` ile başlatılır.
2.  **Kullanıcı Girdisi ve Debounce:** `PB8` pinine bağlı buton (Pull-down konfigürasyonlu) okunduğunda, yazılımsal debounce (ark önleme) uygulanarak sayacın stabil şekilde 1 artırılması sağlanır. Sayaç 9'u geçtiğinde tekrar 0'a döner.
3.  **Aksiyon ve Kayıt:** Sayaç güncellendiği anda yeni değer hem I2C üzerinden EEPROM'a yazılır, hem 7-Segment ekranda gösterilir, hem de PWM 'Duty Cycle' değeri artırılarak LED parlaklığı ayarlanır.

## 💻 Nasıl Çalıştırılır?

1.  Projeyi `STM32CubeIDE` ile açın, derleyin (`Build`) ve karta yükleyin.
2.  Sistem ilk açıldığında 7-Segment ekranda `0` (veya EEPROM'da kalan son rakam) görünecektir.
3.  Butona (`PB8`) her bastığınızda sayacın arttığını ve LED parlaklığının değiştiğini gözlemleyin.
4.  **Kalıcılık Testi:** Sayacı belirli bir değere (örn: 5) getirin. Kartın gücünü (USB kablosunu) tamamen kesin. Yaklaşık 10 saniye bekledikten sonra gücü tekrar verin. Sistem 0'dan değil, EEPROM'dan okuduğu **5** rakamıyla doğrudan açılacaktır!
