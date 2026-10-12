---
name: edu-chem-video
description: >-
  Bir kimya problemi için açıklayıcı / çözüm videosu istendiğinde kullanın (redoks dengeleme ve çift çizgi köprüsü, madde miktarı hesaplamaları, kimyasal denge ICE tablosu, denge sabiti, dönüşüm oranı, reaksiyon hızı, iyonik reaksiyonlar, elektrokimya, titrasyon, endüstriyel süreçler), problem ekran görüntüsü veya metinden, Türkçe (veya İngilizce) seslendirme (edge-tts veya GLM-TTS), çift dilli tr+en altyazılar ve kimyasal denklem etrafında kurulmuş tahta tarzı canvas animasyonu ile MP4'e render edilmiş.
  Tetikleyici kelimeler: kimya açıklayıcı video, redoks, elektron korunumu, çift çizgi köprüsü, madde miktarı, kimyasal denge, ICE tablosu, titrasyon; chemistry explainer video.
---

# Kimya Problemi Açıklayıcı Videosu (edu-chem-video)

Bir kimya problemini 16:9, 1920×1080 MP4'e dönüştürür: Türkçe/İngilizce seslendirme + çift dilli altyazılar + **yeşil tahta** tarzı animasyon. Boru hattı diğer video yetenekleriyle aynıdır; farklar çözüm yöntemi ve resimdir:
- Çözüm: ana hat "**korunum + madde miktarının merkezde olması**"dır: denklemi doğru yazın ve dengeleyin (elektron korunumu → atom korunumu → yük korunumu) → ilişkileri bulun (katsayı oranları, korunum, ilişkiler) → n merkezli hesaplayın / ICE tablosu → korunumu yeniden kontrol edin.
- Resim: koyu yeşil tahta + tebeşir çizgileri; **üstte sabitlenmiş denklem şeridi**, solda demo, sağda taş levha. Yükseltgenme sayıları sayı doğrusunda yükselir/alçalır, elektronlar sayılar eşleşene kadar gruplanır, katsayılar elektron satırından denkleme uçar, atom sayımları ✓ alır, çift çizgi köprüsü elektron akışını gösterir, molekül kutuları katsayı oranında reaksiyona girer, ICE tablosu satır satır dolar. Matematikten (bej defter) ve fizikten (koyu mavi blueprint) görsel olarak ayırt edilir.

**Temel ilke: denklem problemi açıklar ve korunum ekranda "sayılır".** Her anlatım satırının o kimyasal akıl yürütmeyi canlandıran bir eylemi vardır; tüm oranlar ve miktarlar dengelenmiş denklemden gelir; her animasyon `S.at(k, f)` ile zamanlanır.

Yetenek tüm kodu içerir (bakır + seyreltik nitrik asit örneğiyle şablon, equation/valence/bridge/eFlow/tally/molMap/iceTable/molecule/beaker/graph için chem.js prop kütüphanesi, examples/equilibrium, sıfır bağımlılıklı dengeleme doğrulaması için chem_check.py, kimya terimleriyle paylaşılan telaffuz, betikler).

## Uyulması gereken kurallar
Aynı 11 adımlık disiplin, artı:
- Önce kimya yöntemiyle çözün ve çözüm kartı yazın. Denklemler `chem_check.py` ile doğrulanmalıdır.
- `tts` formüller yerine isimler kullanır (`HNO3` → nitrik asit, `SO4^{2-}` → sülfat iyonu, birimler konuşulur).
- Kimyasal doğruluk: yükseltgenme sayıları, katsayılar, köprü uç noktaları ve elektron sayıları, molekül sayıları çözüm kartıyla eşleşir. Renk kuralı: kaybedilen elektronlar / yükseltgenme mavi, kazanılan elektronlar / indirgenme kırmızı.
- Düzen: denklem şeridi y 140~300, demo alanı x 60~900 y 310~860, taş levha `board()` + `bm()` ile.

## 11 Adım
1. Kimya yöntemiyle çözün, çözüm kartı yazın (chemistry-solving referansını okuyun). `chem_check.py balance` ve `check` çalıştırın.
2. Proje oluştur + ortam kontrolü.
3. TTS'yi test edin.
4. Problem görüntüsü + kutuları hazırlayın.
5. episode.json + script.json yazın (sahneler çözüm akışını takip eder: intro → denklem/yükseltgenme sayıları → dengeleme → (çift çizgi köprüsü) → hesaplama/ICE → (yeniden kontrol, yaygın hatalar) → outro).
6. storyboard.md yazın (üstte çözüm kartı + şekil listesi; işaret et/hareket et/bırak; kimya eylem tablosundan seçin).
7. Telaffuz ve script kontrolü 0'a.
8. anim.js yazın (denklem spec ve katsayılarla kimyasal sabitler alanı, eqDraw, her sahne için SC.<id>).
9. Ücretsiz önizleme: motion + stills. Ekran görüntülerinde kimyasal doğruluğu kontrol edin.
10. Gerçek TTS + ASR + yeniden kontrol.
11. Render edin ve teslim edin.

## Yaygın hatalar
Dengesiz denklemle hesaplamaya başlamayın, dengeleme için yalnızca "gözlem yöntemi" demeyin (yükseltgenme sayıları → elektron korunumu → atom korunumu gösterin), tts'ye formül yazmayın, çift çizgi köprüsünü farklı elementler arasında bağlamayın, "indirgenen nitrik asit"i "reaksiyona giren nitrik asit" ile karıştırmayın, K için her zaman konsantrasyonları (miktarları değil) kullanın, her zaman ekran görüntülerini kontrol edin vb.

## Referans dosyaları
chemistry-solving.md, visual-design.md, animation.md, script-writing.md, pronunciation.md, glm-tts-setup.md, troubleshooting.md.
