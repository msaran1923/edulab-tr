# edulab

**Türkçe** · [English](../edulab-en/README.md) · [简体中文](https://github.com/wy51ai/edulab/blob/master/README.zh-CN.md)

> **Yerelleştirme notu**: Bu, orijinal [wy51ai/edulab](https://github.com/wy51ai/edulab) projesinin Türkçe yerelleştirilmiş sürümüdür. Yetenek açıklamaları, tetikleyici kelimeler ve iş akışı talimatları çevrilmiştir. Temel hesaplama çekirdekleri, şablonlar ve medya dosyaları değişmeden kalmıştır.

Akademik problemleri **etkileşimli ders web sayfalarına** ve **anlatımlı açıklayıcı videolara** dönüştüren eğitim yetenekleri koleksiyonu.

## Nasıl kullanılır

1. **Yetenekleri kurun** (birini seçin):
   ```bash
   npx skills add msaran1923/edulab-tr
   ```
   Veya Claude Code eklentisi olarak:
   ```
   /plugin marketplace add msaran1923/edulab-tr
   /plugin install edulab
   ```

2. **Bir yeteneği tetikleyin** — Türkçe olarak ne istediğinizi tarif edin, örneğin:
   - “Bu küp problemi için etkileşimli bir katı geometri sayfası oluştur”
   - “Bu elips kiriş uzunluğu problemi için analitik geometri çözüm sayfası yap”
   - “Bu matematik problemini açıklayıcı bir videoya dönüştür”

3. **Ajan yetenek iş akışını takip eder**: sympy çekirdeği ile kesin çözer, kendi içinde bağımsız bir HTML sayfası (veya video) oluşturur ve sonucu geçerli dizininize teslim eder.

4. **Videolar için**: Türkçe ses kullanmak için `TTS_ENGINE=edge` ve bir Türkçe ses (ör. `tr-TR-EmelNeural`) ayarlayın.

## Kurulum

**Önerilen** — [skills](https://github.com/vercel-labs/skills) ile tek komutla kurun:

```bash
npx skills add msaran1923/edulab-tr
```

Daha sonra en son sürüme güncellemek için:

```bash
npx skills update
```

> **Not:** `npx skills update` yalnızca **zaten kurulu** olan yetenekleri yeniler — depoya yeni eklenen yetenekleri otomatik olarak çekmez. Bu depo yeni bir yetenek kazandığında (ör. yeni bir `edu-*`), tekrar `npx skills add msaran1923/edulab-tr` çalıştırın.

Veya Claude Code eklenti pazarı olarak kullanın:

```
/plugin marketplace add msaran1923/edulab-tr
/plugin install edulab
```

Kurulduktan sonra yetenekler tetikleyici kelimelerinde otomatik etkinleşir ve manuel olarak da çağrılabilir.

## Yetenek: edu-solid-geometry

![edu-solid-geometry demo](edu-solid-geometry.gif)

Bir katı geometri problemini kendi içinde bağımsız etkileşimli bir ders sayfasına çözer. Üç giriş noktası:

| Giriş noktası | Ne yapar |
|---|---|
| Metin problemi | İfadeyi çıkarır ve doğrudan çözer |
| Görüntü yükleme | Görüntüden problemi görsel ile okur, onay için geri yansıtır, sonra çözer |
| Rastgele problem | Rastgele parametrelerle çözer; cevap temiz değilse yeniden çeker |

**Kapsanan problem türleri**: doğru-düzlem açısı, dihedral açı, aykırı doğrular arasındaki açı, noktadan düzleme uzaklık, hacim ve daha fazlası — küpler / dikdörtgenler prizmaları, piramitler / prizmalar, silindirler / koniler üzerinde. Hepsi "koordinat sistemi + vektör yöntemi" ile birleşik çözülür.

**Tetikleyici kelimeler**: katı geometri, doğru-düzlem açısı, dihedral açı, aykırı doğrular arasındaki açı, düzleme uzaklık, etkileşimli geometri çözüm sayfası; solid geometry, line-plane angle, dihedral angle, distance to plane, interactive geometry solution page vb.

### Bağımlılık

Hesaplama çekirdeği `lib/geometry_kernel.py` **sympy**'ye bağımlıdır. Sympy içe aktarabilen herhangi bir `python3` kullanın:

```bash
python3 -m pip install sympy   # sympy eksikse
```

### Komut satırından üret (Claude olmadan)

```bash
cd skills/edu-solid-geometry
python3 scripts/generate.py cube   ./cube.html     # küp · doğru-düzlem açısı
python3 scripts/generate.py box    ./box.html      # dikdörtgenler prizması · hacim
python3 scripts/generate.py random 7 ./random.html # rastgele problem (seed=7)
python3 lib/geometry_kernel.py                     # kernel yerleşik kendi kendini kontrol
```

> Çıktı yolu vermezseniz **geçerli çalışma dizinine (cwd)** yazar.

## Yetenek: edu-analytic-geometry

![edu-analytic-geometry demo](edu-analytic-geometry.gif)

Bir analitik geometri (konik kesitler) problemini kendi içinde bağımsız etkileşimli bir ders sayfasına çözer. Yukarıdakiyle aynı üç giriş noktası (metin / görüntü / rastgele). **2D Canvas tahtası + KaTeX** üzerine kurulu, genel, veri odaklı etkileşimli motor: bir parametre kaydırıcısı türetilmiş yapıları (doğru∩konik, konik üzerindeki nokta, merkezi yansıma, teğet…) ve canlı okumaları yönlendirir, teorik aralık çubuğu veya sabit değer göstergesi ile.

**Kapsanan problem türleri**: standart denklem, kiriş uzunluğu, nokta çarpım aralığı / sabit değer, üçgen alan ekstremumu, sabit nokta, sabit değer (eğim çarpımı), geometrik yer, teğet, eksantriklik — elipsler / hiperboller / paraboller / daireler üzerinde. Hepsi "parametreli doğru `x = my + c` + sistem + Vieta formülleri + yerine koyma" ile birleşik çözülür.

**Tetikleyici kelimeler**: analitik geometri, konik kesitler, elips, hiperbol, parabol, kiriş uzunluğu, nokta çarpım aralığı, sabit nokta, sabit değer, geometrik yer, eksantriklik, etkileşimli analitik geometri çözüm sayfası; analytic geometry, conic sections, ellipse, hyperbola, parabola, chord length, dot product range, fixed point, fixed value, locus, eccentricity, interactive analytic geometry solution page vb.

### Bağımlılık

Hesaplama çekirdeği `lib/analytic_kernel.py` **sympy**'ye bağımlıdır (yukarıdakiyle aynı).

### Komut satırından üret (Claude olmadan)

```bash
cd skills/edu-analytic-geometry
python3 scripts/generate.py list                          # kayıtlı problem türlerini listele
python3 scripts/generate.py ellipse_dot_range ./sol.html  # elips · MA·MB aralığı [-3, 7/4]
python3 scripts/generate.py parabola_dot_const ./sol.html # parabol odak kirişi · OA·OB ≡ -3
python3 scripts/generate.py all ./out_dir                 # tüm kayıtlı türler
python3 lib/analytic_kernel.py                            # kernel yerleşik kendi kendini kontrol
```

> Yukarıdaki gibi, çıktı yolu yoksa **geçerli çalışma dizinine (cwd)** yazar.

## Yetenek: edu-chem-reaction

Bir kimyasal reaksiyonu kendi içinde bağımsız **mikroskopik 3D gösterim** sayfasına dönüştürür: etkileşimli Three.js moleküler animasyon (kaydırıcıyı sürükleyerek bağların kırılmasını / oluşmasını ve atomların yeniden birleşmesini izleyin, adım vurgulama ile) yanında KaTeX denklemi, adım adım anlatım, atom korunumu sayacı ve isteğe bağlı enerji–reaksiyon koordinatı eğrisi. Aynı üç giriş noktası (metin / görüntü / rastgele).

**Tetikleyici kelimeler**: kimyasal reaksiyon, mikroskopik / moleküler animasyon, yanma, elektroliz, redoks elektron transferi, esterleşme mekanizması, bağ kırılması ve oluşması, atom korunumu, denklemi dengele, etkileşimli kimya reaksiyon sayfası; chemistry reaction, microscopic molecular animation, combustion, electrolysis, redox, esterification, bond breaking and forming, atom conservation, balance equation, interactive chemistry reaction page vb.

## Yetenek: edu-math-video

Bir matematik problemini (geometri, cebir, fonksiyonlar, hareket problemleri…) **16:9 1920×1080 açıklayıcı MP4**'e dönüştürür: Türkçe seslendirme (edge-tts veya uyumlu TTS), çift dilli tr + en altyazılar (`.srt` de), anlatım zaman çizelgesi tarafından yönlendirilen el çizimi defter tarzı canvas animasyonu. Girdi problem ekran görüntüsü veya düz metindir.

**Not**: Orijinal Çin seslendirmesi (GLM-TTS) yerine Türkçe için `TTS_ENGINE=edge` ve Türkçe ses (ör. `tr-TR-EmelNeural`) kullanın.

**Tetikleyici kelimeler**: matematik açıklayıcı video, çözüm videosu, problem çözme videosu, mikro ders; math explainer video, walkthrough video, problem-solving video, micro-lesson vb.

## Yetenek: edu-physics-video ve edu-chem-video

Benzer video boru hattı, fizik ve kimya problemleri için. Aynı TTS uyarlamaları geçerlidir.

## Nasıl çalışır

1. **Problem spec'i al** — üç giriş noktasını yapılandırılmış bir açıklamaya normalleştirin (gövde tipi ve boyutlar, verilen koşullar, istenen miktar, dil).
2. **Kesin kernel hesaplaması** — sympy kesin koordinatları, anahtar vektörleri, normalleri, nihai cevabı ve her ara değeri hesaplar (LaTeX dizeleri olarak). Asla elle.
3. **Şablonu birleştir ve enjekte et** — `lesson` / `steps` / `model` verilerini veri odaklı şablona besleyin; 3D tepe koordinatları `kernel.to_three(...)`'den gelir, çözümle aynı kaynağı paylaşır.
4. **Kendi kendini kontrol** — kernel cevabı == cevap kartı == son adımda gösterilen nihai değer; yerel statik sunucu + önizleme kontrolü konsol hatası olmadığını ve doğru formül/vurgulama render'ını doğrular.
5. **Teslim et** — bitmiş sayfa kullanıcının geçerli çalışma dizinine yazılır, `solution-<kisa-aciklama>.html` gibi adlandırılır.

## Lisans

Apache-2.0

Orijinal proje: https://github.com/wy51ai/edulab
