# BlokDuck

**Secure, reliable, and reversible data obfuscation.**

BlokDuck detects sensitive information (PII, credentials, financial data) in your
documents and images, then redacts or reversibly encodes it — entirely on your own
machine. No cloud, no data egress. Redaction is key-based: only a holder of the
private key can recover the original values.

## Supported platforms

Prebuilt binaries are published for each release:

| Platform | Architecture | Artifact |
| --- | --- | --- |
| macOS | Apple Silicon (arm64) | `rde_obfuscator-aarch64-apple-darwin-v<version>.tar.gz` |
| Linux | x86_64 | `rde_obfuscator-x86_64-unknown-linux-gnu-v<version>.tar.gz` |

Each archive contains two binaries:

- **`rde_cli`** — the command-line tool for redaction and encoding (what most users want).
- **`rde_obfuscator`** — the HTTP service, for integrating obfuscation into a stack.

Windows binaries are not yet published.

## Installation

Download the archive for your platform from the
[latest release](https://github.com/Quantafin-Lab/blokduck/releases/latest),
verify it, and extract:

```bash
# macOS (Apple Silicon)
curl -fsSLO https://github.com/Quantafin-Lab/blokduck/releases/latest/download/rde_obfuscator-aarch64-apple-darwin-v<version>.tar.gz

# Linux (x86_64)
curl -fsSLO https://github.com/Quantafin-Lab/blokduck/releases/latest/download/rde_obfuscator-x86_64-unknown-linux-gnu-v<version>.tar.gz

tar xzf rde_obfuscator-*.tar.gz
sudo mv rde_obfuscator-*/rde_cli /usr/local/bin/
```

### Verify your download

Every release includes a `checksums.sha256` file. After downloading, confirm the
archive is intact:

```bash
shasum -a 256 -c checksums.sha256
```

## Quick start

```bash
# Redact sensitive fields in a document → public report + private key report
rde_cli redact intake.csv --out intake.public.json --key intake.private.json

# Redact an image (requires the `tesseract` binary on PATH)
rde_cli redact-image scan.png --out-image scan.redacted.png --key scan.private.json

# Reversibly encode any file, then recover it with the key
rde_cli encode secret.bin
rde_cli decode secret.bin.cipher.json --key secret.bin.rdekey.json --out restored.bin
```

## Command reference

| Command | Purpose |
| --- | --- |
| `encode <input>` | RDE-encode the raw bytes of any file. Emits a public ciphertext and a separate secret key file. |
| `decode <cipher> --key <keyfile>` | Recover the original bytes from a ciphertext + its key. |
| `redact <input>` | Detect sensitive fields in a document, tokenize them, and write a public report plus a private (key) report. |
| `redact-image <image>` | OCR an image, detect sensitive text and table cells, paint redaction boxes, and write public + private reports. |
| `decode-redaction <private.json>` | Recover field → original-text mappings from a private report. |

Common options: `--out <path>`, `--key <path>`, `--threshold N` (detection score,
default 70), `--m-rows N` (default 128), `--wiener`. Run `rde_cli` with no arguments
for full usage.

## Supported file formats

- **Documents:** PDF, TXT, DOCX, CSV, TSV, XLSX, XLS, XLSM, ODS, JSON, XML
- **Images:** PNG, JPEG

## Troubleshooting

- **`command not found`** — ensure the install location (e.g. `/usr/local/bin`) is on
  your `PATH`, or invoke the binary by its full path.
- **Permission denied** — mark the binary executable: `chmod +x rde_cli`.
- **macOS "cannot be opened"** — remove the quarantine attribute:
  `xattr -d com.apple.quarantine rde_cli`.
- **`redact-image` errors** — install Tesseract OCR (`brew install tesseract` /
  `apt-get install tesseract-ocr`) and ensure `tesseract` is on `PATH`.
