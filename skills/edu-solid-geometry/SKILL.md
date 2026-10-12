---
name: edu-solid-geometry
description: >-
  Bir katı geometri problemini kendi içinde bağımsız etkileşimli bir ders web sayfasına dönüştürür:
  sol tarafta MathJax adım adım çözüm, sağ tarafta Three.js etkileşimli 3D model
  (adım vurgulama + kamera değişimleri). Üç giriş noktasını destekler — verilen metin problemi,
  rastgele problem üretimi veya görüntü yükleme ile tanıma sonrası çözüm. Küp/dikdörtgenler prizması,
  piramit/prizma, silindir/koni üzerinde doğru-düzlem açısı, dihedral açı, aykırı doğrular açısı,
  noktadan düzleme uzaklık, hacim vb. konuları kapsar. Birleşik "koordinat sistemi + vektör yöntemi",
  kesin sympy hesaplamasıyla yönlendirilir (cevap, 3D koordinatlar, adım değerleri tutarlı). Diğer ajanlar
  da bu yeteneği çağırarak bu tür sayfalar üretebilir.
  Tetikleyici kelimeler: katı geometri, doğru-düzlem açısı, dihedral açı, aykırı doğrular arasındaki açı,
  düzleme uzaklık, düzgün kare piramit, küp açısı, bu geometri problemini çöz,
  rastgele katı geometri problemi, bu resimdeki katı geometri; solid geometry, interactive geometry solution page.
---

# Katı Geometri Problemi → Etkileşimli Web Sayfası

## Bu yetenek ne üretir
Tarayıcıda doğrudan açılabilen tek bir HTML sayfası: sol tarafta problem/cevap/adım adım çözüm (MathJax formülleri),
sağ tarafta ilgili 3D model (Three.js, döndürülebilir/yakınlaştırılabilir, temel öğelerin adım vurgulaması ve kamera değişimleri).
`template/lesson.html` yapısıyla uyumludur.

## Bağımlılıklar (önemli)
Hesaplama çekirdeği `lib/geometry_kernel.py` **sympy**'ye bağımlıdır. Betikleri çalıştırmadan önce sympy içe aktarabilen bir `python3` olduğunu doğrulayın:
`python3 -c "import sympy"`.

**Eksik olduğunda**: İçe aktarma başarısız olursa (sympy veya sonraki herhangi bir kütüphane), **önce kullanıcıya yüklemek isteyip istemediğini sorun**, onay aldıktan sonra yükleyin
(`python3 -m pip install <paket>`), veya zaten yüklü bir yorumlayıcıya geçin; **sormadan yüklemeyin**.
Aşağıdaki tüm `python3` bağımlılıkları çalıştırabilen bir yorumlayıcıyı ifade eder.

## İş akışı

### Adım 1: Problem spesifikasyonunu elde et (üç giriş noktasını birleştir)
Problemi yapılandırılmış bir spec'e dönüştürün (`references/problem-schema.md` formatı): katı tipi ve boyutlar,
verilen yapı noktaları/koşullar, istenen tip ve nesneler, **dil**.
- **Metin problemi**: doğrudan çıkar.
- **Görüntü**: görsel ile çıkar ve **tanınan problemi kullanıcıya onay için geri yansıt** (ifade/katı/boyutlar/istenen/dil) devam etmeden önce.
- **Rastgele problem**: katı ve tip seç, rastgele parametrelerle kernel kullan, cevap temiz değilse yeniden çek.

> **Çıktı dili istem dilini takip eder**: Türkçe istem → Türkçe sayfa vb. Spec'e `language` kaydedin.

### Adım 2: Kernel ile kesin hesaplama (elle hesaplamayın)
`references/conventions.md`'deki koordinat kuralları ve çözüm tariflerini takip ederek `lib/geometry_kernel.py` çağırın:
kesin koordinatlar, anahtar vektörler, normal vektörler, nihai cevap ve her adım için ara değerler (hepsi LaTeX dizeleri olarak).
Tepe noktalarının Three.js koordinatları `kernel.to_three(points, scale)` ile.

Önce komut satırından kernel'i doğrulayabilirsiniz:
```bash
python3 lib/geometry_kernel.py    # yerleşik örnek kendi kendini kontrol
```

### Adım 3: Ders verisini birleştir ve şablona enjekte et

> 📍 **Çıktı konumu (önemli)**: Bitmiş HTML her zaman **kullanıcının geçerli çalışma dizinine (`Path.cwd()`)** yazılır, kullanıcı açıkça bir yol belirtmedikçe.
> **Asla** yeteneğin kendi dizinine yazmayın (`skills/edu-solid-geometry/output/` vb.) — bu yeteneğin iç geliştirme örnek dizinidir.
> Geçici derleme betikleri de cwd veya geçici bir dizine (ör. `/tmp`) gider ve kullanımdan sonra silinebilir.

Kernel, bodies, generate'ı içe aktaran, `lesson` / `steps` / `model` verilerini birleştiren **geçici bir derleme betiği** yazın
(şema `references/problem-schema.md`'de), ardından `generate.render_html(data, out)` çağırarak şablonu enjekte edin ve HTML üretin.
`out` **cwd altında mutlak bir yol** kullanır:

```python
from pathlib import Path
out = Path.cwd() / "solution-<kisa-aciklama>.html"   # kullanıcının geçerli dizinine düşer, yetenek dizinine değil
generate.render_html(data, out)
```

- `steps[*].content` içindeki tüm sayısal değerler **doğrudan kernel sonuçlarına referans verir**; model yalnızca açıklayıcı metni düzenler (hedef dilde yazılır).
- `model.points` `kernel.to_three(...)` sonuçlarını kullanır; `model.spheres`/`edges` `lib/bodies.py`'den topoloji kullanır
  (`quad_pyramid` / `tri_pyramid` / `cuboid` / `cube` / `prism`), nadir katılar için kenarlar elle yazılabilir.
- Her adımın `highlight` (o adım için görünür öğelerin mutlak kümesi) ve `cameraPos` vardır.
- **Problem segment uzunlukları verdiğinde**: ilgili kenar için bir `measure` öğesi ekleyin (`label` LaTeX'te, ör. `2\sqrt{2}`),
  ve onu "koordinat sistemi kur / bilinen koşulları listele" adımının `highlight`'ına koyun, 3D figürde uzunluğu işaretleyin (problem-schema'ya bakın).
- Türkçe çıktı için `lesson.ui`'yi Türkçe metinlerle doldurun ve `lesson.language="tr"` ayarlayın.

**Doğrudan referans alınabilecek örnekler**: `scripts/generate.py` içindeki `build_data()` (düzgün kare piramit · doğru-düzlem açısı),
`build_cube_data()` (küp · doğru-düzlem açısı), `build_box_volume_data()` (dikdörtgenler prizması · hacim) tam şablonlardır — onları uyarlayın.

`generate.py` kayıtlı problemleri doğrudan çıkarabilir; **yol verilmediğinde geçerli çalışma dizinine (cwd) varsayılan** olarak yazar, veya cwd altında açık bir dosya adı verebilirsiniz
(yetenek dizininin `scripts/generate.py`'sini kullanarak, çıktı cwd'ye düşer):
```bash
python3 <yetenek-dizini>/scripts/generate.py cube ./cube.html
python3 <yetenek-dizini>/scripts/generate.py box  ./box.html
```

**Rastgele problemler**: `generate.py random <seed> [cikti.html]`, dahili olarak `kernel.is_clean(...)` ile cevabın temiz olup olmadığını yargılar, değilse yeniden çeker:
```bash
python3 <yetenek-dizini>/scripts/generate.py random 7 ./random.html   # yol yoksa ./random.html (cwd) varsayılan
```
Rastgele problem türlerini genişletirken "rastgele parametreler → çöz → is_clean değilse yeniden çek" izleyin.

### Adım 4: Kendi kendini kontrol (doğruluk planına karşılık)
- kernel cevabı == cevap kartı `answerValue` == son adımda gösterilen nihai değer (generate.py'de assert örnekleri vardır).
- 3D tepe koordinatları `kernel.to_three`'den gelir (çözümle aynı kaynak).
- Yerel bir statik sunucu başlatın (**çıktı dosyasının dizinini** servis eden, yani cwd) ve önizleme kontrolü yapın: konsol hatası yok, formüller doğru render, adım vurgulama/kamera beklentilere uyuyor.
  (Yetenek deposu içinde geliştirirken `.claude/launch.json`'un `geom-preview`'i kullanılabilir; başka yerde cwd üzerinde geçici bir statik sunucu başlatın.)

> ⚠️ **Açtığınız portları/hizmetleri mutlaka kapatın**: Önizleme kontrolünden hemen sonra yerel sunucuyu durdurun, **asla port işgal eden süreçler bırakmayın**.
> - Önizleme aracıyla açılan: kontrol ettikten hemen sonra `preview_stop` çağırın (ilgili serverId'yi geçirin).
> - Doğrudan `http.server`: işiniz bitince `kill` edin veya `lsof -nP -iTCP:<port> -sTCP:LISTEN` ile serbest bırakıldığını doğrulayın.
> - Teslimattan önce portun serbest bırakıldığını doğrulayın, ardından kullanıcıya sonucu söyleyin. Açıp kapatmamak = tamamlanmamış kendi kendini kontrol.

### Adım 5: Teslimat
Bitmiş ürün **kullanıcının geçerli çalışma dizinine (cwd)** yazılır, `solution-<kisa-aciklama>.html` gibi adlandırılır, kullanıcıya (cwd) yolu söyleyin, tarayıcıda doğrudan açılabilir.
Teslimattan önce doğrulayın: **(1)** bitmiş ürün cwd'de, yetenek dizininde değil; **(2)** bu önizlemeden açık kalan yerel hizmet/port yok.

## Genişletme
- **Problem türleri ekle**: `geometry_kernel.py`'de çözüm fonksiyonları ekleyin (conventions tarif tablosuna bakın), `generate.py`'de bir `build_*` ekleyin.
- **Katılar ekle**: `geometry_kernel.py`'de koordinat oluşturma fonksiyonları ekleyin, `bodies.py`'de kenar topolojisi ekleyin.

## Dizin
- `template/lesson.html` — veri odaklı şablon (genel 3D renderlayıcı + veri adası `__LESSON_DATA__`)
- `lib/geometry_kernel.py` — sympy kesin hesaplama çekirdeği
- `lib/bodies.py` — katı kenar topoloji kütüphanesi
- `scripts/generate.py` — şablon enjeksiyonu + örnek derleme fonksiyonları
- `references/problem-schema.md` — veri formatı
- `references/conventions.md` — koordinat kuralları, çözüm tarifleri, kendi kendini kontrol
