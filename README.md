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

### NAS mit Docker (ZimaOS, CasaOS, Synology, QNAP, UGREEN, TrueNAS, Unraid …): fertiges Abbild

Gebaut wird nichts — das fertige Abbild kommt von `ghcr.io/schmenkman/omnitv-rekorder` und gibt
es für Intel wie ARM. Die passende Vorlage für dein NAS liegt in
[`omnitv-rekorder-docker.zip`](../../releases/latest/download/omnitv-rekorder-docker.zip):

| Datei | Für |
|---|---|
| `omnitv-rekorder-zimaos.yml` | ZimaOS und CasaOS |
| `docker-compose.yml` | Synology, QNAP, UGREEN, TrueNAS, Portainer, Docker ohne Oberfläche |
| `my-OmniTV-Rekorder.xml` | Unraid |

Das gilt für alle:

- **Host-Netz ist Pflicht** (`network_mode: host`, keine `ports:`-Zeilen): Die App sucht den
  Rekorder per Rundruf im Heimnetz, und Rundrufe kommen durch eine Docker-Bridge nicht
  hindurch. Die Vorlagen sind schon so eingestellt.
- **Den Aufnahme-Ordner vorher in der NAS-Oberfläche anlegen**, am besten in der Freigabe, über
  die du auch sonst Videos schaust. Legt Docker ihn selbst an, gehört er root, und über die
  Freigabe lässt sich darin womöglich nichts löschen.
- **`TZ`** bestimmt die Uhrzeit im Dateinamen („Tatort 2026-11-22 2015.ts“). Ohne Angabe gilt
  `Europe/Berlin`.
- **`PUID`/`PGID`:** Damit schreibt der Rekorder als dein NAS-Nutzer statt als root, und die
  Aufnahmen gehören dir. Die Zahlen nennt `id <name>` per SSH; ohne die beiden läuft er als root.
  `UMASK=002` heißt: Die Gruppe darf Aufnahmen ändern und löschen.

**ZimaOS:** „+“ über den App-Symbolen → „Install a customized app“ → Import-Symbol oben rechts →
Reiter „Docker Compose“ → Inhalt von `omnitv-rekorder-zimaos.yml` einfügen → Senden. Den
Aufnahme-Ordner prüfen: `/DATA` liegt auf der Systemplatte, für viele Aufnahmen besser ein Ordner
auf der großen Platte (`/media/<Platte>/…`). Danach die Kachel antippen: Erscheint
`{"name":"ZimaOS-Rekorder",…}`, läuft der Rekorder im Host-Netz. Lädt die Seite nicht, hat der
Import das Host-Netz verworfen (bekannt aus ZimaOS 1.7.0) — dann in den Einstellungen der App im
YAML-Editor `networks:` entfernen und `network_mode: host` eintragen. Scheitern Aufnahmen mit
„Aufnahmedatei nicht anlegbar“ (etwa auf einer NTFS-Platte), `PUID` und `PGID` auf 0 setzen.
**CasaOS:** App-Store → „Custom Install“ → Import.

**Synology, QNAP, UGREEN, TrueNAS:** In `docker-compose.yml` die beiden Pfade anpassen, dann
einfügen — Synology: Container Manager → Projekt → Erstellen; QNAP: Container Station →
Anwendungen → Erstellen; UGREEN: Docker → Projekt → Erstellen; TrueNAS: Apps → Discover Apps →
⋮ → Install via YAML. Ohne Oberfläche: `docker compose up -d`. Synology legt fehlende Ordner
nicht an — dort auch den Daten-Ordner vorher in der File Station anlegen.

**Unraid:** `my-OmniTV-Rekorder.xml` nach `/boot/config/plugins/dockerMan/templates-user/` legen
(z. B. über die Freigabe „flash“), dann Docker → Add Container → bei „Template“
„OmniTV-Rekorder“ wählen → Aufnahme-Ordner prüfen → Apply.

**Neue Fassungen:** `docker compose pull && docker compose up -d`, in den NAS-Oberflächen
„Aktualisieren“ bzw. „Neu erstellen“. Unraid meldet neue Fassungen von selbst im Docker-Reiter.

---

## In der App verbinden

**Einstellungen → Aufnahmen → OmniTV-Rekorder → „Im Heimnetz suchen“** — der Rekorder taucht
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
| `TZ` | `Europe/Berlin` | Zeitzone für die Uhrzeit in Dateinamen und Log |
| `PUID` / `PGID` | leer (root) | Nur Docker: Nutzer und Gruppe, denen die Aufnahmen gehören |
| `UMASK` | `022` | Nur Docker: Rechte neuer Dateien (`002` = Gruppe darf löschen) |

## Was der Rekorder (noch) nicht kann — Stand 0.4

- **Nur Direkt-Ströme** (die üblichen Xtream-`.ts`-Adressen). Sender, die ausschließlich
  HLS (`m3u8`) liefern, lehnt er mit klarer Meldung ab, statt Datenmüll aufzuzeichnen.
- Die Uhrzeit kommt vom Gerät, auf dem er läuft — geht dessen Uhr falsch, nimmt er daneben
  auf. Die App warnt, wenn die Uhr mehr als zwei Minuten abweicht.
- Reißt die Verbindung während einer Sendung, setzt er innerhalb des Zeitfensters selbst
  wieder an; war er zur Sendezeit ganz aus, meldet er den Timer ehrlich als „verpasst“.
- Serientimer und automatisches Aufräumen alter Aufnahmen sind Ausbaustufen.

## Rechtliches

Der OmniTV-Rekorder ist urheberrechtlich geschützt. Copyright © 2026 Schmenkman — alle
Rechte vorbehalten; Einzelheiten in der [LICENSE](LICENSE). Aufnahmen sind Privatkopien für
den eigenen Gebrauch (§ 53 UrhG) und dürfen nicht weitergegeben oder veröffentlicht werden.
