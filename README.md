# BlokDuck

**Secure, reliable, and reversible data obfuscation.**

BlokDuck detects sensitive information (PII, credentials, financial data) in your
documents and images, then redacts or reversibly encodes it — entirely on your own
machine. No cloud, no data egress. Redaction is key-based: only a holder of the
private key can recover the original values.

BlokDuck runs inside a Docker container with deliberately limited host access
(read-only input, write-only output, localhost-only networking), so the redaction
engine never touches anything beyond the folders you mount.

## Installation

### Docker (recommended — all platforms)

The official image is multi-architecture (`linux/amd64` + `linux/arm64`) and runs
identically on Windows, macOS, and Linux:

```bash
docker pull ghcr.io/quantafin-lab/blokduck:latest

docker run -d --name blokduck -p 8787:8787 \
  -v "$HOME/Blokduck/input:/input:ro" \
  -v "$HOME/Blokduck/output:/output" \
  -v "$HOME/Blokduck/config.json:/etc/blokduck/config.json:ro" \
  -e RDE_PROFILE=default \
  ghcr.io/quantafin-lab/blokduck:latest
```

Then open http://localhost:8787 in your browser. The web UI (redaction, profiles,
and audit artifacts) is baked into the image; the server binds loopback only and
makes no outbound calls.

Pin a specific release instead of `latest` by using the version tag, e.g.
`ghcr.io/quantafin-lab/blokduck:0.4.10`.

### One-line install (Docker-first)

Each installer provisions Docker (if missing), creates folders + `config.json`,
pulls the image, and launches the container on port 8787.

**Windows (PowerShell):**

```powershell
iwr https://github.com/Quantafin-Lab/blokduck/releases/latest/download/install-windows.ps1 -OutFile install-windows.ps1; .\install-windows.ps1
```

**macOS:**

```bash
curl -fsSL https://github.com/Quantafin-Lab/blokduck/releases/latest/download/install-mac.sh | bash
```

**Linux:**

```bash
curl -fsSL https://github.com/Quantafin-Lab/blokduck/releases/latest/download/install-linux.sh | bash
```

### Platform installers

Each release ships the install scripts above plus a macOS menu-bar app:

| Platform | Asset | Notes |
| --- | --- | --- |
| Windows | `install-windows.ps1` | PowerShell script — checks for Docker Desktop, writes `config.json`, launches the container. |
| macOS | `install-mac.sh` | Bash script — provisions Docker, launches the container. |
| Linux | `install-linux.sh` | Bash script — installs Docker if missing, launches the container. |
| macOS menu app | `BlokduckMenuApp-Mac-Arm.tar.gz` / `BlokduckMenuApp-Mac-x86.tar.gz` | Extract and drop `BlokduckMenu.app` into Applications. The menu-bar icon edits `config.json` and starts/stops the container. |

All artifacts are attached to the
[latest release](https://github.com/Quantafin-Lab/blokduck/releases/latest).

## Using the CLI

The container bundles `rde_cli`. Run it inside the running container via
`bin/docker_cli.sh` (see the release notes) or directly:

```bash
docker exec -it blokduck rde_cli --help
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

- **Docker not installed** — the Windows and Linux installers guide you through
  installing Docker. Manually: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
  (Windows/macOS) or [Docker Engine](https://docs.docker.com/engine/install/) (Linux).
- **Port 8787 already in use** — run the container on another host port with
  `-p 8080:8787`, or set `port` in `config.json`.
- **`redact-image` errors** — install Tesseract OCR (`brew install tesseract` /
  `apt-get install tesseract-ocr`) and ensure `tesseract` is on `PATH`.
