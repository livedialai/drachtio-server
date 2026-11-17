# Build notes — drachtio-server for the warm-transfer stack

Fork von `drachtio/drachtio-server` (MIT). Zweck: Unabhaengigkeit von einem
einzelnen Upstream-Account und reproduzierbarer Build fuer unsere Kiste.

Gebaut und im Betrieb bewaehrt am 17.11.2026, Commit `cd1a1d0`, Debian 12
(bookworm), x86_64. Ergebnis: `drachtio --version` → `cd1a1d0`.

## Build (Debian 12)

```bash
apt-get install -y git build-essential autoconf automake libtool libtool-bin \
  pkg-config libssl-dev libcurl4-openssl-dev libpcre3-dev libhiredis-dev \
  cmake libboost-all-dev zlib1g-dev

git clone https://github.com/drachtio/drachtio-server.git
cd drachtio-server

./autogen.sh
mkdir -p build && cd build
../configure CPPFLAGS='-DNDEBUG'
make -j"$(nproc)"
make install          # -> /usr/local/bin/drachtio
strip /usr/local/bin/drachtio     # 62 MB -> 3.2 MB
```

### Stolpersteine (real aufgetreten)

| Problem | Ursache | Loesung |
|---|---|---|
| `bootstrap: libtool not found. You need libtool version 1.5.14 or newer` | Debian trennt das Paket: `libtool` liefert nur `libtoolize` | **`libtool-bin`** zusaetzlich installieren |
| `g++: fatal error: Killed signal terminated program cc1plus` | 2 GB RAM, Boost-Templates | Swap anlegen (2 GB) und `make -j1` |
| Binary startet nicht, verlangt `JAMBONES_MYSQL_*` | das ist der **jambonz**-Fork, nicht Upstream | Upstream bauen oder das Upstream-Binary nehmen |
| `Cannot open /etc/drachtio.conf.xml` | Upstream erwartet die Konfigurationsdatei | Datei anlegen, siehe `warmtransfer/drachtio.conf.xml` |

## Betrieb

```bash
install -m 0644 warmtransfer/drachtio.conf.xml /etc/drachtio.conf.xml
install -m 0644 warmtransfer/drachtio.service   /etc/systemd/system/
systemctl daemon-reload && systemctl enable --now drachtio
```

Wichtig: **der SIP-Server selbst muss den Trunk registrieren.** Bei easybell
kommen eingehende INVITEs an den Port, von dem aus registriert wurde — ein
separater Registrar auf einem anderen Port bekommt die Anrufe und der
SIP-Eingang bleibt stumm.

## Bezug

- Upstream: https://github.com/drachtio/drachtio-server (MIT)
- Fork: https://github.com/livedialai/drachtio-server
- Spiegel: https://codeberg.org/weserai/drachtio-server
- Fertiges Binary: Release `cd1a1d0-x86_64` in diesem Fork
