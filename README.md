# OmniTV-Rekorder

Der Aufnahmediener für OmniTV-Haushalte: Er läuft auf dem Gerät, das ohnehin durchläuft —
NAS, Raspberry Pi oder ein alter PC —, nimmt Timer von der OmniTV-App entgegen und schreibt
die Sendungen zur Sendezeit auf die Platte. **Die App plant nur; aufgenommen wird hier** —
auch wenn Fernseher, Stick und Handy längst aus sind.

Kein Konto, kein fremder Server: Die App und der Rekorder reden direkt miteinander, nur in
deinem Heimnetz. Die Timerliste liegt ausschließlich auf deinem Gerät.

Diese Ablage enthält **keinen Quelltext** — sie stellt die fertigen Fassungen bereit.
Die jeweils neueste liegt unter [Releases](../../releases).

Du brauchst **OmniTV ab Version 1.2.6.1** — dort ist die Rekorder-Anbindung eingebaut.

---

## 1. Rekorder starten

### Raspberry Pi oder Linux-Rechner

Einmalig Java 21 installieren, dann das Release-Zip entpacken und starten:

```bash
sudo apt install openjdk-21-jre-headless
unzip OmniTV-Rekorder-*.zip && cd omnitv-rekorder
OMNITV_AUFNAHMEN=/pfad/zu/aufnahmen OMNITV_DATEN=/pfad/zu/daten ./bin/omnitv-rekorder
```

Damit er nach jedem Neustart von selbst läuft, als Dienst eintragen (Beispiel systemd):

```ini
# /etc/systemd/system/omnitv-rekorder.service
[Unit]
Description=OmniTV-Rekorder
After=network-online.target

[Service]
Environment=OMNITV_AUFNAHMEN=/pfad/zu/aufnahmen
Environment=OMNITV_DATEN=/pfad/zu/daten
ExecStart=/pfad/zu/omnitv-rekorder/bin/omnitv-rekorder
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Danach: `sudo systemctl enable --now omnitv-rekorder`

### Windows-PC

Java 21 installieren (z. B. [Eclipse Temurin](https://adoptium.net)), Zip entpacken, dann:

```bat
set OMNITV_AUFNAHMEN=D:\Aufnahmen
bin\omnitv-rekorder.bat
```

### NAS mit Docker (Synology, QNAP, Portainer …)

Im Release liegt ein kleines Docker-Paket (`omnitv-rekorder-docker.zip`): entpacken, in
`docker-compose.yml` den Aufnahme-Ordner anpassen (am besten derselbe Ordner, den deine
SMB-Freigabe teilt — dann sieht die App die fertigen Dateien sofort), dann:

```bash
docker compose up -d
```

*Ehrlicher Hinweis: Der Docker-Weg ist auf unseren Geräten noch nicht praxisgeprüft —
er folgt dem Standardmuster (Temurin-21-Basisbild plus dieses Zip). Rückmeldungen unter
[Issues](../../issues) sind willkommen.*

## 2. Einstellungen (Umgebungsvariablen)

| Variable | Vorgabe | Bedeutung |
|---|---|---|
| `OMNITV_AUFNAHMEN` | `/aufnahmen` | Wohin die Aufnahmen geschrieben werden |
| `OMNITV_DATEN` | `/daten` | Wo die Timerliste (`timer.json`) liegt |
| `OMNITV_PORT` | `48014` | Anschluss der Schnittstelle |
| `OMNITV_NAME` | Rechnername | Anzeigename im Suchergebnis der App |
| `OMNITV_GEHEIMNIS` | leer | Wenn gesetzt: Die App muss dasselbe Geheimnis eintragen |

## 3. In der App verbinden

In OmniTV: **Einstellungen → Aufnahmen → OmniTV-Rekorder → „Im Heimnetz suchen"** — der
Rekorder taucht mit Namen auf, ein Druck übernimmt ihn (alternativ die Adresse von Hand
eingeben). Ab dann:

- **Timer stellen:** Sender gedrückt halten → „Timer stellen" → „Auf dem Rekorder" — oder im
  EPG eine Sendung gedrückt halten → „Auf dem Rekorder aufnehmen".
- **Stand ansehen und absagen:** im Reiter **Aufnahmen**, Abschnitt „Auf dem Rekorder".

## 4. Was der Rekorder (noch) nicht kann — Stand 0.1

- **Nur Direkt-Ströme** (die üblichen Xtream-`.ts`-Adressen). Sender, die ausschließlich
  HLS (`m3u8`) liefern, lehnt er mit klarer Meldung ab, statt Datenmüll aufzuzeichnen.
- Die Uhrzeit kommt vom Gerät, auf dem er läuft — geht dessen Uhr falsch, nimmt er daneben
  auf. Die App warnt, wenn die Uhr mehr als zwei Minuten abweicht.
- Reißt die Verbindung während einer Sendung, setzt er innerhalb des Zeitfensters selbst
  wieder an; war er zur Sendezeit ganz aus, meldet er den Timer ehrlich als „verpasst".
- Serientimer und automatisches Aufräumen alter Aufnahmen sind Ausbaustufen.

## Rechtliches

Der OmniTV-Rekorder ist urheberrechtlich geschützt. Copyright © 2026 Schmenkman — alle
Rechte vorbehalten; Einzelheiten in der [LICENSE](LICENSE). Aufnahmen sind Privatkopien für
den eigenen Gebrauch (§ 53 UrhG) und dürfen nicht weitergegeben oder veröffentlicht werden.
