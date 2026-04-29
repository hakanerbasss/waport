# WaPort — Kurulum & Güncelleme Rehberi

> Bu dosyayı GitHub reposuna ekle: `hakanerbasss/waport/WAPORT_REHBER.md`  
> 6 ay sonra bile sıfırdan kurabilirsin.

---

## 1. Proje Bilgileri

| Bilgi | Değer |
|---|---|
| **Ürün adı** | WaPort |
| **Site URL** | https://waport.wizaicorp.com |
| **Yedek URL** | https://waport.wizaicorp.workers.dev |
| **GitHub repo** | https://github.com/hakanerbasss/waport |
| **Cloudflare Worker** | `waport` |
| **Cloudflare hesap** | wizaicorp@gmail.com |
| **Cloudflare Account ID** | a5261bd67f0f14f94f1d22e6a6511d5c |
| **WhatsApp 1** | +905530930325 |
| **WhatsApp 2** | +905461427866 |
| **Email** | wizaicorp@gmail.com |
| **WA Panel** | https://wa.wizaicorp.com |

---

## 2. Repo Yapısı

```
waport/
├── public/
│   ├── index.html        ← Ana site dosyası
│   ├── prices.json       ← Fiyatlar (buradan değiştir)
│   └── i18n.json         ← TR/EN metinler
├── .github/
│   └── workflows/
│       └── deploy.yml    ← GitHub Actions otomatik deploy
├── wrangler.toml         ← Cloudflare Worker ayarları
├── .gitignore
└── WAPORT_REHBER.md      ← Bu dosya
```

---

## 3. Sıfırdan Kurulum (Termux Silinmişse)

### 3.1 Termux Kurulumu
```bash
# Termux'u Play Store veya F-Droid'den kur
# Açtıktan sonra:
pkg update && pkg upgrade
pkg install git
```

### 3.2 GitHub Token Oluştur
1. https://github.com/settings/tokens/new adresine git
2. Note: `waport-deploy`
3. Expiration: `No expiration`
4. Scope: **repo** ✓ ve **workflow** ✓ işaretle
5. **Generate token** → token'ı kopyala

### 3.3 Repoyu Klonla
```bash
cd ~
git clone https://hakanerbasss:GITHUB_TOKEN@github.com/hakanerbasss/waport.git
cd waport
git config user.email "hakanerbasss@gmail.com"
git config user.name "Hakan Erbaş"
```

### 3.4 Deploy Alias Ekle
```bash
echo 'alias wdeploy="cd ~/waport && git add -A && git commit -m \"update\" && git push"' >> ~/.bashrc
source ~/.bashrc
```

Artık `wdeploy` komutu ile deploy edebilirsin.

---

## 4. GitHub Actions Secrets (Bir Kere Kurulur)

GitHub repo → Settings → Secrets and variables → Actions

| Secret Adı | Değer |
|---|---|
| `CF_ACCOUNT_ID` | `a5261bd67f0f14f94f1d22e6a6511d5c` |
| `CF_API_TOKEN` | Cloudflare'den yeni token oluştur (aşağıya bak) |

### Cloudflare API Token Oluşturma
1. https://dash.cloudflare.com/profile/api-tokens
2. **Create Token** → **Edit Cloudflare Workers** template
3. **Use template** → **Create Token**
4. Token'ı GitHub secret olarak ekle

---

## 5. Günlük Kullanım

### Fiyat Değiştirme
```bash
cd ~/waport
nano public/prices.json
# Değiştir, Ctrl+X → Y → Enter
wdeploy
```

### Metin Değiştirme (TR/EN)
```bash
cd ~/waport
nano public/i18n.json
wdeploy
```

### HTML Düzenleme
```bash
cd ~/waport
nano public/index.html
wdeploy
```

---

## 6. Manuel Deploy (wdeploy çalışmazsa)

```bash
cd ~/waport
git add -A
git commit -m "guncelleme"
git push
```

---

## 7. GitHub Actions deploy.yml İçeriği

```yaml
name: Deploy to Cloudflare Workers

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Deploy with Wrangler
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          accountId: ${{ secrets.CF_ACCOUNT_ID }}
```

---

## 8. wrangler.toml İçeriği

```toml
name = "waport"
compatibility_date = "2026-04-29"

[assets]
directory = "./public"
```

---

## 9. Cloudflare DNS Ayarı

`waport.wizaicorp.com` → Cloudflare Workers custom domain olarak bağlı.  
DNS değişikliği gerekmez, Cloudflare otomatik yönetir.

---

## 10. Token Güvenliği

> ⚠️ GitHub token ve Cloudflare API token'larını asla herkese açık paylaşma.

Token iptal etmek:
- **GitHub:** https://github.com/settings/tokens
- **Cloudflare:** https://dash.cloudflare.com/profile/api-tokens

---

## 11. Sorun Giderme

| Sorun | Çözüm |
|---|---|
| `git push` reddedildi | GitHub token süresi dolmuş, yeni token oluştur |
| GitHub Actions kırmızı | https://github.com/hakanerbasss/waport/actions → hata loguna bak |
| Site güncellenmedi | Cloudflare cache — URL sonuna `?v=2` ekle |
| Cloudflare token hatası | https://dash.cloudflare.com/profile/api-tokens → yeni token oluştur, GitHub secret güncelle |
| `wdeploy` bulunamadı | `source ~/.bashrc` çalıştır |

---

*Son güncelleme: Nisan 2026*
