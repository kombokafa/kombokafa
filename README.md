## Otel Satınalma Agent Tasarımı (Çoklu Otel + Tedarikçi Entegrasyonu)

Bu doküman, bir **otel satınalma agentı** için ürün kapsamını, veri modelini, forecast yaklaşımını ve ek iş akışlarını özetler.

---

### 1) Hedef: Agent ne yapacak?

Agent aşağıdaki sorulara yanıt verir:

- **Hangi ürünü** almalıyım?
- **Ne zaman** almalıyım?
- **Ne kadar** almalıyım?

Ayrıca şu analizleri üretir:

- Satınalma trend analizi
- Tüketim oran analizi (kişi başı / oda başı / geceleme başı)
- Sapma analizi (forecast vs gerçekleşen)
- Kritik stok riski ve stok-out uyarıları

---

### 2) Çoklu otel yönetimi (yeni gereksinim)

Aynı satınalmacı birden fazla otelden sorumlu olabilir. Bu nedenle:

- Her otel için ayrı **otel profili** tanımlanır.
- Tüm oteller tek panelde görüntülenir.
- Kullanıcı üst menüden otel seçerek tekil yönetim yapar.
- İsteğe bağlı olarak “toplu görünüm” ile zincir/genel konsolide rapor alınır.

**Otel profili örnek alanları:**

- `hotel_id`, `hotel_name`
- Konsept (BB/HB/AI)
- Sezon başlangıç/bitiş ayı
- Depo çalışma kuralları
- Onay hiyerarşisi
- Varsayılan tedarikçi listesi

---

### 3) Kullanılacak temel girdiler

#### A) Talep (misafir) verisi

- Günlük konaklayan kişi sayısı
- Aylık/yıllık forecast misafir sayısı
- Oda doluluk oranı, geceleme sayısı (varsa)
- Segment kırılımı (grup/bireysel, yerli/yabancı)

#### B) Tüketim / çıkış verisi

- Depodan alınan günlük ürünler
- Ürün bazında miktar
- Birim tipi (kg, lt, adet, koli)
- Departman kırılımı (mutfak, housekeeping, minibar)

#### C) Satınalma / stok verisi

- Anlık stok, minimum stok, güvenlik stoğu
- Tedarikçi teslim süresi (lead time)
- Satınalma fiyat geçmişi
- Sipariş frekansı ve minimum sipariş miktarı

#### D) Ürün sınıflandırma (yeni gereksinim)

Ürünler aşağıdaki yapıda gruplanabilmelidir:

- Ana grup (Gıda, İçecek, Temizlik, Sarf vb.)
- Alt grup
- Marka/kalite segmenti
- Kritiklik seviyesi (A/B/C)

Bu sınıflandırma forecast, onay ve rapor ekranlarında filtre olarak kullanılacaktır.

---

### 4) Forecast mantığı

Agent, kişi sayısı ve geçmiş tüketim ilişkisini kullanır:

1. Ürün başına **kişi başı tüketim katsayısı** hesaplanır.
2. Bu katsayı, sezon/ay/hafta günü etkileriyle normalize edilir.
3. Gelecek dönemin misafir forecast’i ile çarpılarak dönemsel ihtiyaç bulunur.
4. Stok, teslim süresi ve güvenlik stoğu eklenerek satınalma önerisi üretilir.

Basit formül:

`Önerilen Alım = Dönem İhtiyacı + Güvenlik Stoğu - Mevcut Kullanılabilir Stok`

---

### 5) Çıktı periyotları

Agent aynı hesaplama motoru ile farklı periyotlarda öneri verebilir:

- **Aylık satınalma öngörüsü**
- **3 aylık plan**
- **6 aylık plan**
- **Yıllık plan**

Yıllık planda sezon ayları takvim üzerinden seçilebilir:

- Sezon başlangıç/bitiş ayı
- Sezonun kaç ay süreceği
- Yüksek/düşük sezon profili

---

### 6) Tedarikçi entegrasyonu ve sipariş akışı (yeni gereksinim)

Hedef yapı: otel app + tedarikçi app birbirine bağlı çalışır.

#### 6.1 Tek tuşla talep/sipariş

Otel analiz sonucu ürün ihtiyacı oluştuğunda kullanıcı:

- İlgili ürünleri seçer
- **Tek tuşla** ilgili tedarikçilere talep/sipariş gönderir

#### 6.2 Tedarikçi tipine göre süreç

1. **Yıllık anlaşmalı tedarikçiler**
   - Doğrudan sipariş akışına girer
   - Teslim tarihi ve miktar onayı yönetilir

2. **Sor-Al (teklif) modeli tedarikçiler**
   - RFQ/talep gönderilir
   - Tedarikçi fiyat teklifini iletir
   - Sistem teklifleri ürün, vade, teslim süresi, kalite puanı ile kıyaslar
   - Satınalma onaylarsa teklif **siparişe dönüşür**

#### 6.3 Önerilen durum akışı (status)

- `draft` → `rfq_sent` → `quote_received` → `evaluation` → `approved` → `po_created` → `delivered` → `closed`

---

### 7) Depo stok takibi (yeni gereksinim)

Depo stokları için günlük kapanış mantığı:

- Her gün sonunda sistem stok kapanışı bekler
- Kapanış iki şekilde yapılabilir:
  - **Manuel güncelleme** (depo sorumlusu giriş yapar)
  - **Otomatik güncelleme** (POS/ERP/WMS entegrasyonu)
- Kapanış tamamlanmadan ertesi gün önerileri “ön izleme” statüsünde kalabilir

Ek öneri:

- Sayım sapmaları için tolerans limiti
- Sapma aşılırsa onay veya açıklama zorunluluğu

---

### 8) Raporlama (yeni gereksinim)

Sistemden alınabilecek temel raporlar:

- Otel bazlı satınalma özeti (ay/çeyrek/yıl)
- Ürün grubu bazlı tüketim ve maliyet
- Tedarikçi performans raporu (fiyat, termin, tam teslim)
- Forecast doğruluk raporu (MAPE/WAPE)
- Stok devir hızı ve kritik stok raporu
- Tekliften siparişe dönüşüm oranı

Raporlar filtrelenebilir olmalı:

- Otel
- Tarih aralığı
- Ürün grubu
- Tedarikçi
- Departman

---

### 9) Demo talep butonu (yeni gereksinim)

Ürün ya da modül kartlarında **“Demo Talep Et”** butonu bulunabilir.

Beklenen davranış:

- Kullanıcı butona basar
- İlgili modül/ürün bilgisi otomatik eklenir
- İletişim bilgisi alınır
- Talep CRM/satış kuyruğuna düşer
- Satış ekibine bildirim gider

---

### 10) Önerilen ekranlar

1. **Multi-Hotel Dashboard**
   - Otel seçici (tekil/toplu)
   - Toplam bütçe, risk, kritik ürünler

2. **Forecast ve Planlama**
   - Aylık/3A/6A/Yıllık plan
   - Sezon takvimi seçici

3. **Tedarikçi ve RFQ Yönetimi**
   - Teklif toplama, karşılaştırma, onay
   - Yıllık anlaşmalı tedarikçi sipariş paneli

4. **Depo Kapanış Ekranı**
   - Gün sonu manuel/otomatik kapanış
   - Sayım sapma kontrolü

5. **Raporlama Merkezi**
   - Şablon raporlar + dışa aktarma (Excel/PDF)

---

### 11) MVP (ilk sürüm) önerisi

İlk fazda:

- 2-3 otel profili
- 30-50 kritik ürün
- Aylık + 3 aylık forecast
- Günlük stok kapanış (manuel)
- RFQ akışı (sor-al) + temel teklif kıyaslama
- Otel/ürün grubu bazlı raporlar

İkinci faz:

- 6 ay/yıl planı
- Otomatik stok güncelleme entegrasyonları
- Yıllık anlaşmalı tedarikçi otomatik sipariş
- Gelişmiş senaryo simülasyonu

---

### 12) Örnek veri sözlüğü

- `hotel_id`
- `date`
- `product_id`
- `product_name`
- `product_group`
- `sub_group`
- `unit`
- `daily_issue_qty`
- `guest_count`
- `forecast_guest_count`
- `current_stock_qty`
- `safety_stock_qty`
- `lead_time_days`
- `supplier_id`
- `supplier_type` (`annual_contract` | `rfq`)
- `unit_price`
- `season_flag`
- `department`
- `closing_status`

Bu alanlar agent’ın çoklu otel, çoklu tedarikçi ve günlük depo kapanışı senaryolarında doğru karar üretmesi için temel omurgayı oluşturur.
