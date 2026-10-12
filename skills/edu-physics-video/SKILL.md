---
name: edu-physics-video
description: >-
  Bir fizik problemi için açıklayıcı / çözüm videosu istendiğinde kullanın (mekanik: kuvvet analizi, Newton yasaları, eğik düzlem, konveyör, yatay atış, dairesel hareket, enerji, momentum; optik: kırılma; elektromanyetizma: yüklü parçacıklar), problem ekran görüntüsü veya metinden, Türkçe (veya İngilizce) seslendirme (edge-tts veya GLM-TTS), çift dilli tr+en altyazılar ve hareketin gerçek analitik çözümü takip ettiği blueprint tarzı canvas animasyonu ile MP4'e render edilmiş.
  Tetikleyici kelimeler: fizik açıklayıcı video, kuvvet diyagramı, Newton yasaları, eğik düzlem, yatay atış, dairesel hareket, enerji, momentum, kırılma; physics explainer video.
---

# Fizik Problemi Açıklayıcı Videosu (edu-physics-video)

Bir fizik problemini 16:9, 1920×1080 MP4'e dönüştürür: Türkçe/İngilizce seslendirme + çift dilli altyazılar + blueprint tarzı animasyon. Boru hattı edu-math-video ile aynıdır; değişen şey çözüm yöntemi ve resimdir: önce fizik iş akışını takip ederek doğru çözün (modelle → bölümlere ayır → kuvvetler → yasa seç → çöz → doğrula), ardından resmin fiziksel süreci gerçekten canlandırmasını sağlayın — kuvvet okları uygulama noktasından tutarlı ölçekte büyür, nesne analitik çözüm `x(t)`'ye göre `simT()` doğrusal eşlemesiyle hareket eder, v–t grafiği senkron büyür, sürtünme göreceli hareket tersine döndüğünde tersine döner.

**Temel ilke: resim anlatımı takip eder ve fiziksel süreç şekilde gerçekten gerçekleşir.** Her anlatım satırının o fiziği canlandıran bir eylemi vardır; her animasyon `S.at(k, f)` ile zamanlanır; nesne konumu yalnızca analitik çözümden gelir.

Yetenek tüm kodu içerir (eğik düzlem örneğiyle şablon, phys.js prop kütüphanesi, optik için examples/laser-curvature, paylaşılan telaffuz, betikler). Diğer projelerden kopyalamayın. WS = kullanıcının geçerli dizini.

## Uyulması gereken kurallar
Math-video ile aynı 11 adımlık disiplin, artı:
- Önce fizik yöntemiyle çözün ve çözüm kartı yazın (çalışma nesnesi, süreç bölümleri, her bölüm için kuvvetler, kullanılan yasalar, cevap, doğrulama).
- `tts` konuşulabilir olmalıdır (birimler yüksek sesle okunur: "saniyede on metre", Yunan harfleri "teta" olarak vb.).
- Fiziksel doğruluk: kuvvet yönleri (sürtünme göreceli hareketin tersi), kuvvet okları için tutarlı ölçek `FS`, konumlar analitik çözümden, özel açılar için ders kitabı yaklaşımları (sin37° = 0.6).
- Tüm içerik y ≤ 860 içinde. Grafikler solda, türetme sağda `board()` ile, alt simgeler için `bm()`.

## 11 Adım
1. Fizik yöntemiyle çözün, çözüm kartı yazın (physics-solving referansını okuyun). Çok bölümlü veya sembolik cevaplar için sympy kullanın.
2. Proje oluştur + ortam kontrolü (`new_video.sh` + `setup_check.sh`). Eksik GLM_API_KEY için kullanıcıyı yönlendirin veya edge-tts yedeklemesini kullanın.
3. Kısa bir Türkçe/İngilizce cümleyle TTS'yi test edin.
4. `problem.png` + vurgu kutularını hazırlayın.
5. `episode.json` + `script.json` yazın (sahneler çözüm akışını takip eder: intro → kuvvetler/modelleme → her süreç bölümü → kritik yargı → doğrulama → outro).
6. `storyboard.md` yazın (üstte çözüm kartı + şekil listesi; her satır için işaret et/hareket et/bırak; fizik eylem tablosundan eylem seçin).
7. Telaffuz ve script kontrolü 0'a.
8. `anim.js` yazın (PROBLEM, TAGS, fizik sabitleri, analitik çözümler `sPos(t)`, `vel(t)`, ölçek sabitleri `FS`/`VS`, her sahne için `SC.<id>`).
9. Ücretsiz önizleme: motion + stills + contact sheet. STATIC olanları düzeltin.
10. Gerçek TTS + ASR + stills'i yeniden kontrol edin.
11. Videoyu render edin (`node render.mjs video 6`) ve yolları, süreyi, sahne içeriklerini, cevabı teslim edin. Kullanıcıya telaffuz için dinlemesini hatırlatın.

## Yaygın hatalar
Modelleme/bölümlendirmeyi atlamayın, sürtünmeyi yalnızca hızın tersi olarak çizmeyin (yön değişiminde tersine dönmelidir), nesne konumlarını lerp etmeyin (analitik çözüm + doğrusal zaman kullanın), kuvvet oku uzunluklarını uydurmayın (tutarlı FS kullanın), zamanları sabit kodlamayın, her zaman ekran görüntülerini kontrol edin vb.

## Referans dosyaları
physics-solving.md, visual-design.md, animation.md, script-writing.md, pronunciation.md, glm-tts-setup.md, troubleshooting.md.
