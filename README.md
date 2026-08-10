# ClauraOffice — Sistem Dokümantasyonu

| | |
|---|---|
| **Sürüm** | 2.0 |
| **Tarih** | 10 Ağustos 2026 |
| **Dosya** | `docs/ClauraOffice-Dokumantasyon.md` |

---

## İçindekiler

1. [Sistem Nedir?](#1-sistem-nedir)
2. [Sistem Nasıl Çalışır?](#2-sistem-nasıl-çalışır)
3. [Kurulum](#3-kurulum)
4. [Ortam Değişkenleri (.env)](#4-ortam-değişkenleri-env)
5. [Dosya Yapısı](#5-dosya-yapısı)
6. [Kimlik Doğrulama ve Roller](#6-kimlik-doğrulama-ve-roller)
7. [Panel Genel Yapısı](#7-panel-genel-yapısı)
8. [Panel Kullanımı — Giriş](#8-panel-kullanımı--giriş)
9. [Panel Kullanımı — Özet Sayfası](#9-panel-kullanımı--özet-sayfası)
10. [Panel Kullanımı — Mail Sayfası](#10-panel-kullanımı--mail-sayfası)
11. [Panel Kullanımı — WhatsApp Sayfası](#11-panel-kullanımı--whatsapp-sayfası)
12. [Panel Kullanımı — OpenRouter Sayfası](#12-panel-kullanımı--openrouter-sayfası)
13. [Panel Kullanımı — Haberler Sayfası](#13-panel-kullanımı--haberler-sayfası)
14. [Panel Kullanımı — Ayarlar Sayfası](#14-panel-kullanımı--ayarlar-sayfası)
15. [Panel Kullanımı — Panel Hesapları (Admin)](#15-panel-kullanımı--panel-hesapları-admin)
16. [Haber Skorlama Mantığı](#16-haber-skorlama-mantığı)
17. [WhatsApp'a Giden Mesaj Formatı](#17-whatsappa-giden-mesaj-formatı)
18. [Tüm Endpoint Listesi](#18-tüm-endpoint-listesi)
19. [Ayar Dosyası Yapısı (data/settings.json)](#19-ayar-dosyası-yapısı-datasettingsjson)
20. [Panel Kullanıcı Dosyası (data/panel-users.json)](#20-panel-kullanıcı-dosyası-datapanel-usersjson)
21. [Örnek Kullanım Senaryoları](#21-örnek-kullanım-senaryoları)
22. [Sık Karşılaşılan Sorunlar](#22-sık-karşılaşılan-sorunlar)
23. [Güvenlik](#23-güvenlik)
24. [Hızlı Başvuru](#24-hızlı-başvuru)
25. [Değişiklik Geçmişi](#değişiklik-geçmişi)

---

## 1. Sistem Nedir?

ClauraOffice, gelen e-postaları otomatik izleyen, haber olma ihtimali yüksek mailleri puanlayan ve belirli bir skorun üzerindeki içerikleri WhatsApp üzerinden ilgili numaraya gönderen bir **izleme ve bildirim platformudur**.

### Temel işlevler

- Birden fazla IMAP mail hesabı tanımlama
- Birden fazla WhatsApp bağlantısı yönetme
- Gelen mailleri periyodik tarama (varsayılan 30 sn)
- OpenRouter ile yapay zeka destekli mail analizi ve özet üretimi
- Haber skoru hesaplama (0–10)
- Yüksek skorlu mailleri WhatsApp bildirim eşiğine göre filtreleme
- Tüm işlenen mailleri panelde kart listesi olarak gösterme
- Rol tabanlı panel erişimi (Admin / Editör)
- Panel üzerinden tüm ayarları yönetme

### Adresler

| | |
|---|---|
| Giriş | http://localhost:5600/giris |
| Panel | http://localhost:5600/panel |
| Port | `5600` (`PORT` ortam değişkeni ile değiştirilebilir) |

---

## 2. Sistem Nasıl Çalışır?

Sistem başladığında şu adımlar otomatik yapılır:

1. Panel kullanıcıları `data/panel-users.json` dosyasından yüklenir. (Dosya yoksa `.env`'den ilk admin hesabı oluşturulur.)
2. Ayarlar `data/settings.json` dosyasından okunur.
3. Aktif WhatsApp bağlantıları başlatılır.
4. Mail kontrol servisi belirli aralıklarla çalışmaya başlar.
5. Her döngüde aktif mail hesapları IMAP üzerinden taranır.
6. Son 5 dakikadaki okunmamış mailler alınır.
7. Her mail için haber skoru ve özet hesaplanır.
8. Skor ≥ `yuksekHaberSkoru` ise ilgili WhatsApp hattına bildirim gider.
9. Tüm işlenen mailler (skor fark etmeksizin) `data/haberler.json`'a kaydedilir.

### Varsayılan değerler

| Ayar | Değer |
|------|-------|
| Kontrol aralığı | 30 saniye |
| WhatsApp bildirim eşiği | 8 (`yuksekHaberSkoru`) |
| Panel min. skor filtresi | 5 (`minHaberSkoru`) |
| Oturum süresi | 12 saat |

### Gönderim kuralı

| Koşul | Sonuç |
|-------|--------|
| Skor ≥ `yuksekHaberSkoru` | WhatsApp bildirimi gider |
| Skor < `yuksekHaberSkoru` | Yalnızca panelde kaydedilir |

---

## 3. Kurulum

### Gereksinimler

- Node.js (v18+ önerilir)
- npm
- IMAP destekli e-posta hesabı
- WhatsApp kurulu bir telefon
- Opsiyonel: OpenRouter API anahtarı

### Kurulum adımları

```bash
cd ClauraOffice
npm install
cp .env.example .env
# .env dosyasını düzenleyin
npm start
```

`.env` içinde en az şu alanları doldurun:

```env
PANEL_ADMIN_USERNAME=admin
PANEL_ADMIN_PASSWORD=guclu_sifre_min_12_karakter
SESSION_SECRET=rastgele_en_az_32_karakter_uzunlugunda_gizli_anahtar
```

1. Tarayıcıdan giriş yapın: http://localhost:5600/giris
2. Admin olarak giriş yaptıktan sonra:
   - WhatsApp bağlantısı kurun → `/panel/whatsapp`
   - Mail hesaplarını ekleyin → `/panel/mail`
   - OpenRouter ayarlarını yapın → `/panel/ai` (opsiyonel)
   - Editör hesapları ekleyin → `/panel/hesaplar` (opsiyonel)

**Geliştirme modu:** `npm run dev` (nodemon ile otomatik yeniden başlatma)

---

## 4. Ortam Değişkenleri (.env)

İlk kurulumda `.env` dosyası kullanılır. Panel devreye girdikten sonra mail, WhatsApp ve OpenRouter ayarları panelden yönetilebilir.

### Mail ve WhatsApp (ilk kurulum)

```env
SMTP_HOST=imap.gmail.com
SMTP_PORT=993
EMAIL_USER=mail@gmail.com
EMAIL_PASS=app_password
WHATSAPP_NUMBER=905555555555
```

### Sistem

```env
PORT=5600
CHECK_INTERVAL=30000
MIN_HABER_SKORU=5
YUKSEK_HABER_SKORU=8
SADECE_YUKSEK_HABER=true
HABER_KATEGORILERI=akilli-telefon,yazilim,donanim,yapay-zeka,otomobil,oyun,guvenlik,5g,giyilebilir,bilim,uzay,spor,politika,ekonomi,teknoloji,saglik
```

### OpenRouter (opsiyonel)

```env
OPENROUTER_API_KEY=sk-or-...
OPENROUTER_MODEL=google/gemma-2-9b-it:free
```

### Panel güvenliği (ZORUNLU üretim ortamında)

```env
PANEL_ADMIN_USERNAME=admin
PANEL_ADMIN_PASSWORD=guclu_sifre_min_12_karakter
SESSION_SECRET=rastgele_en_az_32_karakter_uzunlugunda_gizli_anahtar
```

### Alan açıklamaları

| Değişken | Açıklama |
|----------|----------|
| `SMTP_HOST` / `SMTP_PORT` | IMAP sunucu adresi ve portu. Gmail: `imap.gmail.com` / `993` |
| `EMAIL_USER` / `EMAIL_PASS` | E-posta adresi ve şifresi (Gmail için App Password) |
| `WHATSAPP_NUMBER` | Bildirimlerin gideceği numara. Ülke kodu ile: `905551234567` |
| `PORT` | Sunucu portu. Varsayılan: `5600` |
| `CHECK_INTERVAL` | Mail kontrol aralığı (ms). `30000` = 30 saniye |
| `MIN_HABER_SKORU` | Panel min. skor filtresi referansı. 0–10. Varsayılan: `5` |
| `YUKSEK_HABER_SKORU` | WhatsApp bildirim eşiği. 0–10. Varsayılan: `8` |
| `SADECE_YUKSEK_HABER` | Eski alan; geriye dönük uyumluluk |
| `HABER_KATEGORILERI` | Virgül ile ayrılmış kategori listesi |
| `OPENROUTER_API_KEY` / `OPENROUTER_MODEL` | OpenRouter API anahtarı ve model adı |
| `PANEL_ADMIN_USERNAME` | İlk admin kullanıcı adı. 3–32 karakter, küçük harf, rakam, alt çizgi |
| `PANEL_ADMIN_PASSWORD` | İlk admin şifresi. Min. 12 karakter |
| `SESSION_SECRET` | Oturum imzalama anahtarı. Min. 32 karakter |

---

## 5. Dosya Yapısı

```
ClauraOffice/
├── data/
│   ├── settings.json        # Sistem ayarları
│   ├── haberler.json        # Son işlenen haber kayıtları (max 200)
│   └── panel-users.json     # Panel kullanıcı hesapları (scrypt hash)
├── src/
│   ├── app.js               # Ana uygulama
│   ├── middleware/
│   │   └── auth.js          # Oturum, CSRF, rol kontrolü
│   ├── routes/
│   │   └── panelRoutes.js   # Panel sayfaları ve API
│   ├── services/
│   │   ├── authService.js   # Giriş, oturum, kullanıcı CRUD
│   │   ├── settingsStore.js # Ayar yönetimi
│   │   ├── mailChecker.js   # IMAP mail tarama
│   │   └── baileysManager.js# WhatsApp bağlantı servisi
│   ├── utils/
│   │   └── haberDegeri.js   # Haber skorlama (LLM + anahtar kelime)
│   └── views/               # EJS panel ekranları
├── whatsapp_auth/           # WhatsApp oturum dosyaları
├── docs/
│   └── ClauraOffice-Dokumantasyon.md
├── package.json
├── .env
└── .env.example
```

---

## 6. Kimlik Doğrulama ve Roller

Panel erişimi zorunlu giriş gerektirir. Yetkisiz erişim `/giris` sayfasına yönlendirilir.

### Roller

#### Admin (Yönetici)

Tam yetki. Tüm panel sayfaları ve API işlemleri:

- Özet, Haberler (görüntüleme)
- Mail hesapları (ekle / düzenle / sil)
- WhatsApp bağlantıları (ekle / düzenle / sil / yeniden bağlan)
- OpenRouter ayarları ve test
- Genel sistem ayarları
- Panel kullanıcı yönetimi
- Manuel mesaj gönderimi

#### Editör

Salt okunur erişim. Yalnızca haber takibi:

- Özet sayfası (istatistikler + son mailler)
- Haberler sayfası (kart listesi + detay)
- `GET /api/haberler`
- `GET /api/auth/me`

Diğer sayfalara ve değişiklik API'lerine erişemez.

### Güvenlik özellikleri

| Özellik | Detay |
|---------|-------|
| Şifre saklama | scrypt hash (N=16384, r=8, p=1), timing-safe karşılaştırma |
| Oturum | HttpOnly + SameSite=Strict çerezler, HMAC-SHA256 imzalı |
| Oturum süresi | 12 saat |
| CSRF koruması | Tüm POST / PUT / DELETE → `X-CSRF-Token` gerekli |
| Brute-force | 8 başarısız deneme → 15 dakika IP kilidi |
| Güvenlik başlıkları | X-Frame-Options, X-Content-Type-Options, nosniff |
| Dosya izinleri | `panel-users.json` chmod 600 |
| Son admin koruması | Son admin silinemez, devre dışı bırakılamaz |

### Menü görünürlüğü

| Rol | Menü |
|-----|------|
| **Admin** | Özet · Mail · WhatsApp · OpenRouter · Haberler · Ayarlar · Hesaplar |
| **Editör** | Özet · Haberler |

---

## 7. Panel Genel Yapısı

| Adres | Açıklama |
|-------|----------|
| `GET /` | `/panel`'a yönlendirir (giriş gerekli) |
| `GET /giris` | Giriş sayfası (herkese açık) |
| `GET /panel` | Özet sayfası |
| `GET /dashboard` | `/panel/whatsapp`'a yönlendirir (eski adres) |

Üst barda: görünen ad, rol etiketi (Admin / Editör), çıkış butonu.

---

## 8. Panel Kullanımı — Giriş

| | |
|---|---|
| **Adres** | `GET /giris` · `POST /api/auth/giris` |
| **Yetki** | Herkese açık |

### Giriş adımları

1. http://localhost:5600/giris adresine gidin
2. Kullanıcı adı ve şifrenizi girin
3. Başarılı giriş sonrası `/panel`'e yönlendirilirsiniz

### İlk admin hesabı

- `data/panel-users.json` yoksa otomatik oluşturulur
- `.env`'deki `PANEL_ADMIN_USERNAME` / `PANEL_ADMIN_PASSWORD` kullanılır
- Şifre tanımlanmadıysa konsola geçici şifre yazılır

### Çıkış

Sağ üstteki **Çıkış** butonu veya `POST /api/auth/cikis`

### Başarısız giriş

- "Kullanıcı adı veya şifre hatalı" mesajı
- 8 başarısız denemeden sonra 15 dakika kilit

---

## 9. Panel Kullanımı — Özet Sayfası

| | |
|---|---|
| **Adres** | `GET /panel` |
| **Yetki** | Admin + Editör |

### İstatistik kartları

- Aktif mail hesap sayısı
- Bağlı WhatsApp sayısı
- Son haber kayıt sayısı (son 10)
- WhatsApp bildirim eşiği

### Son mailler (kart listesi)

- Skor kutusu (/10)
- Rozet: **Haber** (yeşil) / **Orta** (sarı) / **Düşük** (gri)
- Konu ve kısa özet, kategori, WA durumu, tarih
- Karta tıklanınca sağ tarafta **offcanvas** detay paneli açılır

---

## 10. Panel Kullanımı — Mail Sayfası

| | |
|---|---|
| **Adres** | `GET /panel/mail` |
| **Yetki** | Yalnızca Admin |

### Form alanları

| Alan | Açıklama |
|------|----------|
| Ad | Hesabın panelde görünecek adı |
| Host | IMAP sunucu (ör. `imap.gmail.com`) |
| Port | IMAP portu (genelde 993) |
| E-posta | Mail adresi |
| Şifre | Düzenlemede boş = korunur |
| WhatsApp | Bildirimlerin gideceği hat |
| Aktif | İşaretli ise hesap taranır |

> Gmail için App Password kullanın. Her mail hesabı bir WhatsApp hattına bağlı olmalıdır.

---

## 11. Panel Kullanımı — WhatsApp Sayfası

| | |
|---|---|
| **Adres** | `GET /panel/whatsapp` |
| **Yetki** | Yalnızca Admin |

### Bağlantı durumları

| Durum | Açıklama |
|-------|----------|
| `baglaniyor` | Bağlantı kuruluyor |
| `qr_bekleniyor` | QR kod bekleniyor |
| `baglantili` | Başarıyla bağlandı |
| `yeniden_baglaniyor` | Kopma sonrası tekrar deneniyor |
| `auth_hatasi` | Oturum hatası, yeniden eşleme gerekir |
| `baslatilmadi` | Henüz başlatılmamış |

### QR ile bağlama

1. Telefonda WhatsApp → Bağlı cihazlar → Cihaz bağla
2. Paneldeki QR kodu okutun

> Oturum bilgileri `whatsapp_auth/` altında saklanır. Sayfa her 5 saniyede durumu yeniler.

---

## 12. Panel Kullanımı — OpenRouter Sayfası

| | |
|---|---|
| **Adres** | `GET /panel/ai` |
| **Yetki** | Yalnızca Admin |

### Hazır modeller

- `google/gemma-2-9b-it:free`
- `meta-llama/llama-3.2-3b-instruct:free`
- `google/gemma-3-27b-it:free`
- `deepseek/deepseek-chat-v3-0324:free`
- `anthropic/claude-3.5-sonnet`
- `openai/gpt-4o-mini`

OpenRouter çalışmazsa sistem otomatik anahtar kelime analizine geçer.

---

## 13. Panel Kullanımı — Haberler Sayfası

| | |
|---|---|
| **Adres** | `GET /panel/haberler` |
| **Yetki** | Admin + Editör |

### Filtre seçenekleri

| Filtre | Açıklama |
|--------|----------|
| tümü | Tüm kayıtlar |
| skor ≥ 5 | Orta ve üzeri |
| skor ≥ 8 (haber) | **Varsayılan** |

### WA durumu

| Gösterim | Anlam |
|----------|--------|
| WA ✓ | WhatsApp'a gönderildi |
| WA — | Gönderilmedi |

> En yeni kayıtlar üstte. Sayfa 15 saniyede bir yenilenir. Max 200 kayıt saklanır.

---

## 14. Panel Kullanımı — Ayarlar Sayfası

| | |
|---|---|
| **Adres** | `GET /panel/ayarlar` |
| **Yetki** | Yalnızca Admin |

| Alan | Açıklama | Varsayılan |
|------|----------|------------|
| Kontrol aralığı (sn) | Mail tarama sıklığı | 30 |
| WhatsApp bildirim eşiği | 0–10 | 8 |
| Panel min. skor filtresi | 0–10 | 5 |
| Kategoriler | Virgül ile ayrılmış liste | — |

Kaydet sonrası mail kontrol servisi yeniden başlar.

---

## 15. Panel Kullanımı — Panel Hesapları (Admin)

| | |
|---|---|
| **Adres** | `GET /panel/hesaplar` |
| **Yetki** | Yalnızca Admin |

### Yeni hesap ekleme

| Alan | Kural |
|------|-------|
| Kullanıcı adı | 3–32 karakter, küçük harf, rakam, alt çizgi |
| Görünen ad | Panelde görünecek isim |
| Şifre | Min. 12 karakter (zorunlu) |
| Rol | `admin` veya `editor` |

### Güvenlik kuralları

- Şifreler scrypt ile hashlenir; düz metin saklanmaz
- Son adminin rolü düşürülemez veya devre dışı bırakılamaz
- Kendi hesabınızı silemezsiniz
- Silinen kullanıcının aktif oturumları sonlandırılır

---

## 16. Haber Skorlama Mantığı

Her mail için **0–10** arası skor üretilir.

### 1. OpenRouter (LLM) analizi

Mail konusu ve gövdesi modele gönderilir. Model döndürür: skor, kategori, 2–3 cümlelik özet.

### 2. Anahtar kelime analizi (fallback)

OpenRouter kullanılamazsa devreye girer: güncellik ifadeleri, teknoloji kelimeleri, haber tipi ifadeleri, link sayısı.

### Skor sınıflandırması

| Skor | Rozet | Renk |
|------|-------|------|
| ≥ `yuksekHaberSkoru` (8) | Haber | Yeşil |
| ≥ 5 | Orta | Sarı |
| < 5 | Düşük | Gri |

### Gönderim kuralı

```
yuksekHaberSkoru = 8 (varsayılan)
Skor ≥ 8  →  WhatsApp bildirimi
Skor < 8  →  Yalnızca panelde kayıt
```

---

## 17. WhatsApp'a Giden Mesaj Formatı

```
📰 *Yüksek Haber Değeri*

📝 *Yeni ürün lansmanı*
📧 ornek@firma.com
📊 Skor: 8/10 · teknoloji

📋 *Özet:*
Apple yeni serisini tanıttı; lansman içeriği haber niteliğinde.
```

- Yalnızca skor ≥ `yuksekHaberSkoru` olan mailler gönderilir
- Özet alanı LLM veya anahtar kelime analizinden gelir

---

## 18. Tüm Endpoint Listesi

### Yetki açıklamaları

| Etiket | Anlam |
|--------|-------|
| `[Açık]` | Giriş gerekmez |
| `[Oturum]` | Giriş yapmış herhangi bir kullanıcı |
| `[Admin]` | Yalnızca admin rolü |
| `[CSRF]` | `X-CSRF-Token` başlığı zorunlu |

### 18.1 Kimlik doğrulama

| Method | Endpoint | Yetki | Açıklama |
|--------|----------|-------|----------|
| GET | `/giris` | Açık | Giriş sayfası |
| POST | `/api/auth/giris` | Açık | Giriş yap |
| GET | `/api/auth/me` | Oturum | Kullanıcı bilgisi + CSRF token |
| POST | `/api/auth/cikis` | Oturum, CSRF | Çıkış yap |

**POST `/api/auth/giris` istek:**

```json
{
  "username": "admin",
  "password": "sifreniz"
}
```

**Başarılı cevap:**

```json
{
  "basarili": true,
  "kullanici": {
    "id": "usr_...",
    "username": "admin",
    "displayName": "Yönetici",
    "role": "admin"
  }
}
```

### 18.2 Sayfa endpoint'leri

| Method | Endpoint | Yetki | Açıklama |
|--------|----------|-------|----------|
| GET | `/` | Oturum | `/panel`'a yönlendirir |
| GET | `/panel` | Oturum | Özet sayfası |
| GET | `/panel/haberler` | Oturum | Haberler |
| GET | `/panel/mail` | Admin | Mail hesapları |
| GET | `/panel/whatsapp` | Admin | WhatsApp |
| GET | `/panel/ai` | Admin | OpenRouter |
| GET | `/panel/ayarlar` | Admin | Ayarlar |
| GET | `/panel/hesaplar` | Admin | Panel kullanıcıları |
| GET | `/dashboard` | Oturum | `/panel/whatsapp`'a yönlendirir |

### 18.3 Durum

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/status` | Açık |

### 18.4 Ayarlar

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/api/settings` | Admin |
| PUT | `/api/settings` | Admin, CSRF |
| PUT | `/api/settings/openrouter` | Admin, CSRF |

### 18.5 Mail hesapları

| Method | Endpoint | Yetki |
|--------|----------|-------|
| POST | `/api/mail-accounts` | Admin, CSRF |
| PUT | `/api/mail-accounts/:id` | Admin, CSRF |
| DELETE | `/api/mail-accounts/:id` | Admin, CSRF |

### 18.6 WhatsApp

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/api/whatsapp/status` | Admin |
| POST | `/api/whatsapp` | Admin, CSRF |
| PUT | `/api/whatsapp/:id` | Admin, CSRF |
| DELETE | `/api/whatsapp/:id` | Admin, CSRF |
| POST | `/api/whatsapp/:id/reconnect` | Admin, CSRF |

### 18.7 Haberler

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/api/haberler` | Oturum |
| GET | `/api/haberler?minSkor=5` | Oturum |
| GET | `/api/haberler?minSkor=8` | Oturum |

### 18.8 Analiz testi

| Method | Endpoint | Yetki |
|--------|----------|-------|
| POST | `/api/ai/test` | Admin, CSRF |

### 18.9 Panel kullanıcıları

| Method | Endpoint | Yetki |
|--------|----------|-------|
| GET | `/api/panel-users` | Admin |
| POST | `/api/panel-users` | Admin, CSRF |
| PUT | `/api/panel-users/:id` | Admin, CSRF |
| DELETE | `/api/panel-users/:id` | Admin, CSRF |

### 18.10 Manuel mesaj

| Method | Endpoint | Yetki |
|--------|----------|-------|
| POST | `/mesaj-gonder` | Admin, CSRF |

---

## 19. Ayar Dosyası Yapısı (data/settings.json)

```json
{
  "checkInterval": 30000,
  "minHaberSkoru": 5,
  "yuksekHaberSkoru": 8,
  "sadeceYuksekHaber": true,
  "openrouter": {
    "apiKey": "sk-or-...",
    "model": "google/gemma-2-9b-it:free",
    "enabled": true
  },
  "mailAccounts": [
    {
      "id": "mail_abc123",
      "name": "Ana Mail",
      "host": "imap.gmail.com",
      "port": 993,
      "user": "mail@gmail.com",
      "password": "app_password",
      "enabled": true,
      "whatsappId": "default"
    }
  ],
  "whatsappConnections": [
    {
      "id": "default",
      "name": "Ana WhatsApp",
      "targetNumber": "905551234567",
      "enabled": true
    }
  ],
  "haberKategorileri": ["teknoloji", "yazilim", "donanim"]
}
```

---

## 20. Panel Kullanıcı Dosyası (data/panel-users.json)

Şifreler **scrypt hash** olarak tutulur; düz metin şifre asla yazılmaz.

```json
{
  "users": [
    {
      "id": "usr_6cbca56ba88d3ac3",
      "username": "admin",
      "displayName": "Yönetici",
      "role": "admin",
      "enabled": true,
      "passwordSalt": "a1b2c3...",
      "passwordHash": "d4e5f6...",
      "createdAt": "2026-08-10T10:00:00.000Z",
      "lastLogin": "2026-08-10T12:00:00.000Z"
    },
    {
      "id": "usr_abc123def456",
      "username": "editor1",
      "displayName": "Haber Editörü",
      "role": "editor",
      "enabled": true,
      "passwordSalt": "...",
      "passwordHash": "...",
      "createdAt": "2026-08-10T11:00:00.000Z",
      "lastLogin": null,
      "createdBy": "usr_6cbca56ba88d3ac3"
    }
  ]
}
```

> Dosya izinleri: `chmod 600`. Manuel düzenleme önerilmez; panel üzerinden yönetin.

---

## 21. Örnek Kullanım Senaryoları

### Senaryo 1 — İlk kurulum

1. `npm install && npm start`
2. `/giris` adresinden admin olarak giriş yap
3. WhatsApp bağlantısı kur (QR ile)
4. Mail hesabı ekle
5. OpenRouter anahtarını gir (opsiyonel)

### Senaryo 2 — Editör ekibi

1. Admin olarak `/panel/hesaplar`'a git
2. `editor1` kullanıcısı oluştur (rol: Editör)
3. Editör yalnızca Özet ve Haberler'i görür

### Senaryo 3 — Departman bazlı yönlendirme

1. "Haber Ekibi" ve "Operasyon" WhatsApp hatları oluştur
2. Her mail hesabını farklı hatta bağla

### Senaryo 4 — Üretim ortamı güvenliği

1. `.env`'de güçlü `PANEL_ADMIN_PASSWORD` ve `SESSION_SECRET`
2. `NODE_ENV=production` (Secure çerezler)
3. Reverse proxy ile HTTPS
4. Sunucuyu yalnızca iç ağ / VPN arkasında çalıştır

---

## 22. Sık Karşılaşılan Sorunlar

| Sorun | Çözüm |
|-------|--------|
| Port 5600 kullanımda | `lsof -ti:5600 \| xargs kill -9` → `npm start` |
| `/giris`'e yönlendiriliyor | Normal. Giriş yapın. İlk kurulumda konsol veya `.env` şifresi |
| Çok fazla başarısız deneme | 15 dakika bekleyin |
| WhatsApp bağlanmıyor | Yeniden bağlan, QR tekrar tara, gerekirse `whatsapp_auth/` temizle |
| Mail çekilmiyor | Host/port/şifre kontrol, Gmail App Password, IMAP açık mı |
| OpenRouter sonucu yok | API anahtarı, farklı model; fallback anahtar kelime çalışır |
| Mail var, WhatsApp yok | WA bağlı mı? Skor ≥ eşik mi? Hedef numara doğru mu? |
| Editör mail ayarlarına erişemiyor | Normal — Editör yalnızca Özet + Haberler |
| CSRF hatası (403) | Oturum süresi dolmuş; çıkış yapıp tekrar giriş yapın |

---

## 23. Güvenlik

### Hassas veriler

| Dosya | İçerik |
|-------|--------|
| `data/settings.json` | Mail şifreleri, OpenRouter API anahtarı |
| `data/panel-users.json` | Şifre hash'leri |
| `whatsapp_auth/` | WhatsApp oturum dosyaları |
| `data/haberler.json` | Mail içerikleri |
| `.env` | Tüm gizli anahtarlar |

### Mevcut koruma

Kimlik doğrulama · Rol tabanlı erişim · scrypt hash · HttpOnly çerezler · CSRF · Brute-force kilidi · Güvenlik başlıkları · Son admin koruması · API maskeleme

### Üretim önerileri

1. Güçlü `SESSION_SECRET` (min 32 karakter)
2. Güçlü `PANEL_ADMIN_PASSWORD` (min 12 karakter)
3. `NODE_ENV=production`
4. HTTPS (nginx, Caddy)
5. İç ağ / VPN
6. `data/` ve `whatsapp_auth/` erişim sınırı
7. Düzenli yedekleme
8. Editör hesaplarını ihtiyaç kadar ver

---

## 24. Hızlı Başvuru

```bash
npm start                          # Başlat
```

| | |
|---|---|
| Giriş | http://localhost:5600/giris |
| Panel | http://localhost:5600/panel |
| Durum | `GET http://localhost:5600/status` |

| Rol | Erişim |
|-----|--------|
| `admin` | Tam yetki |
| `editor` | Özet + Haberler (salt okunur) |

| Eşik | Değer | Açıklama |
|------|-------|----------|
| `yuksekHaberSkoru` | 8 | WhatsApp bildirim eşiği |
| `minHaberSkoru` | 5 | Panel filtre referansı |

---

## Değişiklik Geçmişi

### v2.0 (10 Ağustos 2026)

- Panel kimlik doğrulama sistemi (admin / editör rolleri)
- Panel hesap yönetimi (`/panel/hesaplar`)
- scrypt şifre hashleme, CSRF koruması, brute-force kilidi
- HttpOnly + HMAC imzalı oturum çerezleri
- Rol tabanlı menü ve API erişim kontrolü
- `data/panel-users.json`

### v1.1 (10 Ağustos 2026)

- `yuksekHaberSkoru` WhatsApp bildirim eşiği
- Kart listesi + offcanvas detay paneli
- WhatsApp mesaj formatı güncellendi

### v1.0 (10 Ağustos 2026)

- İlk sürüm: mail tarama, WhatsApp bildirimi, OpenRouter analizi

---

*Bu doküman ClauraOffice sisteminin kurulum, panel kullanımı, kimlik doğrulama, rol yönetimi, endpoint listesi, güvenlik ve günlük operasyon adımlarını kapsar.*
