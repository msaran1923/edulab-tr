---
name: edu-chem-reaction
description: >-
  Bir kimyasal reaksiyonu kendi içinde bağımsız mikroskopik 3D etkileşimli bir gösterim web sayfasına dönüştürür: sol/üst Three.js etkileşimli moleküler animasyon (kaydırıcıyı sürükleyerek bağların kırılmasını · oluşmasını · atomların yeniden birleşmesini izleyin, adım vurgulama ile), sağ KaTeX reaksiyon denklemi + adım adım açıklama + atom korunumu sayacı + isteğe bağlı enerji–reaksiyon ilerleme eğrisi. Üç giriş noktasını destekler — verilen metin reaksiyon/denklem, rastgele problem veya görüntü yükleme ile tanıma sonrası gösterim. Yanma, birleşme/ayrışma/yer değiştirme/çift yer değiştirme, redoks (elektron transferi görselleştirme), organik mekanizmalar (esterleşme vb. katalizör · geçiş hali ile) kapsar. Sympy ile birleşik kesin yönlendirme: denklemi otomatik dengeler, atom korunumu ve atom haritasını doğrular, hangi bağların kırıldığını/oluştuğunu türetir; cevap ve geometri aynı kaynaktan gelir. Otomatik seçimli yerleşik çift motor: morph (atom morfı, atom korunumunu gösterir) ve mechanism (mekanizma anahtar kareleri, kataliz/geçiş halini gösterir). Hibrit geometri: varsayılan yerleşik VSEPR molekül kütüphanesi; RDKit yüklüyse SMILES'ten konformer üretebilir (asla otomatik yüklemez). Diğer ajanlar da bu yeteneği çağırarak bu tür sayfalar üretebilir. Biçim olarak edu-solid-geometry / edu-analytic-geometry'ye paraleldir, ancak moleküler reaksiyonları göstermek için Three.js + KaTeX kullanır.
  Tetikleyici kelimeler: kimyasal reaksiyon, mikroskopik gösterim, moleküler animasyon, yanma, metan yanması, hidrojen yanması, su elektrolizi, redoks, elektron transferi, esterleşme, reaksiyon mekanizması, bağ kırılması ve oluşması, atom korunumu, kütle korunumu, kimyasal denklem dengeleme, bu kimyasal reaksiyonu çöz, rastgele kimyasal reaksiyon, bu resimdeki kimyasal reaksiyon; chemistry reaction, interactive chemistry reaction page.
---

# Kimyasal Reaksiyon Mikroskopik Gösterimi → Etkileşimli Web Sayfası

## Bu yetenek ne üretir
Tarayıcıda doğrudan açılabilen tek bir HTML sayfası: bir taraf reaksiyona karşılık gelen **3D moleküler animasyon** (Three.js, döndürülebilir/yakınlaştırılabilir, "reaksiyon ilerleme" kaydırıcısını sürükleyerek kare kare **kimyasal bağların kırılmasını/oluşmasını ve atomların yeniden birleşmesini** izleyin, adım vurgulama + yüzen molekül etiketleri ile), diğer taraf **KaTeX reaksiyon denklemi + adım adım açıklama + atom korunumu sayacı**, ve isteğe bağlı olarak **enerji–reaksiyon ilerleme eğrisi**, alev, katalizör protonları, elektron transferi katmanları vb. `template/reaction.html` yapısıyla uyumludur.

## Bağımlılıklar (önemli)
Hesaplama çekirdeği `lib/reaction_kernel.py` **sympy**'ye bağımlıdır (dengeleme için). Betikleri çalıştırmadan önce sympy içe aktarabilen bir `python3` olduğunu doğrulayın: `python3 -c "import sympy"`.

**Eksik olduğunda**: İçe aktarma başarısız olursa (sympy veya sonraki herhangi bir kütüphane), **önce kullanıcıya yüklemek isteyip istemediğini sorun**, onay aldıktan sonra yükleyin (`python3 -m pip install <paket>`) veya yorumlayıcı değiştirin; **sormadan yüklemeyin**.
**RDKit isteğe bağlıdır**: yüklüyse hibrit geometri SMILES'ten gerçek konformerler üretmek için kullanır; yüklü değilse yerleşik VSEPR kütüphanesi kullanılır — ikisi de çalışır ve **bu yetenek asla RDKit'i otomatik olarak yüklemez**.

## İş akışı

### Adım 1: Reaksiyon spesifikasyonunu elde et (üç giriş noktasını birleştir)
Reaksiyonu yapılandırılmış bir spec'e dönüştürün (`references/problem-schema.md` formatı): reaktanlar/ürünler, atom haritası veya açık atomlar, koşullar (ateşleme/elektrifikasyon/kataliz/tersinir), kategori, adım adım açıklama, **dil**.
- **Metin reaksiyon/denklem**: reaktanları ve ürünleri doğrudan çıkar, dengelemek için kernel'i çağır.
- **Görüntü**: görsel ile denklemi çıkar ve **tanınan reaksiyonu kullanıcıya onay için geri yansıt** (denklem/koşullar/kategori/dil) devam etmeden önce.
- **Rastgele problem**: kayıttan bir reaksiyon seç veya kütüphane içindeki türleri birleştir ve `balanced_coefficients` ile dengele, yalnızca cevap temizse kullan.

> **Çıktı dili istem dilini takip eder**: Türkçe istem → Türkçe sayfa vb. Spec'e `meta.language` kaydedin.

### Adım 2: Kernel ile kesin hesaplama (elle hesaplamayın)
`references/conventions.md`'deki modelleme kurallarını takip ederek `lib/reaction_kernel.py` çağırın:
- `balanced_coefficients(...)` sympy null space kullanarak **otomatik dengeler** (denklem katsayıları garantilidir);
- `assemble_data(spec)` molekül örneklerini genişletir, **atom korunumu ve atom haritası bijeksiyonunu doğrular**, **hangi bağların kırıldığını/oluştuğunu (fark kümesi) türetir**, her atomun reaktan/ürün durumlarındaki dünya koordinatlarını hesaplar ve şablona enjekte edilecek `data`'yı üretir.

Önce komut satırından kendi kendini kontrol edebilirsiniz:
```bash
python3 lib/reaction_kernel.py     # dengeleme + korunum + bağ farkı kendi kendini kontrol
python3 lib/molecules.py           # molekül kütüphanesi kendi kendini kontrol
```

### Adım 3: Spec'i birleştirmek ve şablona enjekte etmek için build_* yaz

> 📍 **Çıktı konumu (önemli)**: Bitmiş HTML her zaman **kullanıcının geçerli çalışma dizinine (`Path.cwd()`)** yazılır, kullanıcı açıkça bir yol belirtmedikçe.
> **Asla** yeteneğin kendi dizinine yazmayın (`skills/edu-chem-reaction/output/` vb.) — bu yeteneğin iç geliştirme örnek dizinidir.

`scripts/generate.py` içindeki `build_*`'ı uyarlayın, ardından `render_html(K.assemble_data(spec), out)`:
```python
from pathlib import Path
out = Path.cwd() / "reaction-<kisa-aciklama>.html"   # kullanıcının geçerli dizinine düşer
render_html(K.assemble_data(spec), out)
```
**Örnekler (kopyalayıp uyarlayın)**:
- `build_combustion_ch4` (metan yanması · morph · alev · enerji) — yüksek seviye `species + atom_map` şablonu;
- `build_redox_na_cl2` (sodyum + klor · redoks · elektron transferi) — `electrons` katmanı;
- `build_esterification` (esterleşme · mechanism · katalizör · geçiş hali) — düşük seviye `atoms + fragments + anahtar kareler` şablonu.

`generate.py` kayıtlı reaksiyonları doğrudan çıkarabilir; **yol yoksa cwd'ye varsayılan**:
```bash
python3 <yetenek-dizini>/scripts/generate.py combustion_ch4 ./reaction.html
python3 <yetenek-dizini>/scripts/generate.py list
```

### Adım 4: Kendi kendini kontrol (doğruluk planına karşılık)
- sympy dengeleme katsayıları == gösterilen denklem katsayıları == her molekül örneğinin sayısı (`assemble_data` içinde zaten `assert` var).
- Atom haritası reaktan ↔ ürün atomları arasında **bijeksiyon**dur, elementler eşleşir; bağ uç noktaları hepsi mevcuttur (kernel doğrulamıştır).
- Atom korunumu sayacı reaksiyon öncesi ve sonrası değişmez (katalizör sayılmaz).
- Yerel bir statik sunucu başlatın (**çıktı dosyasının dizinini** servis eden) ve önizleme kontrolü yapın: konsol hatası yok, KaTeX denklemi doğru render, kaydırıcıyı sürüklerken bağ kırılma/oluşma vurgulaması adım adım açıklamayla eşleşir.

> ⚠️ **Açtığınız portları/hizmetleri mutlaka kapatın**: Önizleme kontrolünden hemen sonra yerel sunucuyu durdurun, **asla port işgal eden süreçler bırakmayın**.
> Önizleme aracıyla açılan: kontrol ettikten hemen sonra `preview_stop` çağırın. Teslimattan önce portun serbest bırakıldığını doğrulayın, ardından kullanıcıya sonucu söyleyin.

### Adım 5: Teslimat
Bitmiş ürün **kullanıcının geçerli çalışma dizinine (cwd)** yazılır, `reaction-<kisa-aciklama>.html` gibi adlandırılır, kullanıcıya yolu söyleyin, tarayıcıda doğrudan açılabilir.
Teslimattan önce doğrulayın: **(1)** bitmiş ürün cwd'de, yetenek dizininde değil; **(2)** bu önizlemeden açık kalan yerel hizmet/port yok.

## Çift motorlar ve otomatik seçim
Şablon bir birleşik renderlayıcıya ve iki kare başına konum çözücüye sahiptir (bağ farkı çizimi / etiketler / katmanlar / UI paylaşılır):
- **morph** (atom morfı): her atom reaktan durumundan ürün durumuna interpolasyon yapar, doğal olarak **atom korunumu/yeniden birleşmeyi** gösterir, herhangi bir reaksiyon için uygundur.
- **mechanism** (mekanizma anahtar kareleri): atomlar rijit fragmentlere aittir, K0/K1/K2 anahtar karelerine göre bütün olarak hareket eder, gruplar deforme olmaz, **katalizör / geçiş hali / ayrılan grup** organik mekanizmaları için uygundur.

`assemble_data` `meta.engine`'e göre seçer (`auto`/`morph`/`mechanism`): `auto` iken kategori `organic` veya `fragments` varlığı mechanism seçer, aksi halde morph. Katmanların hepsi veri anahtarlarıdır (şemaya bakın):
`flame` (yanma/güçlü ekzotermik), `catalyst` (katalizör protonları + anahtar), `transitionGlow` (geçiş hali enerji parıltısı),
`electrons` (redoks elektron transferi), `energy` (enerji–reaksiyon ilerleme eğrisi), atom korunumu sayacı (varsayılan açık).

**Renk şeması**: genel olarak parlak (ders kitabı top-ve-çubuk stili: atomlar koyu kontur + yumuşak gölge + beyaz panel ile), her reaksiyon vurgulu rengi ayırt etmek için `meta.accent` kullanır (yanma amber, esterleşme indigo, sodyum-klor violet…).

## Genişletme
- **Reaksiyonlar ekle**: `generate.py`'de bir `build_*` ekleyin (yüksek seviye `species+atom_map` veya düşük seviye `atoms+fragments`), `REGISTRY`'ye kaydedin.
- **Molekül/iyonlar ekle**: `lib/molecules.py`'deki `_LIBRARY_BUILDERS`'a bir giriş ekleyin (VSEPR geometrisi + görüntü meta verisi + iç bağlar).
- **Katmanlar ekle**: `template/reaction.html`'de bir `data` alanı tarafından yönlendirilen isteğe bağlı bir modül ekleyin.

## Dizin
- `template/reaction.html` — veri odaklı şablon (birleşik renderlayıcı + çift motor + veri adası `__REACTION_DATA__`)
- `lib/molecules.py` — VSEPR ideal molekül geometri kütüphanesi (element tablosu/renkler/yarıçaplar içerir)
- `lib/reaction_kernel.py` — sympy dengeleme + korunum/harita doğrulama + bağ farkı + sahne montajı + isteğe bağlı RDKit probu
- `scripts/generate.py` — şablon enjeksiyonu + örnek build_* (REGISTRY ve CLI içerir)
- `references/problem-schema.md` — reaksiyon spec ve data için veri formatı
- `references/conventions.md` — modelleme kuralları, motor seçimi, katmanlar, dengeleme ve kendi kendini kontrol
