# Claude Opus 4.6 Model Support - Antigravity IDE

## Problem

Claude Opus 4.6 modeli, Google Antigravity IDE (VS Code fork) uzerindeki Claude eklentisinde
gorunmuyor. Eklenti baslatildiginda `Error: spawn ENOEXEC` hatasi aliyor.

## Neden Gorunmuyor?

### 1. Model Cok Yeni Yayinlandi

Claude Opus 4.6, **5 Subat 2026** tarihinde yayinlandi. Claude Code VS Code eklentisinin
bu modeli desteklemesi icin guncellenmis olmasi gerekiyor.

### 2. Eklenti Versiyonu Eski Olabilir

Claude Code eklentisinin en az **v2.1.32** veya ustu bir surume guncellenmis olmasi gerekir.
Opus 4.6 destegi bu surumle birlikte gelmektedir.

**Kontrol icin:**
- Antigravity'de Extensions panelini acin
- "Claude Code" eklentisini bulun
- Surum numarasini kontrol edin
- Guncelleme varsa "Update" butonuna tiklayin

### 3. Antigravity Uyumluluk Sorunlari

Antigravity, VS Code OSS v1.104 tabanli olup VS Code'un en son surumlerinin (v1.108+)
tum ozelliklerini desteklememektedir. Bilinen uyumluluk sorunlari:

- Secondary Sidebar destegi eksik
- Sidebar ikonlari kaybolabiliyor
- Bazi extension API'lari tam desteklenmiyor

### 4. spawn ENOEXEC Hatasi (Native Binary Sorunu)

Bu hata, eklentinin icinde gelen native binary dosyasinin calistirilamadigi anlamina gelir.
Eklenti Claude Code'u baslatmak icin su yolu kullaniyor:

```
~/.antigravity/extensions/anthropic.claude-code-<version>/resources/native-binary/claude
```

**ENOEXEC** (Exec format error) su sebeplerden olusur:

#### a) Yanlis Platform Binary'si (Dogrulanmis Neden)

Antigravity marketplace'i macOS icin **yanlis platform binary'si** dagitabiliyor.
Dogrulanan ornekte, macOS Apple Silicon (arm64) uzerinde binary su sekilde tespit edildi:

```
ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV),
dynamically linked, interpreter /lib/ld-musl-aarch64.so.1
```

Bu bir **Linux Alpine ARM64** binary'sidir. macOS'un bekledigin format ise **Mach-O**'dur.
Mimari (ARM64) dogru olmasina ragmen, dosya formati (ELF vs Mach-O) uyumsuz oldugu
icin `ENOEXEC` hatasi olusur.

Kontrol icin:

```bash
# Binary'nin formatini kontrol edin
file ~/.antigravity/extensions/anthropic.claude-code-2.1.34/resources/native-binary/claude

# Mac'inizin mimarisini kontrol edin
uname -m
```

Beklenen ciktilar (macOS):
- Apple Silicon: `Mach-O 64-bit executable arm64`
- Intel: `Mach-O 64-bit executable x86_64`

Eger ciktida `ELF` goruyorsaniz, yanlis platform binary'si yuklenmistir.

#### b) Execute Izni Eksik

Binary dosyasinin calistirma izni olmayabilir:

```bash
chmod +x ~/.antigravity/extensions/anthropic.claude-code-2.1.34/resources/native-binary/claude
```

#### c) Binary Bozuk veya Eksik Indirilmis

Eklenti marketinden indirme sirasinda dosya bozulmus olabilir. Eklentiyi kaldirip
yeniden yuklemek sorunu cozebilir.

#### d) Cozum: Sistem Genelindeki Claude CLI'a Yonlendirme

Eger binary uyumsuzsa, Claude Code CLI'i ayri kurup eklentiyi ona yonlendirebilirsiniz:

```bash
# Claude Code CLI kurulumu (dogru mimari icin otomatik indirir)
npm install -g @anthropic-ai/claude-code

# Kurulum yolunu bulun
which claude
# Cikti: /opt/homebrew/bin/claude (Apple Silicon) veya /usr/local/bin/claude (Intel)
```

Antigravity `settings.json` dosyasina ekleyin:

```json
{
  "claude-code.executablePath": "/opt/homebrew/bin/claude"
}
```

### 5. API Erisim / Abonelik Planini Kontrol Edin

Opus 4.6'ya erisim icin uygun bir plana sahip olmaniz gerekir:
- **Claude Pro/Max** aboneligi (claude.ai uzerinden)
- **API Key** ile dogrudan erisim (API konsolu uzerinden)
- **GitHub Copilot** Pro, Pro+, Business veya Enterprise planlari

## Cozum Adimlari

### Adim 1: Claude Code Eklentisini Guncelleyin

```
# Antigravity'de Command Palette acin (Ctrl+Shift+P)
# "Extensions: Check for Extension Updates" yazin ve calistirin
```

### Adim 2: Model Secimini Manuel Yapin

Claude Code CLI uzerinden model secimi:

```bash
# Claude Code CLI'da model ayari
claude config set model claude-opus-4-6

# Veya 1M context window ile
claude config set model claude-opus-4-6[1m]
```

### Adim 3: VS Code Settings ile Model Belirleyin

Antigravity'de `settings.json` dosyasina ekleyin:

```json
{
  "claude-code.model": "claude-opus-4-6"
}
```

### Adim 4: Antigravity Uyumluluk Duzeltmesi

Eger eklenti dogru yuklenmiyorsa:

1. Claude Code eklenti klasorunu bulun
2. `package.json` dosyasinda secondary sidebar tanimlarini kaldiriniz
3. Eklentiyi yeniden yukleyin

### Adim 5: Alternatif - Claude Code CLI Kullanin

Eger VS Code eklentisi calismiyorsa, terminal uzerinden Claude Code CLI kullanabilirsiniz:

```bash
# Claude Code CLI kurulumu
npm install -g @anthropic-ai/claude-code

# Opus 4.6 ile baslatma
claude --model claude-opus-4-6
```

## Desteklenen Model Listesi

| Model ID | Aciklama |
|---|---|
| `claude-opus-4-6` | En yetenekli model, karmasik gorevler icin |
| `claude-opus-4-6[1m]` | Opus 4.6 - 1M context window |
| `claude-sonnet-4-5` | Gunluk gorevler icin onerilen |
| `claude-sonnet-4-5[1m]` | Sonnet 4.5 - 1M context window |
| `claude-haiku-4-5` | Hizli ve verimli, basit gorevler icin |

## Ek Bilgiler

- Opus 4.6 model ID: `claude-opus-4-6`
- Tarihli model ID: `claude-opus-4-6-v1`
- 1M context window destegi beta olarak mevcuttur
- Terminal-Bench 2.0'da %65.4 ile en yuksek skoru elde etmistir

## Kaynaklar

- [Claude Opus 4.6 - Anthropic](https://www.anthropic.com)
- [GitHub Copilot icin Opus 4.6](https://github.blog/changelog/2026-02-05-claude-opus-4-6-is-now-generally-available-for-github-copilot/)
- [Claude Code Bilinen Sorunlar - Antigravity](https://github.com/anthropics/claude-code/issues/15552)
