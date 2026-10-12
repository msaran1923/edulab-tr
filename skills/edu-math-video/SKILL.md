---
name: edu-math-video
description: >-
  Bir matematik problemi için açıklayıcı / çözüm videosu istendiğinde kullanın (geometri, cebir, fonksiyonlar, hareket problemleri), problem ekran görüntüsü veya metinden, Türkçe (veya İngilizce) seslendirme (edge-tts veya uyumlu TTS / GLM-TTS), çift dilli tr+en altyazılar ve el çizimi canvas animasyonu ile MP4'e render edilmiş.
  Tetikleyici kelimeler: matematik açıklayıcı video, çözüm videosu, problem çözme videosu, mikro ders, bu matematik problemini video olarak çöz; math explainer video, walkthrough video.
---

# Matematik Problemi Açıklayıcı Videosu (edu-math-video)

Bir matematik problemini 16:9, 1920×1080 MP4'e dönüştürür: Türkçe/İngilizce seslendirme + çift dilli altyazılar + el çizimi defter tarzı animasyon.

Boru hattı (her video için bir klasör):

```
script.json ──build_audio.py──► build/timeline.json + build/mix.wav + ../<name>.srt
     (anlatım)     (TTS + sentezlenmiş müzik)          │
anim.js + engine.js ◄─────────────────────────┘ (animasyon anlatım zaman damgalarıyla yönlendirilir)
     └──node render.mjs video──► ../<name>.mp4
```

**Temel ilke: resim anlatımı takip eder ve şekil "problemi açıklar."** Her anlatım satırı için şekilde o akıl yürütme satırını gösteren bir eylem vardır (eşit segmentler birbirinin üzerine kayar, eş üçgenler üst üste biner, kamera üst görünüme tween eder, hareketli nokta hareket eder…). Her animasyon `S.at(k, f)` ile zamanlanır ("k-inci anlatım satırının başladığı an"), asla sabit saniyelerle değil.

**Bu yetenek tüm kodu içerir**; diğer projelerden dosya kopyalamayın:
- `template/`: tam çalıştırılabilir örnek proje (dik üçgen hipotenüse medyan problemi). `engine.js` genel motordur (değiştirmeyin), `anim.js` / `script.json` / `episode.json` / `problem.png` her problem için yeniden yazılmalıdır.
- `shared/pron.py` + `shared/pron.json`: telaffuz kontrolü.
- `examples/cone-parallel/`: tam katı geometri örneği.
- `scripts/`: `new_video.sh`, `setup_check.sh`, `make_problem_png.py`, `problem_boxes.py`, `contact_sheet.py`.

Aşağıda `SKILL` = bu dosyanın bulunduğu dizin, `WS` = **kullanıcının geçerli çalışma dizini (`$PWD`)**: video klasörü burada oluşturulur, mp4/srt de buraya çıkar. **Başka bir dizinde `.env`, `node_modules` veya `pron.json` olduğu için WS'yi değiştirmeyin**. `PROJ` = bu videonun klasörü, `PY` = `setup_check.sh` tarafından bildirilen python.

## Uyulması gereken kurallar

**Sıfır olmayan çıkış kodu bitmedi demektir.** `--check` çıkış kodu 1 (tek bir ⚠ çok sesli bile kalsa), `motion` çıkış kodu 1 veya ekran görüntülerinde sorun varsa devam etmeden önce düzeltin.

1. **Aşağıdaki 11 adımı sırayla sıkı şekilde takip edin; her adımın "geçme kriteri" karşılanmadan sonraki adıma geçmeyin.**
2. **Önce problemi net şekilde açıklayın.** İlk sahne orijinal problem görüntüsünü göstermelidir, anlatım her koşulu okur ve okundukça şekilde kutulanır.
3. **`tts` alanı konuşulabilir metin olmalıdır.** Arap rakamları, `= + − × ÷ / ^ √ ∠ △ ∥ ° ( )` vb. semboller olmamalıdır: `AC=3` `AC üç eşit` olarak yazın, `x²` `x kare` olarak. Altyazı alanı normal matematiksel notasyonu korur.
4. **Telaffuz sabitlenmelidir** (özellikle GLM-TTS için). TTS çağrısından önce `--check` `sabitlenmemiş nadir/çok sesli karakterler: 0` ve `script errors: 0` göstermelidir.
5. **API Key yalnızca kullanıcı tarafından sağlanır.** `GLM_API_KEY` eksikse önce kullanıcıya sorun: key'i yapılandırın (en iyi kalite) veya yedek motoru kullanın (edge-tts). Key icat etmeyin, yazdırmayın.
6. **Tüm görsel içerik y ≤ 860 içinde kalmalıdır.**
7. **Önce storyboard yazın, sonra animasyonu.**
8. **Para harcamadan önce ücretsiz önizleme yapın.**
9. **Görüntülere bakarak doğrulayın.**
10. **`engine.js`'yi değiştirmeyin** (kullanıcı stil değişikliği istemedikçe).
11. **`node_modules` veya `package*.json`'u video klasörüne kopyalamayın.**

## 11 Adım

### Adım 1: Problemi anlayın ve kendiniz doğru çözün
- Problem metnini alın (ekran görüntüsü varsa önce okuyun). **Kendiniz tamamen çözün ve doğrulayın**.
- Cevap belirsizse veya problem net değilse kullanıcıya sorun; tahmin etmeyin.
- **Geçme kriteri:** 3–6 "sahne"de açıklayabilirsiniz; cevap doğrulanmıştır.

### Adım 2: Projeyi oluşturun ve ortamı kontrol edin
```bash
bash SKILL/scripts/new_video.sh "$PWD" <ascii_folder_name>
bash SKILL/scripts/setup_check.sh WS/<ascii_folder_name>
```
- `GLM_API_KEY` eksikse: **durun ve kullanıcıyı yönlendirin**. Key yoksa `TTS_ENGINE=auto` otomatik yedekler; edge-tts kurmadan önce onay alın.
- **Geçme kriteri:** `setup_check.sh` `ALL OK` çıktısı verir.

### Adım 3: TTS'yi test edin
```bash
cd PROJ && PY build_audio.py --say "Merhaba, bir matematik problemine bakalım."
```
- **Geçme kriteri:** Bir wav oluşturuldu.

### Adım 4: Problem görüntüsü `problem.png` ve vurgu kutularını hazırlayın
- **Yalnızca metin**: `make_problem_png.py` kullanın.
- **Ekran görüntüsü**: `problem_boxes.py` ile işleyin.
- **Geçme kriteri:** Metin tam, her kutu karşılık gelen metni tam kaplıyor.

### Adım 5: `episode.json` ve `script.json` yazın
- İlk sahne `intro` problemi okur, son sahne `outro` gözden geçirir + cevabı bildirir.
- **Geçme kriteri:** Geçerli JSON; her sahne id'sinin planlanmış bir resmi var.

### Adım 6: Storyboard `storyboard.md` yazın
Her satır için "işaret et / hareket et / bırak" planlayın. "Hareket" akıl yürütmeyi gösteren anlamlı bir eylem olmalıdır.
- **Geçme kriteri:** Her satırın bir satırı var; şekil listesi tam.

### Adım 7: Telaffuz ve script kontrolü, 0'a düzeltin
```bash
cd PROJ && PY build_audio.py --check ; cat build/pron_report.txt
```
- **Geçme kriteri:** Sabitlenmemiş karakterler 0 ve script errors 0.

### Adım 8: `anim.js` yazın
Storyboard'a göre uygulayın.
- **Geçme kriteri:** `--check` ERROR yok.

### Adım 9: Ücretsiz önizleme: hareket kontrolü + düzen kontrolü
```bash
cd PROJ && PY build_audio.py --preview && node render.mjs motion && node render.mjs stills auto && PY SKILL/scripts/contact_sheet.py .
```
- **Geçme kriteri:** `motion check passed`; tüm ekran görüntüleri kontrol edildi.

### Adım 10: Gerçek sesi üretin, ASR kontrolü, görselleri yeniden kontrol edin
```bash
cd PROJ && PY build_audio.py && PY build_audio.py --asr && node render.mjs stills auto && PY SKILL/scripts/contact_sheet.py .
```
- **Geçme kriteri:** `mix + srt written`.

### Adım 11: Videoyu render edin ve teslim edin
```bash
cd PROJ && node render.mjs video 6
```
- Çıktı `WS/<output_name>.mp4` ve `.srt`. Kullanıcıya yolları, süreyi, her sahnenin içeriğini söyleyin ve sesi dinleyemediğinizi belirtin, telaffuzu gözden geçirmelerini isteyin.

## Yaygın hatalar
(Orijinaldeki tabloya bakın; aynı ilkeler geçerlidir: zamanları sabit kodlamayın, STATIC satır bırakmayın, API key icat etmeyin, her zaman ekran görüntülerini kontrol edin vb.)

## Referans dosyaları
Orijinal yetenekteki aynı reference/*.md dosyaları seti.
