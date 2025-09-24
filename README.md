# Kismet Build Guide – Linux Mint i386 (Trixie)

Da es für i386/Trixie keine fertigen Pakete gibt, wird Kismet hier aus dem Quellcode kompiliert.  
Dieses Dokument ist vollständig, copy-paste-fähig und für GitHub geeignet.

---

## Inhalt

- Systeminformationen prüfen
- Schneller Weg: Build-Script
- Inhalt des Build-Scripts
- Manuelle Schritte
- Troubleshooting
- Verifikation

---

## Systeminformationen prüfen

```bash
lsb_release -a
uname -m
lsb_release -cs    # Ausgabe: Codename (z. B. trixie)
```

---

## Schneller Weg: Build-Script

Speichere folgendes Script als `build_kismet_i386.sh`, mache es ausführbar und starte es:

```bash
chmod +x build_kismet_i386.sh
./build_kismet_i386.sh
```

---

## Inhalt von `build_kismet_i386.sh`

```bash
#!/bin/bash
set -euo pipefail

echo "== Kismet Build Script (i386) =="

CODENAME="$(lsb_release -cs 2>/dev/null || true)"
ARCH="$(uname -m)"
echo "Detected: codename='$CODENAME' arch='$ARCH'"

read -p "Continue? (y/n) " yn
if [[ "$yn" != "y" ]]; then
  exit 1
fi

sudo apt update
sudo apt upgrade -y
sudo apt install -y build-essential git pkg-config curl gpg ca-certificates

sudo apt install -y \
  libpcap-dev libnl-3-dev libnl-genl-3-dev libcap-dev \
  libsqlite3-dev libprotobuf-dev protobuf-compiler \
  python3 python3-dev python3-setuptools python3-requests \
  python3-protobuf python3-websockets python3-numpy \
  python3-serial python3-usb libusb-1.0-0-dev librtlsdr-dev \
  libmosquitto-dev libsensors-dev || true

if sudo apt install -y libwebsockets-dev; then
  echo "libwebsockets-dev installed"
else
  echo "libwebsockets-dev missing, will try without"
fi

TMPDIR=$(mktemp -d)
cd "$TMPDIR"
git clone https://www.kismetwireless.net/git/kismet.git
cd kismet

if ldconfig -p | grep -q libwebsockets; then
  ./configure
else
  ./configure --disable-libwebsockets
fi

JOBS=$(nproc)
if [ "$(free -m | awk '/^Mem:/ {print $2}')" -lt 8000 ]; then
  JOBS=2
fi
make -j"$JOBS"

sudo make suidinstall
sudo usermod -aG kismet "$USER"

echo "Build finished."
echo "Reboot or log out/in, then run 'kismet' and open http://localhost:2501"
```

---

## Manuelle Installation

### Abhängigkeiten installieren

```bash
sudo apt update
sudo apt install -y build-essential git pkg-config curl gpg ca-certificates \
  libpcap-dev libnl-3-dev libnl-genl-3-dev libcap-dev \
  libsqlite3-dev libprotobuf-dev protobuf-compiler \
  python3 python3-dev python3-setuptools python3-requests \
  python3-protobuf python3-websockets python3-numpy \
  python3-serial python3-usb libusb-1.0-0-dev librtlsdr-dev \
  libmosquitto-dev libsensors-dev
```

Optional (falls verfügbar):

```bash
sudo apt install -y libwebsockets-dev
```

---

### Quellcode holen

```bash
cd /tmp
git clone https://www.kismetwireless.net/git/kismet.git
cd kismet
```

---

### Konfigurieren

```bash
./configure
# Falls libwebsockets fehlt:
# ./configure --disable-libwebsockets
```

---

### Kompilieren

```bash
make version
make -j$(nproc)
```

Bei wenig RAM ggf.:

```bash
make -j2
```

---

### Installieren

```bash
sudo make suidinstall
sudo usermod -aG kismet $USER
```

Re-login oder Reboot notwendig.

---

## Starten & Verifizieren

```bash
kismet
```

Im Browser öffnen:

```
http://localhost:2501
```

---

## Troubleshooting

- **Fehlende Pakete**: Ausgabe von `./configure` lesen und mit `apt install <paket>` nachinstallieren.
- **OOM (Out of Memory)**: `make -j1` nutzen oder temporären Swap anlegen:
  ```bash
  sudo fallocate -l 4G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  ```
- **libwebsockets nicht verfügbar**: entweder aus Source bauen oder `--disable-libwebsockets` nutzen.
- **Capture funktioniert nicht**: Stelle sicher, dass `suidinstall` genutzt wurde und dein Benutzer in der Gruppe `kismet` ist.
- **Interface kein Monitor Mode**: prüfen mit `iw dev`, ggf. Treiberproblem.

---

## Fertig!

Checkliste:

- [x] Dependencies installiert  
- [x] Kismet kompiliert (`make`)  
- [x] Installiert (`sudo make suidinstall`)  
- [x] Benutzer zur Gruppe `kismet` hinzugefügt  
- [x] Start mit `kismet` → Web UI unter `http://localhost:2501`

---
