# SON IŞIK V14 · İZ SÜR

**Ürün ve teknik yeniden tasarım belgesi**  
**Tarih:** 23 Kasım 2026  
**Durum:** İlk oynanabilir 3B vertical slice / beta. Gerçek iPhone Safari ve fiziksel GPU kabul testleri henüz yapılmadı.  
**Canlı dosya:** `son-isik/v14.html`  
**Korunan önceki sürüm:** `son-isik/v13.html`

## 1. Tasarım kararı: Oyuncu neden oynasın?

Eski oyunda “sekiz oyuncak bul” görevi kayıp kız hikâyesine yeterli neden-sonuç bağı kurmuyordu. 320×320 birimlik orman, çok sayıda rastgele korku olayı ve hud sayacı oyuncuya nereye ve neden gittiğini net göstermiyordu. **Bir korku mekaniği eklemek, oyun döngüsü tasarlamak değildir.**

V14'ün merkezi sözü: **“Karanlıkta hiçbir şeye güvenemezsin. Ama kızını bulmak için ışığını yakmalısın.”** Oyuncu babadır; kızı Elif ormanda kaybolmuştur. Her buluntu bir bilgi sağlar, bir sonraki hedefi açar ve final kararını mantıksal olarak etkiler. Oyunun gerçek ödülü bir sayaç değil, **Elif'in sesini taklit eden varlığı kanıtlara dayanarak ayırt edip ona ulaşmaktır.**

Yeni ana döngü: **karanlığı tarama → fiziksel iz bulma → ipucu çözme → rotaya karar verme → ışık/ses/gizlenme riski → bir sonraki kanıt → finalde çıkarım**. Fener aynı anda bilgi ve tehlike aracıdır. Kapatınca görünür dünya sıfırlanır, ama avcının görsel takibi zorlaşır. Feneri elle şarj etmek ses çıkarır. Gerilim, oyuncunun kendi kararından doğar.

## 2. Üç konsept ve seçim

| Konsept | Duygusal vaat | Temel risk | Karar |
|---|---|---|---|
| **A. İz Sür (Seçilen)** | Kızını bulmak için izleri çöz ve gerçek sesi ayırt et | Işık seni ele veriyor | **V14'te uygulandı** |
| B. Bozuk Hafıza | Orman babanın hafızasını değiştiriyor | Kanıtlar güvenilmez olabilir | Daha sonraki bölüm için, ilk oyunda fazla soyut |
| C. Gece Devriyesi | Tek başına geceyi sağ çıkarmak | Kaynak ve pil kıtlığı | Mevcut hikâyeyi arka plana itiyor |

A konsepti, hikâye ile oynanışı doğrudan bağlar. Diğerleri korku öğesi olarak kullanılabilir ama ilk bölümün ana motivasyonu olmamalı.

## 3. V14 hikâyesi ve dört aşama

1. **Oyun parkı, 0/3 kanıt.** Oyuncu eski salıncağın yanında kırmızı ayakkabı bulur. Kağıt: **“İki kere. Bekle. Bir kere.”** Elif’in sahici yardım ritmi öğrenilir. Artık oyuncunun aradığı bilgi vardır.
2. **Çatallanan yol, 1/3.** Soldaki paslı çan veya sağdaki köprü incelenir. Çan yolu daha erken tehdidi tetikler; köprü daha sonra tetikler. Her rota ikinci bir delil sunar. Oyuncu kendi yönünü seçer ve geri bildirim alır.
3. **Rüzgâr çanları, 2/3.** Elif’in bilekliği ve aynı iki-bir ritmini gösteren düğümler bulunur. İki bağımsız ipucu aynı sonucu destekler. Oyuncu neyin gerçek olduğunu çözer.
4. **Kulübe, 3/3.** İki farklı kapı sesi gelir. Sol tarafta **iki, bekle, bir**, diğerinde **üç** vuruş vardır. Oyuncu kanıtlara güvenirse Elif’i bulur; yanlış sesi seçerse kandırılır. Aynı kapı iki farklı sonuç vermez; görünür kararın gerçek bir sonucu vardır.

**Bağlantılı sonuç:** Avcıya yakalanmak kanıtları silmez. Son kanıt noktasına döner, biraz fener enerjisi kaybedersin. Oyuna devam etmek anlamlı ve mümkün kalır. Bu, rastgele anında ölme ve her şeyi yeniden toplama döngüsünü önler.

## 4. Yalnızca fenerde görünürlük: uygulama

### Seçilen yöntem: tek SpotLight + malzemelerin tümüne ortak dünyasal ışın maskesi

V14, **ambiyans ışığını, ay ışığını, diğer point light'ları ve emissive dünya nesnelerini kaldırır.** Tek bir Three.js `SpotLight`, oyuncunun feneri olarak işlev görür. `MeshStandardMaterial.onBeforeCompile` ile her dünya materyalinin vertex shader'ına **world-space fragment pozisyonu**, fragment shader'ına ışın doğrultusu, iç/dış açı, mesafe ve güç uniformları eklenir. Spot ışık ve dünya maskesinin **kesişimi** görülür; başka hiçbir world pixel'i açık kalmamalıdır. Işık dışındakiler siyaha çarpılır. Fener kapalıyken maske gücü 0’dır. Dünya fonu siyah, sis siyahtır. UI kamera sahnesinin dışında HTML katmanıdır ve okunur kalır.

Üstten bakan izometrik kamerada shader kullanmak önemlidir: sıradan ekrana sabit siyah bir elips çizmek, ağaçların arkasını ve ekran hareketini doğru yönetemez. V14 maskesi ağaçların gerçek dünya koordinatlarını ve `InstancedMesh.instanceMatrix` dönüşümlerini dikkate alır. Spotlight gölge haritası gövde/kanopi engellerinin önündeki ışığı kesmek için kullanılır. Hafif yakın alan aydınlığı fenerin çevresindeki yaklaşık 1,5 birimlik alanda korunur. UI bu maskeden bağımsızdır.

**Önemli teknik düzeltme:** `MeshBasicMaterial` normal spot ışıktan etkilenmez; onu gerçek tek-fener ışıklandırması için kullanmak doğru çözüm değildir. `if (dot(light, normal) < ...) discard` kontrolü de **ışın konisini** hesaplamaz; koniyi belirlemek için oyuncudan fragmana vektör ile fenerin ileri yönü arasındaki açı kullanılır. “Stencil/MaskPass dış bölgeyi çizmez” demek de teknik olarak doğru değildir; post-process çoğunlukla önce sahneyi çizer, sonra görüntüyü maskeler. V14 ekstra tam ekran render pass yerine tek pass materyal maskesi seçti.

**Mobil GPU sınırları:** Gövde/kanopi üç toplu `InstancedMesh` ile çizilir. Gölge haritası mobilde 512×512'dir. Piksel oranı sınırlandırılır; yavaş karelerden sonra azaltılır. 192 ağaç mobilde, 240 masaüstünde oluşturulur. Bu değerler birer başlangıç tercihi, ölçülmüş nihai performans değildir.

## 5. Kontroller ve onboarding

- **iPhone portrait / landscape:** Sol joystick hareket. Sağdaki boş ekran alanını sürüklemek fener yönü. **İNCELE**, **FENER**, basılı tutulan **ŞARJ** ve **DİNLE** düğmeleri. Sesi kapatma ve duraklatma ayrı.
- **Klavye/fare:** WASD veya yön tuşları, fareyle nişan, `E` incele, `F` ışık, basılı `C` şarj, `Q` dinle, `Esc` duraklat.
- **İlk eğitim:** Açılış ekranı arama hedefini ve fener riskini anlatır. İlk ipucunda “2-1” ritmi gösterilir. Kavşakta bir kerelik yön metni çıkar. Ekran daha sonra kalıcı ok ve karmaşık minimap ile doldurulmaz.
- **DİNLE:** Bir sonraki gerçek görev noktasının yönünü konumsal sesle verir; altyazı destekliyse ses yönünü ve yaklaşık adım mesafesini belirtir. Bir soğuma süresi bulunur.
- **Erişilebilirlik:** Patlayan flaş yok; azaltılmış hareket tercihi izlenir. Sesli ipucunun altyazısı duraklatma menüsünden kontrol edilebilir. Kulaklık tavsiye edilir fakat zorunlu değildir. Karanlıktaki dünyanın görünürlüğünü değiştiren parlaklık hilesi bulunmaz.

## 6. İlk deneyim akışı

| Yaklaşık an | Oyuncu eylemi | Bilgi / geri bildirim | Gerilim |
|---|---|---|---|
| Açılış | Aramaya başla | “Elif kayıp; kanıt bul, sesini ayırt et.” | Düşük |
| İlk patika | Sadece fenerle yürüyüp çevreyi tara | Karanlık dünya, ışık konisi ve ilk hedef | Yükselir |
| Oyun parkı | Kırmızı ayakkabıyı incele | İlk **iki-bir** ritmi | Merak |
| Oyun parkı sonrası | Avcı sesi duy, ışığı kapatıp uzaklaş veya hızlan | Işık tehlike çekiyor, kapanınca görünürlük kayboluyor | Zirve |
| Kavşak | Çan veya köprü rotasını seç | Seçimin gerçek tehlike zamanını değiştirir | Orta |
| Rüzgâr çanları | Bilekliği bul | Ritim doğrulanır | Gerilim / çözüm |
| Kulübe | İki kapı sesinden birini seç | Elif’i bulma veya taklitçinin tuzağı | Final |

Bu vertical slice’ın kesin süresi henüz ölçülmedi. Önceki araştırmadaki 15–25 dakika hedefi **bu sürümde garanti edilmiş değildir**. Oynanış testine göre yolların, tehdit ve kanıt anlarının süresi yeniden kalibre edilmelidir.

## 7. Kabul kriterleri / test planı

### Fonksiyonel

- [x] V13 dosyası değiştirilmeden V14 ayrı HTML dosyasında oluşturuldu.
- [x] Genel, directional, hemisphere ışıkların sıfır olması kodda kontrol edildi.
- [x] Yalnızca fener maskesine bağlanmış materyal üretimi ve üç `InstancedMesh` testi yapıldı.
- [x] İlk delil, her iki kavşak rotası, üçüncü delil ve doğru son mantık testi geçti.
- [x] V14 ilerlemesi V13 kayıt anahtarına dokunmaz; yeni ilerleme ayrı saklanır.
- [ ] Gerçek Three.js/WebGL ile 0 güçte tüm dünya piksellerinin RGB=0 olduğunun screenshot/pixel testi.
- [ ] Safari iPhone 320–430 px ve landscape'te gerçek joystick, aim ve UI QA.
- [ ] Gerçek GPU'da hedef 30+ FPS ve yüksek yüzdelik frame-time ölçümleri.
- [ ] Kullanıcı testi: İlk 90 saniyede “niçin oynuyorum?” sorusunun yanıtı.
- [ ] En az beş yeni oyuncuyla görev tamamlama, kaybolma ve gerilim eğrisi değerlendirmesi.
- [ ] Ses kulaklık/mono/mute modunda ipuçlarının ulaşılabilirliğinin ölçümü.

### Başarı ölçümleri (hedef, henüz gerçekleşmiş sonuç değil)

**İlk buluntuya kadar geçen süre**, 2 dakikanın altında hedef. **Görev anlayışı**, ilk iki dakikada test edilen oyuncuların en az %80'i; **kanıt-finale bağlantısı**, test edilen oyuncuların en az %75'i. **Kavşak kararının nedeni**, en az %70 açıklayabilmeli. **Tutarlı mobil kare hızı**, gerçek iPhone'da medyan 30+ FPS. **Korku hissi**, röportaj ve sesli oynanış gözlemiyle değerlendirilecek; tek başına telemetri yüzdesiyle kanıtlanamaz.

## 8. Teknik risk ve sonraki mühendislik işleri

**P0:** Gerçek Safari/Three.js GPU testini geçir. Shader `onBeforeCompile` chunk uyumluluğunu GPU üzerinde doğrula; özellikle `USE_INSTANCING`, `#include <project_vertex>` ve `#include <dithering_fragment>` noktaları. Shader uyuşmazlığında oyunun açılamaması kabul edilemez: hata ve uyumlu düşük kalite fallback gerekir.

**P1:** Dinamik ışık gölgelerinin gövdeleri ve ağaçları gerçekte örtüp örtmediğini görsel olarak kontrol et. Siluet arkada ışık olmadan görünüyorsa shader veya özel materyal kaçağı vardır. Harita/çevre sesleriyle yön bulma kör noktaları giderilmeli.

**P2:** Özellikle mobilde kameranın parmak etkileşiminde nişanın yönüne göre hareket kontrolü test edilmeli. Düşman ağaca takıldığında görsel olarak duvar içinden geçmemeli; mevcut basit çarpışma/yan adım mantığı ölçülmeli.

**P3:** Yaratık animasyonlarını hand-painted varlıklara yükselt, fakat **sadece** yukarıdaki sistem çalıştıktan sonra. Güzel gözüken ama amacını anlatamayan bir korku ormanı üretim hedefi olmamalı.

### Sürüm / veri politikası

V14, V13'ün veri formatını **bilinçli olarak taşımıyor**, çünkü sekiz oyuncaklı koleksiyon ilerlemesi ile üç kanıtlı yeni anlatı birbirine eşdeğer değil. Kullanıcının eski V13 ilerlemesi silinmez. V13'e dönüş menüde korunur. V14 kendi `sonIsikV14Chapter01` anahtarında saklanır; yeni oyun bu anahtarı temizler. Geri dönüş planı `v13.html` bağlantısıyla ayrı çalışan dosyayı açmaktır.

## 9. Referanslar

- [Darkwood görünürlük mekaniği, Game Developer](https://www.gamedeveloper.com/design/how-i-darkwood-i-s-visibility-mechanics-create-a-new-kind-of-horror) – dar görüş ve bilgi maliyeti.
- [Amnesia: The Bunker'ın yaratığı, AI and Games](https://www.aiandgames.com/p/how-the-beast-works-in-amnesia-the) – düşman, ses ve oyuncu ajansı ilişkisi.
- [Three.js Material.onBeforeCompile resmi dokümanı](https://threejs.org/docs/pages/Material.html) – uniform ve shader enjeksiyonu.
- [Xbox Accessibility Guideline 118](https://docs.xbox.com/build/game-principles/accessibility/xag-deep-dives/xag-118-photosensitivity) – yoğun flaş ve fotosensitivite.
- [Darkwood, PlayStation Blog](https://blog.playstation.com/archive/2019/05/01/darkwood-is-a-ps4-survival-horror-that-favours-creepy-atmosphere-over-cheap-jump-scares/feed/) – görüş ve sesin korkuya etkisi.

---

**Ürün değerlendirmesi:** V14, “bir sürü yaratık” yerine küçük bir **amaç-bilgi-karar-sonuç** deneyimine dönüşmüştür. Bu hâlâ **beta**: gerçek cihaz testi ve kullanıcı gerilimi ölçümü yapılmadan “korkutucu/çalışmaya hazır” olduğu ileri sürülmemelidir.