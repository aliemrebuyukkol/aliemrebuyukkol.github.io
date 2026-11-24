# SON IŞIK V23 | BİRİNCİ ŞAHIS YENİDEN TASARIM

## Ürün kararı

V1–V22'nin izometrik yapısı yerine doğrudan **oyuncu göz hizasındaki kamera** kullanılır. Kameranın yüksekliği ~1,72 m, mobil dikey görüş açısı 78°, yatay görüş açısı 73°'dir. Fenerin yönü kameranın yönüyle aynıdır. Artık feneri ayrı bir analog ile yönetmek gerekmez.

**Temel vaat:** Elif'i bulmak isteyen oyuncu, karanlığı aydınlattığında kendini de ele verir. Feneri kapattığında yönünü ancak seslerle anlayabilir. Pil tamamen biterse oyun kaybedilir.

## Oyun döngüsü

1. Dur, karanlığı dinle, Elif'in kanıtının yönünü öğren.
2. Feneri kısa süre açarak yürünebilir patikayı ve nesneleri incele.
3. Yeni kanıta bakıp `İNCELE` ile ilerle.
4. Yedek pil bul, yanlış yöne girmemek için enerji yönet.
5. Avcıdan ışıkla uzaklaşmasını sağlayarak veya feneri kapatıp saklanarak kurtul.
6. Son bölümde gerçek ve taklit vuruşların farkından doğru kapıyı seç.

## Bölümler

| Bölüm | Hedef | Çevre ve tehlike | Kontrol noktası |
|---|---|---|---|
| I. Kayıp Park | Ayakkabı, ardından kavşak | Salıncak, kırık çit, ilk vuruş ritmi | Orman girişi |
| II. Boğulmuş Koruluk | Çan veya dere rotası; bileklik | Boş kıyafetler, su, taklit ritim, avcı | Seçilen rotanın girişi |
| III. Son Geçit | Kulübeye ulaş, iki kapıyı dinle | İnsan biçimli nöbetçiler, fener arızası, karar | Son geçit girişi |

V23, **tamamen farklı haritalar** değil, ortak ormanın görsel olarak farklılaşan bölgeleridir. Bu bir kapsam kararıdır; üç ayrı harita yapıldığı iddia edilmemelidir.

## Kamera ve görsel sistem

- `Three.js r160` mevcut `son-isik/vendor/three.module.js` dosyasından yüklenir.
- `PerspectiveCamera`; `OrthographicCamera` veya oyuncuyu dışarıdan gösteren kamera kullanılmaz.
- Dünya aydınlatması: sıfır ortam ışığı, siyah sahne/fog, yalnızca kamera ile yönlenen bir `SpotLight`.
- Mobilde hedef piksel oranı en fazla 1.35, masaüstünde en fazla 1.75; mobil shadow map 512, masaüstü 1024.
- Ağaçlar `InstancedMesh` ile çizilir; önemli objeler normal geometrilerdir.
- Fener arızası 1–1.25 saniyelik kararmadır; yoğun flaş patlaması yoktur.
- Spot ışık dışındaki dünya nesneleri aydınlatılmaz. **Bu kuralın gerçek iPhone Safari/WebGL çıktısı henüz fiziksel cihazda doğrulanmamıştır.**

## Kontroller

| Girdi | Mobil | Masaüstü |
|---|---|---|
| Hareket | Sol başparmak analogu | WASD / yön tuşları |
| Bakış / fener | Sağ ekranda sürükle | Fare hareketi, pointer lock |
| İncele | Bağlamsal `İNCELE` | E veya Space |
| Fener | FENER | F |
| Yedek pil | PİL TAK | R |
| Dinle | DİNLE (feneri kapatır) | Q |
| Duraklat | Ⅱ | Esc / P |

Oyuncu yürürken nişan alabilir. Fener yönü otomatik olarak insanın baktığı yöndür. Mobil kontrolün varsayılan davranışı nişan almak için özel düğme gerektirmez.

## Kayıt ve geçiş

- V23 kendi `localStorage` anahtarını kullanır: `sonIsikV23FirstPerson`.
- V22'nin `sonIsikV22ThreeZones` anahtarı korunur. Açılış ekranında kullanıcı isterse V22 ilerlemesini içe alır.
- Eski izometrik koordinat güvenli oynanabilir koridorun dışındaysa bölüm girişine taşınır.
- Oyun ölümünden sonra devam edilirse bölüm başlangıcından güvenli şekilde devam edilir.

## Test kanıtları ve sınırlar

- Node sözdizimi doğrulaması geçti.
- 16 bağımsız mantık testi: iki rota, erişilebilirlik, çarpışma sınırları, bakış, kayıt, pil ve karanlıkta nesne incelememe.
- Chromium + **3B motor taklidi** ile 320, 390 ve 844 px arayüz / giriş / bölümler / ölüm akışı geçti.
- Hem çan hem dere rotasında ayakkabı → kavşak → bileklik → doğru kapı → final uçtan uca test edildi.
- Mobil analog, sürükleme ile bakış, fener ve tek dokunuşla dinleme girdileri test edildi.
- Bu testler **gerçek Three.js WebGL görüntüsünü, fiziksel iPhone FPS değerlerini veya oyunun korkutuculuğunu doğrulamaz.**
- Canavar geometrisi ve sesler hâlâ prosedürel. Profesyonel animasyon ve kayıtlı korku sesleri yoktur.

## Öncelikli sonraki işler

1. Gerçek iPhone Safari üzerinde oyunun açılışını, görüş maskesini, ısıyı ve FPS'yi test et.
2. Sağ başparmak nişan hassasiyeti ile oyuncu geri bildirimini ölç, ayarlanabilir hassasiyet ekle.
3. Yaratığı özgün rig/animasyon ve ses prodüksiyonu ile değiştir.
4. Sessizlik ve takip arasında 15–20 dakikalık dramatik tempo dengesini oynanış testleriyle doğrula.
5. WCAG dokunma alanları, hareket azaltma ve ses olmadan oynanabilirlik için gerçek cihaz erişilebilirlik testi yap.