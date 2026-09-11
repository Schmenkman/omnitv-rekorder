# OmniTV-Rekorder

Der Aufnahmediener für OmniTV-Haushalte: Er läuft auf dem Gerät, das ohnehin durchläuft —
NAS, Raspberry Pi oder ein alter PC —, nimmt Timer von der OmniTV-App entgegen und schreibt
die Sendungen zur Sendezeit auf die Platte. **Die App plant nur; aufgenommen wird hier** —
auch wenn Fernseher, Stick und Handy längst aus sind.

Kein Konto, kein fremder Server: App und Rekorder reden direkt miteinander, nur in deinem
Heimnetz. Die Timerliste liegt ausschließlich auf deinem Gerät.

Du brauchst **OmniTV ab Version 1.2.6.1** — dort ist die Rekorder-Anbindung eingebaut.
Diese Ablage enthält keinen Quelltext; die neueste Fassung liegt unter [Releases](../../releases).

---

## Installation — such dir dein Gerät aus

### Raspberry Pi oder Linux-Rechner: ein Befehl

```bash
curl -fsSL https://github.com/Schmenkman/omnitv-rekorder/releases/latest/download/installieren.sh | sudo sh
```

Das Skript fragt, **wohin die Aufnahmen sollen**, installiert bei Bedarf Java, richtet den
Dienst ein (startet nach jedem Neustart von selbst) und meldet, wenn alles läuft. Dasselbe
Kommando aktualisiert später auf neue Fassungen; `… | sudo sh -s -- --entfernen` räumt
wieder auf.

### Windows-PC: Doppelklick

[`OmniTV-Rekorder.zip`](../../releases/latest/download/OmniTV-Rekorder.zip) herunterladen,
entpacken, **`Installieren.bat` doppelklicken**. Sie fragt nach dem Aufnahme-Ordner, richtet
den Autostart bei der Anmeldung ein und startet den Rekorder gleich. (Fehlt Java, versucht
sie die Installation selbst über winget — sonst kurz Java 21 von
[adoptium.net](https://adoptium.net) holen.) Entfernen: `Installieren.bat entfernen`.

### NAS mit Docker (Synology, QNAP, Unraid, Portainer …): fertiges Abbild

Nichts herunterladen und nichts bauen — das fertige Abbild kommt von
`ghcr.io/schmenkman/omnitv-rekorder` und gibt es für Intel wie ARM. Diese
`docker-compose.yml` anlegen und darin **eine Zeile** auf deinen Aufnahme-Ordner zeigen lassen
(am besten derselbe, den deine SMB-Freigabe teilt):

```yaml
services:
  omnitv-rekorder:
    image: ghcr.io/schmenkman/omnitv-rekorder:latest
    container_name: omnitv-rekorder
    restart: unless-stopped
    # Host-Netz ist Pflicht: Die App sucht den Rekorder per Rundruf im Heimnetz,
    # und Rundrufe kommen durch eine Docker-Bridge nicht hindurch.
    network_mode: host
    environment:
      OMNITV_NAME: "NAS-Rekorder"
      # Optional: Wer Timer anlegen will, muss dieses Geheimnis mitschicken.
      # OMNITV_GEHEIMNIS: "bitte-aendern"
    volumes:
      - /volume1/video/OmniTV-Aufnahmen:/aufnahmen   # ← diese Zeile anpassen
      - ./daten:/daten
```

Dann:

```bash
docker compose up -d
```

Neue Fassungen holt `docker compose pull && docker compose up -d`.

**Synology mit Container Manager:** Projekt anlegen, obige Datei einfügen, als Netzwerk
**Host** wählen. **Unraid:** Container hinzufügen, Repository
`ghcr.io/schmenkman/omnitv-rekorder:latest`, Network Type **Host**, einen Pfad auf
`/aufnahmen` legen. Unraid meldet neue Fassungen danach von selbst im Docker-Reiter.

---

## In der App verbinden

**Einstellungen → Aufnahmen → OmniTV-Rekorder → „Im Heimnetz suchen"** — der Rekorder taucht
mit Namen auf, ein Druck übernimmt ihn (alternativ Adresse von Hand eingeben). Ab dann:

- **Timer stellen:** Sender gedrückt halten → „Timer stellen" → „Auf dem Rekorder" — oder im
  EPG eine Sendung gedrückt halten → „Auf dem Rekorder aufnehmen".
- **Stand ansehen und absagen:** im Reiter **Aufnahmen**, Abschnitt „Auf dem Rekorder".

## Feineinstellungen (Umgebungsvariablen)

| Variable | Vorgabe | Bedeutung |
|---|---|---|
| `OMNITV_AUFNAHMEN` | `/aufnahmen` | Wohin die Aufnahmen geschrieben werden |
| `OMNITV_DATEN` | `/daten` | Wo die Timerliste (`timer.json`) liegt |
| `OMNITV_PORT` | `48014` | Anschluss der Schnittstelle |
| `OMNITV_NAME` | Rechnername | Anzeigename im Suchergebnis der App |
| `OMNITV_GEHEIMNIS` | leer | Wenn gesetzt: Die App muss dasselbe Geheimnis eintragen |

## Was der Rekorder (noch) nicht kann — Stand 0.1

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
