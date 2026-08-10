# ClauraOffice — Sistem Dokümantasyonu

| | |
|---|---|
| **Sürüm** | 2.1 |
| **Tarih** | 10 Ağustos 2026 |
| **Tür** | Teknik · Operasyon · API referansı |

---

## İçindekiler

1. [Genel bakış](#1-genel-bakış)
2. [Mimari](#2-mimari)
3. [Kurulum](#3-kurulum)
4. [Ortam değişkenleri](#4-ortam-değişkenleri)
5. [Kimlik doğrulama ve roller](#5-kimlik-doğrulama-ve-roller)
6. [Panel kullanımı](#6-panel-kullanımı)
7. [Haber skorlama ve taslak üretimi](#7-haber-skorlama-ve-taslak-üretimi)
8. [Kaynak / GSM çıkarımı](#8-kaynak--gsm-çıkarımı)
9. [WhatsApp bildirim formatı](#9-whatsapp-bildirim-formatı)
10. [API referansı](#10-api-referansı)
11. [Veri dosyaları](#11-veri-dosyaları)
12. [Senaryolar](#12-senaryolar)
13. [Sorun giderme](#13-sorun-giderme)
14. [Güvenlik](#14-güvenlik)
15. [Değişiklik geçmişi](#15-değişiklik-geçmişi)

---

## 1. Genel bakış

ClauraOffice; IMAP üzerinden gelen e-postaları izleyen, haber değerini puanlayan, **yayımlanabilir taslak metin** üreten ve eşik üstü içerikleri WhatsApp’a **kaynak bilgisiyle** (ad, e-posta, GSM) ileten bir operasyon platformudur.

### Temel yetenekler

- Çoklu IMAP hesabı
- Çoklu WhatsApp hattı (Baileys)
- OpenRouter LLM analizi + kurallı fallback
- Manşet, özet, taslak haber, editör notu
- Gönderen GSM / telefon çıkarımı
- Rol tabanlı panel (Admin / Editör)
- Dosya tabanlı kalıcılık (`data/`)

### Adresler

| URL | Açıklama |
|-----|----------|
| `/giris` | Panel girişi |
| `/panel` | Özet |
| `/status` | Sağlık durumu |
| Varsayılan port | `5600` |

---

## 2. Mimari

```text
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  IMAP Mail  │────▶│  Analiz      │────▶│  Panel + JSON   │
│  hesapları  │     │  (LLM/KW)    │     │  haberler.json  │
└─────────────┘     └──────┬───────┘     └─────────────────┘
                           │ skor ≥ eşik
                           ▼
                    ┌──────────────┐
                    │  WhatsApp    │
                    │  (Baileys)   │
                    └──────────────┘
```

| Katman | Bileşen |
|--------|---------|
| Sunum | EJS panel (`src/views`) |
| HTTP | Express (`src/app.js`, `panelRoutes.js`) |
| Auth | `middleware/auth.js`, `services/authService.js` |
| Mail | `services/mailChecker.js` |
| WhatsApp | `services/baileysManager.js` |
| Analiz | `utils/haberDegeri.js` |
| Kaynak | `utils/gonderenBilgi.js` |
| Ayarlar | `services/settingsStore.js` |

### Uçtan uca akış

1. Kullanıcılar `panel-users.json` üzerinden yüklenir.
2. Ayarlar `settings.json` okunur.
3. Aktif WhatsApp bağlantıları başlatılır.
4. Mail kontrol döngüsü çalışır (varsayılan 30 sn).
5. Son 5 dakikadaki okunmamış mailler alınır.
6. Gönderen bilgisi (ad, e-posta, GSM) çıkarılır.
7. Skor, manşet, özet, taslak üretilir.
8. Skor ≥ `yuksekHaberSkoru` ise WhatsApp bildirimi gider.
9. Tüm kayıtlar panelde saklanır.

---

## 3. Kurulum

### Gereksinimler

- Node.js **18+**
- npm
- IMAP destekli e-posta (Gmail için App Password)
- WhatsApp kurulu telefon
- Opsiyonel: OpenRouter API anahtarı

### Adımlar

```bash
git clone https://github.com/KULLANICI/ClauraOffice.git
cd ClauraOffice
npm install
cp .env.example .env
# PANEL_ADMIN_* ve SESSION_SECRET zorunlu önerilir
npm start
```

Geliştirme: `npm run dev`

İlk giriş: http://localhost:5600/giris

Admin olarak sırayla: WhatsApp → Mail → (opsiyonel) OpenRouter → (opsiyonel) Editör hesapları.

---

## 4. Ortam değişkenleri

### Güvenlik (zorunlu üretim)

```env
PANEL_ADMIN_USERNAME=admin
PANEL_ADMIN_PASSWORD=guclu_sifre_min_12_karakter
SESSION_SECRET=rastgele_en_az_32_karakter_uzunlugunda_gizli_anahtar
```

### Sistem

```env
PORT=5600
CHECK_INTERVAL=30000
MIN_HABER_SKORU=5
YUKSEK_HABER_SKORU=8
```

### Mail / WhatsApp seed (opsiyonel ilk kurulum)

```env
SMTP_HOST=imap.gmail.com
SMTP_PORT=993
EMAIL_USER=mail@gmail.com
EMAIL_PASS=app_password
WHATSAPP_NUMBER=905555555555
```

### OpenRouter (opsiyonel)

```env
OPENROUTER_API_KEY=sk-or-...
OPENROUTER_MODEL=google/gemma-2-9b-it:free
```

| Değişken | Açıklama |
|----------|---------|
| `YUKSEK_HABER_SKORU` | WhatsApp bildirim eşiği (0–10) |
| `MIN_HABER_SKORU` | Panel sınıflandırma / filtre referansı |
| `SESSION_SECRET` | Oturum HMAC anahtarı (min. 32) |
| `PANEL_ADMIN_PASSWORD` | Yoksa konsola geçici şifre yazılır |

---

## 5. Kimlik doğrulama ve roller

### Roller

| Yetki | Admin | Editör |
|-------|:-----:|:------:|
| Özet, Haberler | ✓ | ✓ |
| Mail, WhatsApp, OpenRouter, Ayarlar | ✓ | |
| Panel hesapları | ✓ | |
| Ayar / CRUD API | ✓ | |
| `GET /api/haberler` | ✓ | ✓ |

### Koruma mekanizmaları

| Mekanizma | Detay |
|-----------|-------|
| Şifre | scrypt (N=16384, r=8, p=1), timing-safe |
| Oturum | HttpOnly, SameSite=Strict, HMAC-SHA256, 12 saat |
| CSRF | `X-CSRF-Token` (POST/PUT/DELETE) |
| Brute-force | 8 deneme → 15 dk IP kilidi |
| Son admin | Silinemez / devre dışı bırakılamaz |

Kullanıcılar: `data/panel-users.json` (chmod 600)

---

## 6. Panel kullanımı

### Sayfalar

| Sayfa | Yetki | Amaç |
|-------|-------|------|
| `/giris` | Açık | Kimlik doğrulama |
| `/panel` | Oturum | Özet istatistik + son içerikler |
| `/panel/haberler` | Oturum | Filtreli haber listesi + detay |
| `/panel/mail` | Admin | IMAP hesap CRUD |
| `/panel/whatsapp` | Admin | QR / pairing, hat yönetimi |
| `/panel/ai` | Admin | OpenRouter + test |
| `/panel/ayarlar` | Admin | Aralık, eşik, kategoriler |
| `/panel/hesaplar` | Admin | Admin / editör hesapları |

### Haber detay paneli

Kart tıklanınca sağ panelde:

- Skor ve değerlendirme
- Editör özeti
- **Taslak haber** (+ kopyala)
- **Kaynak:** ad, e-posta, **GSM** (+ kopyala)
- Kategori, hesap, WhatsApp durumu, tarih

Rozetler:

| Skor | Etiket |
|------|--------|
| ≥ eşik (8) | Haber değeri yüksek |
| ≥ 5 | Orta değer |
| < 5 | Düşük değer |

---

## 7. Haber skorlama ve taslak üretimi

### LLM (OpenRouter)

Üretilen alanlar:

| Alan | Açıklama |
|------|----------|
| `skor` | 0–10 |
| `kategori` | Ön tanımlı listeden |
| `manset` | Kısa haber başlığı |
| `ozet` | 1–2 cümle editör özeti |
| `taslakHaber` | 3–5 cümle yayımlanabilir taslak |
| `degerNotu` | Skor gerekçesi |

### Fallback (anahtar kelime)

LLM yoksa: güncellik ifadeleri, kategori sözlüğü, haber tipi kelimeleri, link sayısı; ardından kural tabanlı manşet / özet / taslak.

### Gönderim kuralı

```text
skor ≥ yuksekHaberSkoru  →  WhatsApp + panel
skor <  yuksekHaberSkoru  →  yalnızca panel
```

---

## 8. Kaynak / GSM çıkarımı

`utils/gonderenBilgi.js` mail `From` başlığı ve gövde/imza metninden:

- **Ad** (`Ad Soyad <mail@x.com>`)
- **E-posta**
- **GSM** (TR cep öncelikli: `05xx`, `+90 5xx`)
- Sabit hatlar ve ek numaralar (`tumTelefonlar`)

Desteklenen örnek kalıplar:

```text
GSM: 0532 111 22 33
Tel: +90 212 555 66 77
WhatsApp: 5321112233
```

Normalize örnek: `0532 111 22 33` → `+90 532 111 22 33`

---

## 9. WhatsApp bildirim formatı

Yalnızca yüksek değerli mailler için:

```text
*CLAURAOFFICE · HABER BİLDİRİMİ*

*<manşet>*

*Değer:* Yüksek haber değeri  ·  *Skor:* 9/10
*Kategori:* akıllı-telefon
*Hesap:* Basın Kutusu
*Tarih:* 10.08.2026 12:00:00

────────────
*TASLAK HABER*

<yayımlanabilir taslak>

────────────
*KAYNAK / GÖNDEREN*

*Ad:* Ayşe Yılmaz
*E-posta:* press@ornekfirma.com
*GSM:* +90 532 111 22 33

_Editör notu: …_
```

GSM bulunamazsa satırda açıkça belirtilir.

---

## 10. API referansı

### Yetki etiketleri

| Etiket | Anlam |
|--------|-------|
| Açık | Giriş gerekmez |
| Oturum | Giriş yapmış kullanıcı |
| Admin | `role === 'admin'` |
| CSRF | `X-CSRF-Token` zorunlu |

### Kimlik

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/giris` | Açık |
| POST | `/api/auth/giris` | Açık |
| GET | `/api/auth/me` | Oturum |
| POST | `/api/auth/cikis` | Oturum + CSRF |

```json
POST /api/auth/giris
{ "username": "admin", "password": "…" }
```

### Sayfalar

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/panel` | Oturum |
| GET | `/panel/haberler` | Oturum |
| GET | `/panel/mail` | Admin |
| GET | `/panel/whatsapp` | Admin |
| GET | `/panel/ai` | Admin |
| GET | `/panel/ayarlar` | Admin |
| GET | `/panel/hesaplar` | Admin |

### Haberler

```http
GET /api/haberler?minSkor=8
```

Örnek kayıt alanları: `manset`, `ozet`, `taslakHaber`, `gonderenAd`, `gonderenEposta`, `gonderenGsm`, `skor`, `gonderildi`, …

### Ayarlar / Mail / WhatsApp / Kullanıcılar

| Grup | Endpoint’ler | Yetki |
|------|--------------|-------|
| Ayarlar | `GET/PUT /api/settings`, `PUT /api/settings/openrouter` | Admin (+ CSRF yazma) |
| Mail | `POST/PUT/DELETE /api/mail-accounts` | Admin + CSRF |
| WhatsApp | `GET /api/whatsapp/status`, CRUD, `/reconnect` | Admin (+ CSRF yazma) |
| Kullanıcılar | CRUD `/api/panel-users` | Admin + CSRF |
| AI test | `POST /api/ai/test` | Admin + CSRF |
| Manuel mesaj | `POST /mesaj-gonder` | Admin + CSRF |
| Durum | `GET /status` | Açık |

---

## 11. Veri dosyaları

| Dosya | İçerik | Git |
|-------|--------|-----|
| `data/settings.json` | Ayarlar, mail, WA, OpenRouter | Ignore |
| `data/haberler.json` | Son ≤200 haber kaydı | Ignore |
| `data/panel-users.json` | Kullanıcı hash’leri | Ignore |
| `whatsapp_auth/` | WA oturumları | Ignore |
| `.env` | Gizli anahtarlar | Ignore |

### Haber kaydı (özet şema)

```json
{
  "id": "…",
  "tarih": "2026-08-10T09:00:00.000Z",
  "hesapAdi": "Basın Kutusu",
  "konu": "…",
  "manset": "…",
  "ozet": "…",
  "taslakHaber": "…",
  "degerNotu": "…",
  "skor": 9,
  "kategori": "akıllı-telefon",
  "degerlendirme": "Yüksek haber değeri",
  "gonderenAd": "Ayşe Yılmaz",
  "gonderenEposta": "press@ornekfirma.com",
  "gonderenGsm": "+90 532 111 22 33",
  "tumTelefonlar": ["+90 532 111 22 33"],
  "gonderildi": true
}
```

---

## 12. Senaryolar

### İlk kurulum

1. `npm install && npm start`
2. Admin ile `/giris`
3. WhatsApp QR eşle
4. Mail hesabı ekle ve WA hattına bağla
5. İsteğe bağlı OpenRouter

### Editör ekibi

1. `/panel/hesaplar` → rol `editor`
2. Editör yalnızca Özet + Haberler görür
3. Taslak ve GSM detayda okunur; ayar değiştiremez

### Sadece kritik bildirimler

1. Ayarlar → WhatsApp bildirim eşiği `8` veya `9`
2. Düşük skorlar panelde kalır, WA’ya gitmez

### Üretim sertleştirme

1. Güçlü `SESSION_SECRET` + admin şifresi
2. `NODE_ENV=production`
3. HTTPS reverse proxy
4. İç ağ / VPN

---

## 13. Sorun giderme

| Belirti | Çözüm |
|---------|--------|
| Port 5600 dolu | `lsof -ti:5600 \| xargs kill -9` |
| Sürekli `/giris` | Normal — giriş yapın |
| Brute-force kilidi | 15 dk bekleyin |
| WhatsApp bağlanmıyor | Yeniden bağlan / QR / `whatsapp_auth` temizle |
| Mail gelmiyor | IMAP, App Password, host/port |
| WA’ya gitmiyor | Skor ≥ eşik mi? Hat bağlı mı? Hedef numara? |
| GSM yok | İmzada numara yoksa “bulunamadı” yazılır |
| CSRF 403 | Çıkış → tekrar giriş |

---

## 14. Güvenlik

### Hassas varlıklar

- Mail şifreleri, OpenRouter anahtarı → `settings.json`
- Panel şifre hash’leri → `panel-users.json`
- WhatsApp oturumları → `whatsapp_auth/`
- Haber içerikleri → `haberler.json`

### Öneriler

1. Asla `data/` ve `.env` commit etmeyin
2. Üretimde HTTPS + güçlü sırlar
3. Editör hesaplarını sınırlı tutun
4. Admin şifresini periyodik değiştirin
5. Düzenli yedek alın

---

## 15. Değişiklik geçmişi

### v2.1 — 10 Ağustos 2026

- Manşet, taslak haber, editör notu üretimi
- Gönderen ad / e-posta / **GSM** çıkarımı
- Güçlendirilmiş WhatsApp bildirim şablonu
- Panel detayında taslak + GSM kopyalama
- GitHub README ve dokümantasyon yenileme

### v2.0 — 10 Ağustos 2026

- Admin / Editör kimlik doğrulama
- CSRF, scrypt, oturum güvenliği
- Panel hesap yönetimi

### v1.1 — 10 Ağustos 2026

- `yuksekHaberSkoru` eşik modeli
- Kart listesi + offcanvas detay

### v1.0 — 10 Ağustos 2026

- İlk sürüm: IMAP tarama, skor, WhatsApp, OpenRouter

---

*ClauraOffice — gelen kutusu → skor → taslak → WhatsApp*
