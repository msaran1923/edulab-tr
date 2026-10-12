---
name: edu-analytic-geometry
description: >-
  Bir analitik geometri (konik kesitler) problemini kendi içinde bağımsız etkileşimli bir ders web sayfasına dönüştürür: sol panel problem ifadesi + dinamik konsol (değişken parametre kaydırıcısı geometrik miktarların canlı yeniden hesaplanmasını + teorik aralık / sabit değer göstergesini yönlendirir), orta panel KaTeX adım adım çözüm, sağ panel 2D Canvas dinamik geometri tahtası (elips/hiperbol/parabol/daire + hareketli doğru/nokta + vektörler + etiketler + çizim araçları). Üç giriş noktasını destekler — verilen metin problemi, rastgele problem veya görüntü yükleme ile tanıma sonrası çözüm. Standart denklemler, kiriş uzunluğu, vektör nokta çarpım aralığı/sabit değer, üçgen alan ekstremumu, sabit nokta, sabit değer (eğim çarpımı), geometrik yer, eksantriklik vb. kapsar. Birleşik "parametreli doğru x=my+c + sistem + Vieta + yerine koyma" yöntemi, kesin sympy hesaplamasıyla yönlendirilir (cevap, koordinatlar, adım değerleri ve etkileşimli motorun teorik aralığı aynı kaynaktan gelir). Diğer ajanlar da bu yeteneği çağırarak bu tür sayfalar üretebilir. Biçim olarak edu-solid-geometry'ye paraleldir, ancak Canvas-2D + KaTeX kullanır (Three.js değil).
  Tetikleyici kelimeler: analitik geometri, konik kesitler, elips, hiperbol, parabol, doğru ve konik, odak kirişi, kiriş uzunluğu, nokta çarpım aralığı, vektör nokta çarpımı, sabit nokta, sabit değer, eğim çarpımı, üçgen alan maksimumu, geometrik yer denklemi, eksantriklik, bu analitik geometri problemini çöz, rastgele konik problemi, bu resimdeki analitik geometri; analytic geometry, interactive analytic geometry solution page.
---

# Analitik Geometri Problemi → Etkileşimli Web Sayfası

## Bu yetenek ne üretir
Tarayıcıda doğrudan açılabilen tek bir HTML sayfası (üç sütun):
- **Sol sütun**: problem ifadesi + dinamik konsol — bir değişken parametre kaydırıcısı (ör. doğru eğim açısı θ / hareketli nokta parametresi t) geometrik miktarların canlı yeniden hesaplanmasını yönlendirir (kesişim koordinatları, eğimler, nokta çarpımları, kiriş uzunlukları, alanlar…), artı bir "teorik aralık çubuğu" veya "sabit değer göstergesi".
- **Orta sütun**: adım adım çözüm (formüller **KaTeX** ile), tahtaya daha fazla yer vermek için tek tıkla daraltılabilir.
- **Sağ sütun**: 2D Canvas dinamik geometri tahtası (konik + hareketli doğru/nokta + vektörler + nokta etiketleri + ızgara eksenleri), üzerine çizim/karalama araç çubuğu eklenmiş.

## Bağımlılıklar (önemli)
Hesaplama çekirdeği `lib/analytic_kernel.py` **sympy**'ye bağımlıdır. Betikleri çalıştırmadan önce sympy içe aktarabilen bir `python3` olduğunu doğrulayın: `python3 -c "import sympy"`.

**Eksik olduğunda**: İçe aktarma başarısız olursa (sympy veya sonraki herhangi bir kütüphane), **önce kullanıcıya yüklemek isteyip istemediğini sorun**, onay aldıktan sonra yükleyin (`python3 -m pip install <paket>`) veya zaten yüklü bir yorumlayıcıya geçin; **sormadan yüklemeyin**.
Aşağıdaki tüm `python3` bağımlılıkları çalıştırabilen bir yorumlayıcıyı ifade eder.

## İş akışı

### Adım 1: Problem spesifikasyonunu elde et (üç giriş noktasını birleştir)
Problemi yapılandırılmış bir spec'e dönüştürün (eğri tipi ve parametreler, verilen noktalar/koşullar, istenen tip ve nesneler, dil).
- **Metin problemi**: doğrudan çıkar.
- **Görüntü**: görsel ile çıkar ve **tanınan problemi kullanıcıya onay için geri yansıt** (ifade/eğri/parametreler/istenen/dil) devam etmeden önce.
- **Rastgele problem**: eğri + problem tipi seç, rastgele parametreler → kernel çöz, `analytic_kernel.is_clean(...)` ile cevabın temiz olup olmadığını yargıla; değilse yeniden çek.

> **Çıktı dili istem dilini takip eder**: Türkçe istem → Türkçe sayfa vb. Spec'e `language` kaydedin.

### Adım 2: Kernel ile kesin hesaplama (elle hesaplamayın)
`references/conventions.md`'deki çözüm tariflerini takip ederek `lib/analytic_kernel.py` ve `lib/conics.py` çağırın:
- `conics.ellipse/hyperbola/parabola/circle(...)` eğri nesnesini verir (kesin a,b,c, odaklar, tepe noktaları, doğrultmanlar, asimptotlar, `eq_latex` ve ön uç motoru için `board` dict).
- `chord_setup(conic, through)` parametreli doğru `x=my+c` sistemini kurarak y cinsinden kuadratik + Vieta miktarlarını (kesin) elde eder.
- Hedef miktarlar: `dot_product_expr` / `chord_len_sq_expr` / `triangle_area_expr` / `slope_product_central` …
- Aralık: `range_over_m(expr, horizontal_valid=?)` — **açık/kapalı uç noktası kararını içerir** (kritik doğruluk noktası, aşağıya bakın).
- Sabit değer: `is_constant_in_m(expr)`.

Kernel'i komut satırından kendi kendini kontrol edebilirsiniz:
```bash
python3 lib/analytic_kernel.py      # amiral gemisi problemler için yerleşik assert kendi kendini kontrol
```

> ⚠️ **Açık/kapalı uç noktalar = doğruluk kritik**: Odak noktasından geçen bir kiriş için hem yatay doğru (x-ekseni, θ=0) hem de dikey doğru (θ=90) geçerli doğrulardır; aldıkları uç noktalar dahil edilmelidir. Örnek: elips MA·MB problemi, x-ekseni −3 alır, dikey doğru 7/4 alır, dolayısıyla cevap **kapalı aralık** `[-3, 7/4]`'tür (birçok ders kitabı yanlışlıkla açık `(-3, 7/4]` yazar). `range_over_m` buna göre karar verir ve cevap etkileşimli araçla eşleşir — kaydırıcıyı 0°'ye sürükleyin ve tam olarak −3 okursunuz. Parabolik odak kirişi için "eksen yönü" dejenere bir doğrudur (yalnızca bir noktada kesişir); sınır uç noktası sayılmaz (`horizontal_valid=False` veya param aralığını kısıtla).

### Adım 3: Veriyi birleştir ve şablona enjekte et

> 📍 **Çıktı konumu ve tek teslimat (en önemli)**: Kullanıcıya teslim edilen **tek** şey bir `.html` dosyasıdır, **geçerli çalışma dizinine (`Path.cwd()`)** yazılır (kullanıcı açıkça bir yol belirtmedikçe). cwd'de **başka hiçbir dosya bırakmayın** — derleme betikleri (`.py`), `__pycache__`, kendi kendini kontrol ekran görüntüleri (`.png`), geçici dosyalar **teslimat değildir**; bunları `/tmp`'ye koyun veya kullanımdan sonra silin.
> Ayrıca **asla** yeteneğin kendi dizinine yazmayın (`skills/edu-analytic-geometry/output/` yeteneğin iç örnek dizinidir).

"Veriyi birleştir + şablona enjekte et" **derleme betiğini geçici bir dizine yazın** (ör. `/tmp/ag_build.py`) ve **yalnızca `.html`'i cwd'ye yazmasını** sağlayın; betik `lesson` / `steps` / `board` verilerini birleştirir (`references/problem-schema.md` şeması), `generate.render_html(data, out)` çağırarak `template/board.html`'e enjekte eder, **ardından betiği silin**:

```python
# Derleme betiği /tmp'ye gider (cwd'ye değil): /tmp/ag_build.py
import sys; sys.dont_write_bytecode = True            # __pycache__ oluşturma
sys.path.insert(0, "<yetenek-dizini>/scripts")
import generate
from pathlib import Path
data = {"lesson": {...}, "steps": [...], "board": {...}}
out = Path.cwd() / "solution-<kisa-aciklama>.html"   # tek teslimat, kullanıcının geçerli dizinine düşer
generate.render_html(data, out)
```

```bash
python3 -B /tmp/ag_build.py && rm -f /tmp/ag_build.py   # -B bayt kodu yok; çalıştırdıktan sonra geçici betiği sil, cwd'de yalnızca .html kalsın
```

- `steps[*].content` içindeki sayısal değerler **doğrudan kernel sonuçlarına referans verir** (LaTeX için `K.tex(...)` kullanın); model yalnızca açıklayıcı metni düzenler (hedef dilde).
- `board` kernel'den gelen eğri `board` dict'ini, kesin nokta koordinatlarını, `param`'ı, `derived` yapı sırasını, `readouts`'ı, `rangeBar` (aralık problemleri) / `constant` (sabit değer problemleri) / `answerBand` (**şekil parametresi problemleri**, ör. eksantriklik aralığı) kullanır.
- **Şekil parametresi problemleri (kaydırıcı = eksantriklik e vb.)**: Doğal dinamik miktar hareketli doğru/nokta değil de eğrinin kendisinin şekli olduğunda, kaydırıcıyı = o parametre yapın, eğrinin `a/b/c`, odaklarını ve hareketli nokta koordinatlarını `@param`'ın **ifade dizeleri** olarak yazın (motor her karede eğriyi/odakları/asimptotları yeniden çizer), eşitsizlik durumunu gösteren bir `status` okuması ve parametre ekseninde cevap aralığını vurgulayan `answerBand` ile eşleştirin. conventions "şekil parametresi problemleri"ne bakın.
- **Doğrudan kopyalanabilir örnekler**: `scripts/generate.py` içindeki 6 `build_*` çeşitli etkileşim paradigmalarını kapsar:
  `ellipse_dot_range` (aralık çubuğu), `ellipse_chord_range`, `ellipse_area_max`,
  `ellipse_slopeprod_const` (sabit değer · merkezi simetri), `parabola_dot_const` (sabit değer · parabol),
  `hyperbola_ecc_range` (**şekil parametresi**: kaydırıcı=e, eğri yeniden çizilir + `status` + `answerBand`).

Kayıtlı problemler doğrudan üretilebilir (`-B` bayt kodu yok; yol yoksa yetenek çıktısına varsayılan — kullanıcıya teslim ederken her zaman cwd altındaki bir `.html`'e değiştirin):
```bash
python3 -B scripts/generate.py list                      # problem türlerini listele
python3 -B scripts/generate.py ellipse_dot_range ./sol.html
python3 -B scripts/generate.py all ./out_dir             # tüm türler
```

### Adım 4: Kendi kendini kontrol (doğruluk planı)
- kernel cevabı == cevap kartı `lesson.answer` == son adımda gösterilen nihai değer == **JS standart konum / tarama segmenti yeniden hesaplanan değer**, dördü tutarlı (`build_*` içinde zaten `assert` var).
- `rangeBar` uç noktaları kernel'in `range_over_m`'inden gelir; `constant` değeri kernel'in sabit değerinden gelir.
- Yerel bir statik sunucu başlatın (**çıktı dosyasının dizinini** servis eden, yani cwd) ve önizleme kontrolü yapın: konsol hatası yok, KaTeX doğru render, kaydırıcı canlı yeniden hesaplama doğru, aralık çubuğu / sabit değer / sabit nokta / geometrik yer davranışı eşleşiyor, çizim araçları ve daraltma paneli çalışıyor.
  (Yetenek deposu içinde geliştirirken `.claude/launch.json`'un `ag-preview`'i kullanılabilir, port 4601; başka yerde cwd üzerinde geçici bir statik sunucu başlatın.)
- **Kendi kendini kontrol ekran görüntüleri yalnızca sizin içindir**: önizleme aracı görüntüyü doğrudan döndürür, **`.png`'yi cwd'ye kaydetmeyin**; yerel statik sunucu salt okunurdur ve dosya üretmez. Kendi kendini kontrol tarafından üretilen herhangi bir geçici dosya (derleme betiği `.py`, ekran görüntüleri `.png`, `__pycache__` vb.) teslimattan önce temizlenmelidir.

> ⚠️ **Açtığınız portları/hizmetleri mutlaka kapatın**: Önizlemeden hemen sonra yerel sunucuyu durdurun, **asla port işgal eden süreçler bırakmayın**.
> - Önizleme aracıyla açılan: `preview_stop` (serverId geçirin).
> - Doğrudan `http.server`: işiniz bitince `kill` edin veya `lsof -nP -iTCP:<port> -sTCP:LISTEN` ile serbest bırakıldığını doğrulayın.
> - Teslimattan önce portun serbest bırakıldığını doğrulayın, ardından kullanıcıya söyleyin. Açıp kapatmamak = tamamlanmamış kendi kendini kontrol.

### Adım 5: Teslimat
Bitmiş ürün **kullanıcının geçerli çalışma dizinine (cwd)** yazılır, `solution-<kisa-aciklama>.html` gibi adlandırılır, kullanıcıya yolu söyleyin, tarayıcıda doğrudan açılabilir.
Teslimattan önce doğrulayın: **(1)** bitmiş ürün cwd'de, yetenek dizininde değil; **(2)** bu önizlemeden açık kalan yerel hizmet/port yok; **(3)** cwd'de **yalnızca bu yeni bir `.html`** var — `.py` / `.png` / `__pycache__` / geçici dosya yok (`git status` veya `ls` ile kontrol edin ve varsa silin).

## Genişletme
- **Problem türleri ekle**: `analytic_kernel.py`'de hedef miktar fonksiyonları ekleyin (m cinsinden ifadeler olarak yazılmış) + `range_over_m` / `is_constant_in_m` yeniden kullanın; `generate.py`'de bir `build_*` ekleyin, bir etkileşim paradigması seçin (aralık çubuğu / sabit değer / sabit nokta / geometrik yer izi / şekil parametresi answerBand). `references/conventions.md`'deki tarif tablosuna bakın.
- **Eğriler ekle**: `conics.py`'de zaten elips/hiperbol/parabol/daire var; ön uç `board.html` motoru dört tür render, asimptotlar ve doğrultman yönlerini zaten destekliyor. Yeni bir eğri için her yerde bir kopya ekleyin.
- **Etkileşimli yapılar ekle**: `board.html`'deki `buildScene` switch yapı kütüphanesidir (`line_through_angle`, `intersect_line_conic`, `point_on_conic`, `point_reflect`, `tangent_at`, `foot_perp`…); gerektiğinde genişletin ve şema belgesine kaydedin.

## Dizin
- `template/board.html` — veri odaklı şablon (genel 2D renderlayıcı + parametre motoru + veri adası `__LESSON_DATA__`)
- `lib/conics.py` — konik sympy tanım kütüphanesi (özel noktalar / LaTeX / board dict)
- `lib/analytic_kernel.py` — sympy kesin çözücü çekirdeği (sistem · Vieta · aralık · sabit değer)
- `scripts/generate.py` — şablon enjeksiyonu + 5 build_* örneği + toplu/tek problem üretimi
- `references/problem-schema.md` — veri formatı (board motoru şeması)
- `references/conventions.md` — standart formlar, çözüm tarif tablosu, Vieta/yerine koyma rutinleri, açık/kapalı uç noktalar, kendi kendini kontrol
