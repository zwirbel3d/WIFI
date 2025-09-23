# WIFI

# Kismet & LinSSID auf Linux Mint (kostenlos) — Install & Run

> Linux Mint ist Ubuntu-basiert. Alle Befehle unten sind für `apt`. Von oben nach unten ausführen.

---

## 1) System aktualisieren
sudo apt update
sudo apt upgrade -y

---

## 2) Basis-Tools (Treiber/CLI)
sudo apt install -y software-properties-common build-essential \
  wireless-tools iw wpasupplicant net-tools curl

---

## 3) Kismet installieren & starten
# Installation (Repo-Version)
sudo apt install -y kismet

# Start (Web-UI auf Port 2501)
sudo kismet
# Browser öffnen: http://localhost:2501

---

## 4) LinSSID installieren & starten
# Versuch mit Mint/Ubuntu-Repos
sudo apt install -y linssid

# Falls Paket nicht gefunden/veraltet: PPA hinzufügen
sudo add-apt-repository -y ppa:wseverin/ppa
sudo apt update
sudo apt install -y linssid

# Starten
linssid

---

## 5) LinSSID Troubleshooting (fehlende Abhängigkeiten)

# Automatisch reparieren
sudo apt --fix-broken install -y
sudo apt update
sudo apt install -f -y

# Häufig nötige Libs
sudo apt install -y libqt5core5a libqt5gui5 libqt5widgets5 \
  libqt5network5 libqt5svg5 qtmultimedia5-dev \
  libqwt-qt5-dev libpcap0.8 libpcap0.8-dev \
  libnl-3-dev libnl-genl-3-dev pkg-config

# Falls nötig: Build aus Source
sudo apt install -y git qtbase5-dev qtmultimedia5-dev qmake make
git clone https://git.code.sf.net/p/linssid/linssid linssid
cd linssid
qmake
make -j"$(nproc)"
sudo make install

---

## 6) Schnelltests
# Interface prüfen
iw dev

# Schnellliste (Signal in %)
nmcli -f SSID,BSSID,SIGNAL,CHAN dev wifi list

# Detaillierter Scan mit dBm (Interface anpassen)
sudo iw dev wlan0 scan | egrep 'BSS|SSID|signal' -B2

---

## 7) Monitor-Mode + CSV-Logging (aircrack-ng)
sudo apt install -y aircrack-ng
sudo airmon-ng start wlan0
sudo airodump-ng -w capture --output-format csv wlan0mon
# Ergebnis: capture-01.csv

---

## 8) Hersteller/OUI-Lookup
# Offline
curl -O https://standards-oui.ieee.org/oui.txt
grep -i 'AA-BB-CC' oui.txt

# Online
curl -s "https://api.macvendors.com/AA:BB:CC:11:22:33"

---

## 9) Typische Fehlerbehebungen
sudo apt --fix-broken install -y
sudo apt update
sudo add-apt-repository -y universe
sudo apt update
sudo apt install -y linssid
linssid
