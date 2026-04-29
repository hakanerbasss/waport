# 📡 WaPort — Self-Hosted WhatsApp API

**Kendi sunucunda çalışan WhatsApp API sistemi.**  
Excel, Python, Node.js, n8n ve HTTP isteği atabilen her şeyden mesaj gönder.

🇬🇧 [English README](README.md)

🌐 **Website:** [waport.wizaicorp.com](https://waport.wizaicorp.com)  
📦 **Panel:** [wa.wizaicorp.com](https://wa.wizaicorp.com)

---

## 🚀 Kurulum

### Gereksinimler
- Node.js 18+
- Termux / VPS / Linux / macOS

### Hızlı Kurulum

```bash
unzip whatsapp-api-setup.zip
cd whatsapp-api-setup
node install.js
```

Sorular:
- `Bu cihaz mi VPS mi?` → `1` (local) veya `2` (VPS)
- `Port?` → Enter (varsayılan 3000)
- `Başlatılsın mı?` → `e`

### Termux (Android)

```bash
pkg install nodejs
cd /sdcard
unzip whatsapp-api-setup.zip
cd whatsapp-api-setup
node install.js
```

Kolay başlatma alias:
```bash
echo "alias wa='pkill -f src/index.js 2>/dev/null; sleep 1; cd ~/whatsapp-api && node src/index.js'" >> ~/.bashrc
source ~/.bashrc
wa
```

### VPS Kurulumu

```bash
node install.js  # 2 seç, IP/root/port gir
```

Panel: `http://SUNUCU_IP:3000`

VPS yeniden başlatma:
```bash
ssh root@SUNUCU_IP
cd ~/whatsapp-api
nohup node src/index.js > logs/server.log 2>&1 &
```

---

## 📋 Panel Sayfaları

| URL | Açıklama |
|---|---|
| `/` | Admin Panel |
| `/kayit` | Müşteri demo kaydı |
| `/musteri` | Müşteri dashboard |
| `/toplu` | Toplu mesaj paneli |

---

## 🔌 API Kullanımı

Tüm isteklerde header gerekli:
```
X-Api-Token: YOUR_TOKEN
Content-Type: application/json
```

---

### Durum Kontrolü

```http
GET /api/status
```

```bash
curl https://your-server:3000/api/status \
  -H "X-Api-Token: YOUR_TOKEN"
```

**Yanıt:**
```json
{
  "customer": "Ad Soyad",
  "plan": "basic",
  "daily_limit": 500,
  "wa_status": "connected",
  "wa_number": "905xxxxxxxxx",
  "expires_at": "2025-12-31"
}
```

---

### Metin Mesajı Gönder

```http
POST /api/send
```

```bash
curl -X POST https://your-server:3000/api/send \
  -H "X-Api-Token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"to": "905xxxxxxxxx", "message": "Merhaba! 👋"}'
```

**Yanıt:**
```json
{"ok": true, "message": "Mesaj gönderildi"}
```

---

### Resim Gönder (URL)

```http
POST /api/send/image
```

```bash
curl -X POST https://your-server:3000/api/send/image \
  -H "X-Api-Token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "905xxxxxxxxx",
    "url": "https://example.com/resim.jpg",
    "caption": "Ürün görseli"
  }'
```

---

### Dosya Gönder (URL)

```http
POST /api/send/document
```

```bash
curl -X POST https://your-server:3000/api/send/document \
  -H "X-Api-Token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "905xxxxxxxxx",
    "url": "https://example.com/dosya.pdf",
    "filename": "fatura.pdf"
  }'
```

---

### Medya Gönder (Dosya Yükle)

```http
POST /api/send/media
Content-Type: multipart/form-data
```

```bash
curl -X POST https://your-server:3000/api/send/media \
  -H "X-Api-Token: YOUR_TOKEN" \
  -F "to=905xxxxxxxxx" \
  -F "caption=Resim açıklaması" \
  -F "file=@/path/to/resim.jpg"
```

Desteklenen: `image/*`, `video/*`, `audio/*`, `application/*`

---

### Toplu Mesaj

```http
POST /api/send/bulk
```

```bash
curl -X POST https://your-server:3000/api/send/bulk \
  -H "X-Api-Token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"to": "905111111111", "message": "Kampanya mesajı 1"},
      {"to": "905222222222", "message": "Kampanya mesajı 2"},
      {"to": "905333333333", "message": "Kampanya mesajı 3"}
    ]
  }'
```

**Yanıt:**
```json
{
  "ok": true,
  "results": [
    {"to": "905111111111", "ok": true},
    {"to": "905222222222", "ok": true},
    {"to": "905333333333", "ok": false, "error": "..."}
  ]
}
```

> ⚠️ Maksimum 50 mesaj/istek. Mesajlar arası 500ms bekleme otomatik eklenir.

---

### Numara Kontrolü

```http
POST /api/check
```

```bash
curl -X POST https://your-server:3000/api/check \
  -H "X-Api-Token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"phones": ["905111111111", "905222222222"]}'
```

---

### WhatsApp Bağlantı

```http
POST /api/connect
```

```bash
# QR ile bağlan
curl -X POST https://your-server:3000/api/connect \
  -H "X-Api-Token: YOUR_TOKEN" \
  -d '{"method": "qr"}'

# Pairing code ile (telefon yanında olmasa da olur)
curl -X POST https://your-server:3000/api/connect \
  -H "X-Api-Token: YOUR_TOKEN" \
  -d '{"method": "pairing", "phone": "905xxxxxxxxx"}'
```

---

## 💻 Kod Örnekleri

### Python

```python
import requests

BASE = "https://your-server:3000"
TOKEN = "YOUR_TOKEN"
HEADERS = {"X-Api-Token": TOKEN, "Content-Type": "application/json"}

def send(to, message):
    r = requests.post(f"{BASE}/api/send", headers=HEADERS,
                      json={"to": to, "message": message})
    return r.json()

def send_bulk(messages):
    r = requests.post(f"{BASE}/api/send/bulk", headers=HEADERS,
                      json={"messages": messages})
    return r.json()

# Kullanım
send("905xxxxxxxxx", "Merhaba! 👋")

send_bulk([
    {"to": "905111111111", "message": "Mesaj 1"},
    {"to": "905222222222", "message": "Mesaj 2"},
])
```

---

### Node.js

```javascript
const BASE = 'https://your-server:3000';
const TOKEN = 'YOUR_TOKEN';
const headers = { 'X-Api-Token': TOKEN, 'Content-Type': 'application/json' };

async function send(to, message) {
  const res = await fetch(`${BASE}/api/send`, {
    method: 'POST', headers,
    body: JSON.stringify({ to, message })
  });
  return res.json();
}

async function sendImage(to, url, caption) {
  const res = await fetch(`${BASE}/api/send/image`, {
    method: 'POST', headers,
    body: JSON.stringify({ to, url, caption })
  });
  return res.json();
}

send('905xxxxxxxxx', 'Merhaba! 👋');
sendImage('905xxxxxxxxx', 'https://example.com/img.jpg', 'Ürün');
```

---

### Excel VBA

```vba
Sub WhatsAppGonder()
    Dim http As Object
    Dim url As String, body As String
    
    url = "https://your-server:3000/api/send"
    
    ' A sütunu: telefon, B sütunu: mesaj, C sütunu: sonuç
    Dim i As Integer
    For i = 2 To 100
        If Cells(i, 1).Value = "" Then Exit For
        
        body = "{""to"":""" & Cells(i, 1).Value & """," & _
               """message"":""" & Cells(i, 2).Value & """}"
        
        Set http = CreateObject("MSXML2.XMLHTTP")
        http.Open "POST", url, False
        http.setRequestHeader "X-Api-Token", "YOUR_TOKEN"
        http.setRequestHeader "Content-Type", "application/json"
        http.Send body
        
        Cells(i, 3).Value = http.responseText
        Application.Wait Now + TimeValue("0:00:01")
    Next i
    
    MsgBox "Tamamlandı!"
End Sub
```

---

### Google Apps Script

```javascript
function sendWhatsApp(to, message) {
  const url = 'https://your-server:3000/api/send';
  const options = {
    method: 'POST',
    headers: {
      'X-Api-Token': 'YOUR_TOKEN',
      'Content-Type': 'application/json'
    },
    payload: JSON.stringify({ to, message })
  };
  return JSON.parse(UrlFetchApp.fetch(url, options).getContentText());
}

function sendFromSheet() {
  const sheet = SpreadsheetApp.getActiveSheet();
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    const phone = data[i][0];
    const message = data[i][1];
    if (!phone) break;
    const result = sendWhatsApp(phone.toString(), message);
    sheet.getRange(i + 1, 3).setValue(result.ok ? '✓' : '✗');
    Utilities.sleep(1000);
  }
}
```

---

### n8n / Make / Zapier

```json
{
  "method": "POST",
  "url": "https://your-server:3000/api/send",
  "headers": {
    "X-Api-Token": "YOUR_TOKEN",
    "Content-Type": "application/json"
  },
  "body": {
    "to": "={{ $json.phone }}",
    "message": "={{ $json.message }}"
  }
}
```

---

## 👥 Müşteri Akışı

1. `/kayit` → Demo kaydı (50 mesaj ücretsiz)
2. `/musteri?token=xxx` → Dashboard'a giriş
3. WhatsApp bağla (QR veya pairing code)
4. API token al, kullanmaya başla
5. Limit bitince "Plan Yükselt" görünür
6. Admin WhatsApp'a bildirim gelir
7. Admin planı yükseltir
8. Müşteriye otomatik WhatsApp bildirimi gider

---

## 🔧 Sorun Giderme

```bash
# Port meşgul
pkill -9 node && wa

# Şifre unuttum
cd ~/whatsapp-api && node reset-password.js

# Veritabanı bozuldu
rm ~/whatsapp-api/data/database.bin && wa

# VPS yeniden başlatma
ssh root@IP
cd ~/whatsapp-api
nohup node src/index.js > logs/server.log 2>&1 &
```

---

## 📊 Hata Kodları

| Kod | Açıklama |
|---|---|
| `400` | Eksik parametre |
| `401` | Geçersiz token |
| `403` | Hesap pasif veya süresi dolmuş |
| `429` | Günlük limit aşıldı |
| `500` | Sunucu hatası |

---

## 📞 İletişim & Destek

- 🌐 [waport.wizaicorp.com](https://waport.wizaicorp.com)
- 💬 WhatsApp: [+90 553 093 03 25](https://wa.me/905530930325)
- 📧 wizaicorp@gmail.com

---

*WaPort by [WizAICorp](https://wizaicorp.com)*
