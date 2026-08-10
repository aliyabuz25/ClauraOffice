# ClauraOffice

**Akıllı e-posta izleme ve haber bildirim platformu**

Gelen kutularını otomatik tarar, haber değerini puanlar, yayımlanabilir taslak metin üretir ve yüksek skorlu içerikleri WhatsApp üzerinden kaynak bilgisiyle birlikte iletir.

[Kurulum](#kurulum) · [Özellikler](#özellikler) · [Panel](#panel) · [API](#api-özeti) · [Güvenlik](#güvenlik) · [Dokümantasyon](#dokümantasyon)

---

## Neden ClauraOffice?

| Sorun | Çözüm |
|-------|--------|
| Basın / PR mailleri gelen kutusunda kaybolur | Periyodik IMAP tarama |
| Her mail bildirim olunca gürültü artar | 0–10 haber skoru + eşik filtresi |
| Editör ham maili yeniden yazmak zorunda kalır | AI / kurallı **taslak haber** üretimi |
| Kaynak kişiye ulaşmak gecikir | Gönderen **ad, e-posta, GSM** çıkarımı |
| Operasyon ve yönetim aynı ekranda karışır | **Admin / Editör** rolleri |

---

## Özellikler

### İzleme ve analiz
- Çoklu IMAP hesap desteği
- OpenRouter LLM analizi (manşet, özet, taslak, editör notu)
- LLM yoksa anahtar kelime tabanlı fallback skorlama
- Yapılandırılabilir WhatsApp bildirim eşiği (`yuksekHaberSkoru`, varsayılan **8**)

### Bildirim
- Çoklu WhatsApp hattı (Baileys)
- Mail hesabı → hat eşlemesi
- Düzgün formatlı bildirim: manşet, taslak haber, kaynak / GSM

### Panel
- Giriş zorunlu, CSRF korumalı API
- Kart listesi + detay paneli (taslak kopyala, GSM kopyala)
- Admin: mail, WhatsApp, OpenRouter, ayarlar, kullanıcı yönetimi
- Editör: özet + haberler (salt okunur)

### Güvenlik
- scrypt şifre hash
- HttpOnly + SameSite + HMAC imzalı oturum
- Brute-force kilidi (8 deneme / 15 dk)
- Hassas veriler `.gitignore` altında

---

## Teknoloji

| Katman | Teknoloji |
|--------|-----------|
| Runtime | Node.js 18+ |
| HTTP | Express |
| Panel | EJS |
| Mail | IMAP + mailparser |
| WhatsApp | Baileys |
| AI | OpenRouter (opsiyonel) |

---

## Kurulum

### 1. Depoyu alın

```bash
git clone https://github.com/KULLANICI/ClauraOffice.git
cd ClauraOffice
npm install
```

### 2. Ortam dosyasını oluşturun

```bash
cp .env.example .env
```

En az şu alanları doldurun:

```env
PANEL_ADMIN_USERNAME=admin
PANEL_ADMIN_PASSWORD=guclu_sifre_min_12_karakter
SESSION_SECRET=rastgele_en_az_32_karakter_uzunlugunda_gizli_anahtar
```

> `PANEL_ADMIN_PASSWORD` tanımlı değilse sistem rastgele şifre üretir ve **konsola yazar**.

### 3. Başlatın

```bash
npm start
# geliştirme: npm run dev
```

| Adres | Açıklama |
|-------|----------|
| http://localhost:5600/giris | Panel girişi |
| http://localhost:5600/panel | Özet |
| http://localhost:5600/status | Sağlık / WhatsApp durumu |

### 4. İlk yapılandırma (Admin)

1. WhatsApp bağlantısı kurun → `/panel/whatsapp` (QR)
2. Mail hesabı ekleyin → `/panel/mail`
3. (Opsiyonel) OpenRouter → `/panel/ai`
4. (Opsiyonel) Editör hesabı → `/panel/hesaplar`

---

## Ortam değişkenleri

| Değişken | Zorunlu | Açıklama |
|----------|---------|----------|
| `PANEL_ADMIN_USERNAME` | Kurulum | İlk admin kullanıcı adı |
| `PANEL_ADMIN_PASSWORD` | Kurulum | Min. 12 karakter |
| `SESSION_SECRET` | Üretim | Min. 32 karakter, rastgele |
| `PORT` | Hayır | Varsayılan `5600` |
| `YUKSEK_HABER_SKORU` | Hayır | WhatsApp eşiği, varsayılan `8` |
| `MIN_HABER_SKORU` | Hayır | Panel filtre referansı, varsayılan `5` |
| `CHECK_INTERVAL` | Hayır | Tarama aralığı (ms), varsayılan `30000` |
| `OPENROUTER_API_KEY` | Hayır | AI analiz için |
| `OPENROUTER_MODEL` | Hayır | Model adı |
| `SMTP_HOST` / `EMAIL_USER` / … | Hayır | İlk mail/WA hesabını `.env`'den seed eder |

Tam liste: [`.env.example`](.env.example)

---

## Panel

### Roller

| | Admin | Editör |
|---|:-----:|:------:|
| Özet / Haberler | ✓ | ✓ |
| Mail / WhatsApp / OpenRouter / Ayarlar | ✓ | |
| Panel hesap yönetimi | ✓ | |
| API ile ayar değiştirme | ✓ | |

### Haber akışı

```
IMAP → skor + manşet + taslak → panel kaydı
              ↓
     skor ≥ eşik?
              ↓ evet
     WhatsApp (taslak + kaynak + GSM)
```

### Örnek WhatsApp bildirimi

```text
CLAURAOFFICE · HABER BİLDİRİMİ

Apple, iPhone 17 serisini resmi olarak duyurdu

Değer: Yüksek haber değeri  ·  Skor: 9/10
Kategori: akıllı-telefon

────────────
TASLAK HABER

Apple, iPhone 17 serisini resmi olarak tanıttı. …
────────────
KAYNAK / GÖNDEREN

Ad: Ayşe Yılmaz
E-posta: press@ornekfirma.com
GSM: +90 532 111 22 33
```

---

## Proje yapısı

```text
ClauraOffice/
├── src/
│   ├── app.js
│   ├── middleware/auth.js
│   ├── routes/panelRoutes.js
│   ├── services/
│   │   ├── authService.js
│   │   ├── settingsStore.js
│   │   ├── mailChecker.js
│   │   └── baileysManager.js
│   ├── utils/
│   │   ├── haberDegeri.js
│   │   └── gonderenBilgi.js
│   └── views/
├── data/                 # runtime (gitignore)
├── whatsapp_auth/        # WA oturumları (gitignore)
├── docs/
│   └── ClauraOffice-Dokumantasyon.md
├── .env.example
└── package.json
```

---

## API özeti

Yetki: `[Açık]` · `[Oturum]` · `[Admin]` · `[CSRF]` = `X-CSRF-Token` zorunlu

| Method | Endpoint | Yetki |
|--------|----------|-------|
| POST | `/api/auth/giris` | Açık |
| POST | `/api/auth/cikis` | Oturum + CSRF |
| GET | `/api/auth/me` | Oturum |
| GET | `/api/haberler?minSkor=8` | Oturum |
| GET/PUT | `/api/settings` | Admin (+ CSRF PUT) |
| CRUD | `/api/mail-accounts` | Admin + CSRF |
| CRUD | `/api/whatsapp` | Admin + CSRF |
| CRUD | `/api/panel-users` | Admin + CSRF |
| POST | `/api/ai/test` | Admin + CSRF |
| GET | `/status` | Açık |

Detaylı referans: [`docs/ClauraOffice-Dokumantasyon.md`](docs/ClauraOffice-Dokumantasyon.md)

---

## Güvenlik

**Commit edilmez:** `.env`, `data/settings.json`, `data/panel-users.json`, `data/haberler.json`, `whatsapp_auth/`

**Üretim kontrol listesi**

- [ ] Güçlü `SESSION_SECRET` ve admin şifresi
- [ ] `NODE_ENV=production`
- [ ] HTTPS (reverse proxy)
- [ ] İç ağ / VPN erişimi
- [ ] Editör hesaplarını yalnızca ihtiyaca göre verin

---

## Dokümantasyon

| Dosya | İçerik |
|-------|--------|
| [`docs/ClauraOffice-Dokumantasyon.md`](docs/ClauraOffice-Dokumantasyon.md) | Tam teknik ve operasyon kılavuzu |
| [`.env.example`](.env.example) | Ortam şablonu |

---

## Lisans

Kurumsal / özel kullanım. Açık kaynak lisansı eklenecekse depo ayarlarından güncelleyin.

---

<p align="center">
  <sub>ClauraOffice · gelen kutusu → skor → taslak → WhatsApp</sub>
</p>
