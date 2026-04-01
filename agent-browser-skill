---
name: "agent-browser"
description: "Browser automation CLI for UI testing, E2E tests, and page interaction. Invoke when user wants to test UI, click buttons, fill forms, or take screenshots."
---

# Agent Browser

Native Rust CLI for browser automation. Lightweight alternative to Playwright.

## Setup

```bash
npm install -g agent-browser
agent-browser install  # First time only
```

## Core Commands

| Command | Description |
|---------|-------------|
| `agent-browser open <url>` | Navigate to URL |
| `agent-browser snapshot` | Get accessibility tree with refs |
| `agent-browser screenshot [path]` | Take screenshot |
| `agent-browser click <sel>` | Click element |
| `agent-browser fill <sel> <text>` | Fill input |
| `agent-browser close` | Close browser |
| `agent-browser find <criteria>` | Find element by role/text/label |
| `agent-browser keyboard type <key>` | Send keyboard input |
| `agent-browser scroll <direction> <px>` | Scroll page |
| `agent-browser mouse <action>` | Mouse operations (move, down, up, wheel) |

## Trae Sandbox Notları

Sandbox ortamında `~/.agent-browser` socket dizinine yazma izni olmadığı için:

```bash
# Her komutu şu prefix ile çalıştır:
HOME=/tmp agent-browser <komut>

# Örnek:
HOME=/tmp agent-browser open http://localhost:5173
HOME=/tmp agent-browser snapshot
HOME=/tmp agent-browser click @e5
```

## Combobox/Dropdown Click Sorunu

Bootstrap custom select'lerde `click @ref` dropdown option'da çalışmayabilir. Çözümler:

```bash
# Çözüm 1: find ile text match
HOME=/tmp agent-browser find text "Kg" click

# Çözüm 2: nth ile spesifik seçim (birden fazla eşleşme varsa)
HOME=/tmp agent-browser find nth 1 text "Kg" click

# Çözüm 3: Keyboard navigation
HOME=/tmp agent-browser click @e44              # Combobox'u aç
HOME=/tmp agent-browser keyboard type "key ArrowDown"  # İlk seçenek
HOME=/tmp agent-browser keyboard type "Enter"           # Seç
```

## Doğrulama Stratejisi

```bash
# Her adımdan sonra snapshot kontrol et
HOME=/tmp agent-browser snapshot | grep "validation mesajı"

# Şüphe varsa screenshot al
HOME=/tmp agent-browser screenshot ./test-step-X.png

# Ref değişebilir - her seferinde kontrol et
HOME=/tmp agent-browser snapshot | grep -A 5 "element adi"

# Dialog içeriğini kontrol et
HOME=/tmp agent-browser snapshot | grep -A 30 "dialog"
```

## Selector Types

- **Ref**: `@e1`, `@e2` (from snapshot)
- **CSS**: `#id`, `.class`, `button`
- **Text**: `button:has-text("Submit")`
- **Role**: `find role button click --name "Submit"`
- **Text match**: `find text "Kg" click`
- **Nth selection**: `find nth 1 text "Kg" click`

## Usage Flow

1. **Browser'ı aç ve kapat**
   ```bash
   # Açık browser varsa kapat
   HOME=/tmp agent-browser close --all 2>/dev/null; sleep 1

   # Sayfayı aç
   HOME=/tmp agent-browser open http://localhost:5173/retailer/application/create
   ```

2. **Elementleri incele**
   ```bash
   HOME=/tmp agent-browser snapshot  # @e1, @e2 gibi ref'leri gösterir
   ```

3. **İnteraksyon**
   ```bash
   HOME=/tmp agent-browser click @e5                      # Ref ile tıkla
   HOME=/tmp agent-browser find text "Kg" click           # Text ile tıkla
   HOME=/tmp agent-browser fill @e10 "değer"             # Input doldur
   HOME=/tmp agent-browser scroll down 300                # Scroll
   ```

4. **Doğrula**
   ```bash
   HOME=/tmp agent-browser screenshot ./test-step-X.png   # Screenshot
   HOME=/tmp agent-browser snapshot | grep "mesaj"       # İçerik kontrolü
   ```

5. **Temizlik**
   ```bash
   HOME=/tmp agent-browser close
   ```

## Examples

### Retailer Panel - Login

```bash
HOME=/tmp agent-browser close --all 2>/dev/null; sleep 1
HOME=/tmp agent-browser open http://localhost:5173/login
HOME=/tmp agent-browser snapshot

# Form alanları (ref'ler değişebilir, snapshot ile kontrol et)
HOME=/tmp agent-browser fill input[type='email'] "basak@tarim.com"
HOME=/tmp agent-browser fill input[type='password'] "basaktarim123"
HOME=/tmp agent-browser click "button[type='submit']"

# Login sonrası yönlendirme kontrolü
sleep 2
HOME=/tmp agent-browser screenshot ./retailer-login.png
HOME=/tmp agent-browser snapshot | head -20
```

### Retailer Panel - Ürün Ekleme

```bash
HOME=/tmp agent-browser open http://localhost:5173/retailer/application/create
HOME=/tmp agent-browser snapshot

# "Ürün ekle" butonunu bul ve tıkla
HOME=/tmp agent-browser find text "Ürün ekle" click
sleep 1

# Modal açıldı mı kontrol et
HOME=/tmp agent-browser screenshot ./modal-open.png
HOME=/tmp agent-browser snapshot | grep -A 30 "dialog"

# Ürün ara
HOME=/tmp agent-browser click @e49  # Ürün arama kutusu
HOME=/tmp agent-browser type @e49 "buğday"
sleep 1
HOME=/tmp agent-browser find text "Buğday" click

# Marka seç (dropdown açıkken)
HOME=/tmp agent-browser find text "Marka" click
sleep 1
HOME=/tmp agent-browser find nth 1 text "Abalım" click

# Birim seç
HOME=/tmp agent-browser find text "Birim" click
sleep 1
HOME=/tmp agent-browser find text "Kg" click

# Adet ve fiyat doldur
HOME=/tmp agent-browser fill @e45 "10"
HOME=/tmp agent-browser fill @e46 "100"

# Form kontrolü
HOME=/tmp agent-browser snapshot | grep "Ekle"
HOME=/tmp agent-browser screenshot ./product-form-filled.png
```

### Admin Panel - Login

```bash
HOME=/tmp agent-browser close --all 2>/dev/null; sleep 1
HOME=/tmp agent-browser open http://localhost:5173/admin/login
HOME=/tmp agent-browser fill input[type='email'] "alper.guven@tarfin.com"
HOME=/tmp agent-browser fill input[type='password'] "93209320Tt"
HOME=/tmp agent-browser click "button[type='submit']"
sleep 2
HOME=/tmp agent-browser screenshot ./admin-login.png
```

## Requirements

- Chrome for Testing (`agent-browser install`)
- No Node.js/Playwright dependency needed for the daemon

## Troubleshooting

| Sorun | Çözüm |
|-------|-------|
| Socket permission error | `HOME=/tmp agent-browser` kullan |
| Element not found | Snapshot ile ref kontrol et |
| Click çalışmıyor | `find text` veya `keyboard` dene |
| Dropdown açılmıyor | Combobox'a tıkla, sonra `find text` ile seçenek seç |
| Form submit olmuyor | Validation mesajlarını kontrol et (`snapshot \| grep "zorunlu"`) |

