# Synology Photos – Externer Zugriff mit DDNS (Projekt-Dokumentation)

## 1. Ziel und Kontext

Dieses Projekt dokumentiert die Herstellung eines **stabilen und performanten externen Zugriffs** auf Synology Photos von einem Pixel 9 Pro mit GrapheneOS aus.

Schwerpunkte:

* Zugang **ohne QuickConnect** (direkt per DDNS/Https)
* **Schnellere Video-Wiedergabe** unterwegs als mit QuickConnect-Relay
* Klare, reproduzierbare Netzwerk-Konfiguration (FritzBox → MikroTik → NAS)
* Vorbereitung für eine spätere Konsolidierung über **Nginx Proxy Manager (NPM)** auf Port 443

## 2. Umgebung

### 2.1 Hardware / Komponenten

* **NAS**: Synology (Modell nicht entscheidend für diese Doku)
* **Router 1 (Upstream)**: AVM FritzBox (öffentliches Internet-Gateway)
* **Router 2 (Haupt-Router)**: MikroTik hAP ax3 (LAN-Gateway, NAT, Firewall)
* **Docker-/NPM-Host**: HP EliteDesk (Ubuntu, Docker, Nginx Proxy Manager)
* **Client mobil**: Pixel 9 Pro mit GrapheneOS

### 2.2 Netzwerktopologie (vereinfacht)

* FritzBox-LAN: z. B. `192.168.178.0/24`

  * MikroTik-WAN: z. B. `192.168.178.x`
* MikroTik-LAN: `192.168.88.0/24`

  * NAS: `192.168.88.32`
  * EliteDesk (NPM): `192.168.88.106`

### 2.3 Dienste / DNS

* Öffentlicher Zugriff auf Heimnetz über **DynDNS** (z. B. MikroTik-DDNS oder externen Provider)
* Synology-DDNS (z. B. `deinname.synology.me`) für direkten Zugriff auf DSM/Photos
* Eigene Domain: `mydomain.de` mit mehreren Subdomains (z. B. für Paperless, Vikunja, zukünftig Photos)

## 3. Ausgangsproblem

### 3.1 Symptome

* Synology Photos-App auf dem Pixel 9 Pro:

  * Anmeldung über **QuickConnect-ID** funktioniert
  * Anmeldung über **DDNS-Hostname** (z. B. Synology-DDNS) schlägt fehl
* Fehlermeldung in der App:

  * sinngemäß: *„Netzwerkverbindung oder NAS-IP überprüfen“*
* Beim Aufruf von `https://<synology-ddns>:5001` aus dem mobilen Internet:

  * Browser-Fehler: `ERR_CONNECTION_RESET`
* DNS-Einträge wirken grundsätzlich korrekt (DDNS zeigt auf die öffentliche IP), dennoch kein Https-Zugriff auf Port 5001 von außen möglich.

### 3.2 QuickConnect vs. DDNS

* **QuickConnect**

  * Vorteil: funktioniert oft „einfach so“, ohne Portfreigaben
  * Nachteil: Video-Streams sind spürbar langsam, da der Datenverkehr über Synology-Relay-Server läuft
* **Direkter DDNS-Zugriff (Port 5001)**

  * Vorteil: direkter Weg vom Client zum NAS → deutlich bessere Performance, keine Relay-Limits
  * Voraussetzung: korrekt konfigurierte Portweiterleitungen (FritzBox + MikroTik)

Ziel war deshalb, den **direkten DDNS-Zugriff** für Synology Photos wiederherzustellen und QuickConnect nur noch als Fallback zu behalten.

## 4. Analyse des Verbindungsproblems

### 4.1 Systematische Checks

Zur Eingrenzung wurden folgende Tests durchgeführt (Client jeweils: Pixel 9 Pro im Mobilnetz, WLAN aus):

1. **Direkter DSM-Zugriff per Synology-DDNS:**

   * `https://<synology-ddns>:5001`
   * Ergebnis: `ERR_CONNECTION_RESET`
2. **Geplante Subdomain für Photos über NPM:**

   * `https://photos.mydomain.de`
   * Ergebnis: ebenfalls Fehler, später konkret `ERR_SSL_UNRECOGNIZED_NAME_ALERT`
3. **Zertifikatsstatus auf der Synology:**

   * Standard-Zertifikat für `<synology-ddns>` ist korrekt als Systemstandard gesetzt
   * Direktes Zuordnen zu „Synology Photos“ im DSM ist nicht nötig, da Photos über DSM-/Reverse-Proxy läuft
4. **Portfreigaben auf der FritzBox:**

   * Port 443 → MikroTik (für NPM) ist vorhanden
   * **Port 5001** → MikroTik war **nicht** freigegeben
5. **MikroTik-Firewall/NAT-Regeln:**

   * `srcnat` Masquerade-Regel von FritzBox-Netz zum MikroTik-LAN vorhanden
   * `dstnat`-Regeln für NPM auf Ports 80 und 443 vorhanden
   * **Keine `dstnat`-Regel für Port 5001 → NAS**

### 4.2 Schlussfolgerung

Die Analyse ergab:

* DDNS zeigte korrekt auf die öffentliche IP
* Die FritzBox leitete **Port 5001 nicht zum MikroTik** weiter
* Der MikroTik leitete **Port 5001 nicht zum NAS** weiter

Damit konnte der Verbindungsaufbau von außen auf `:5001` gar nicht funktionieren → `ERR_CONNECTION_RESET` war die logische Folge.

QuickConnect funktionierte trotzdem, da es die Portfreigaben umgeht und einen Tunnel über Synology-Server verwendet.

## 5. Lösung Phase 1 – Direkter DDNS-Zugriff über Port 5001

Ziel der ersten Phase: **schnelle und robuste Lösung**, die den direkten Zugriff auf DSM/Photos wieder ermöglicht, ohne QuickConnect.

### 5.1 Portfreigabe auf der FritzBox

In der FritzBox wurde eine neue Portfreigabe angelegt:

* **Port extern**: `5001`
* **Protokoll**: TCP
* **Zielgerät**: MikroTik (IP im FritzBox-LAN, z. B. `192.168.178.x`)
* **Port intern**: `5001`

Damit wird externer Traffic auf `:5001` an den MikroTik weitergereicht.

### 5.2 NAT-Regel auf dem MikroTik

Auf dem MikroTik wurde eine `dstnat`-Regel ergänzt, um Port 5001 an den NAS im MikroTik-LAN weiterzuleiten:

```shell
/ip firewall nat
add chain=dstnat \
    in-interface=<WAN-Interface-zur-FritzBox> \
    protocol=tcp dst-port=5001 \
    action=dst-nat to-addresses=192.168.88.32 to-ports=5001 \
    comment="DSM HTTPS 5001 -> NAS"
```

* `<WAN-Interface-zur-FritzBox>` ist das Interface mit `status=bound` im DHCP-Client (z. B. `ether1`)
* `192.168.88.32` ist die NAS-IP im MikroTik-LAN

### 5.3 Verifikation – DSM im Browser

Test vom Pixel 9 Pro (Mobilnetz, WLAN aus):

* Aufruf: `https://<synology-ddns>:5001`
* Ergebnis: DSM-Loginseite wird angezeigt

Damit war der End-to-End-Weg:

```text
Pixel (mobil) → Internet → FritzBox:5001 → MikroTik:5001 → NAS:5001 → DSM
```

wiederhergestellt.

### 5.4 Synology Photos-App neu konfigurieren

In der Synology Photos-App:

* Neue Verbindung anlegen
* **Zugriffstyp**: NAS / Per Hostname/IP
* **Adresse/Hostname**: `<synology-ddns>`
* **Port**: `5001`
* **HTTPS**: aktiviert
* DSM-Benutzername und Passwort hinterlegen

Ergebnis:

* Die Anmeldung in der Photos-App funktionierte sofort
* QuickConnect ist damit nicht mehr erforderlich, bleibt aber als Fallback verfügbar

## 6. Sicherheitseinschätzung

### 6.1 Offene Ports

Nach der Umsetzung von Phase 1 sind von außen relevante Ports:

* **443 → MikroTik → NPM → (weitere Dienste)**
* **5001 → MikroTik → NAS (DSM/Photos)**

Beide Ports sind:

* TLS-verschlüsselt (HTTPS)
* nur nach erfolgreicher DSM-Authentifizierung nutzbar

Die reine Anzahl offener HTTPS-Ports (1 vs. 2) ist im Vergleich zu anderen Faktoren (starke Passwörter, 2FA, regelmäßige Updates, minimale Berechtigungen) sicherheitlich eher sekundär.

Die aktuelle Konfiguration kann als **sicher und praxisnah** eingestuft werden, insbesondere im Vergleich zum ursprünglichen Zustand ohne funktionierenden DDNS-Zugriff.

### 6.2 Perspektive „alle Dienste hinter 443“

Langfristig ist es dennoch sinnvoll, so viele Dienste wie möglich über einen zentralen Reverse Proxy (NPM) auf **Port 443** zu bündeln:

* Einheitlicher TLS-Termination-Point (Let’s Encrypt, HSTS etc.)
* Klare Domänenstruktur (`paperless.mydomain.de`, `vikunja.mydomain.de`, `photos.mydomain.de`, …)
* Nur ein Port nach außen nötig

Aufgrund spezieller TLS-/SNI-Verhaltensweisen auf Port 443 des EliteDesk (TLS-Handshake bricht mit `unrecognized name` ab) wird dieses Thema in einer separaten „Phase 2“ behandelt.

## 7. Performance – Warum war QuickConnect langsam und DDNS schneller?

### 7.1 QuickConnect – Relay statt Direktverbindung

QuickConnect arbeitet typischerweise so:

```text
Client → Synology-Server → NAS → Synology-Server → Client
```

Folgen:

* Zusätzlicher Hop über Synology-Infrastruktur
* Bandbreite oft begrenzt (Relay-Limits)
* Höhere Latenz → Videos starten langsamer, Spulen verzögert

### 7.2 Direkter DDNS-Zugriff

Beim jetzigen DDNS-Setup mit Port 5001:

```text
Client → Internet → FritzBox → MikroTik → NAS (DSM/Photos)
```

Die maximal mögliche Geschwindigkeit wird im Wesentlichen begrenzt durch:

* Upload-Bandbreite des Heimanschlusses
* Mobile Datenverbindung des Geräts (Signalqualität, Bandbreite)
* Leistungsfähigkeit des NAS (Transkodierung von Video, CPU/GPU-Last)
* Eventuelle Hintergrundlast (andere Dienste, Backups etc.)

### 7.3 Gründe, warum ein Video trotzdem „nicht lädt“

Auch mit direktem DDNS-Zugriff kann es zu langsamer Video-Wiedergabe kommen. Typische Ursachen:

1. **Upload-Bandbreite des Heimanschlusses** ist zu niedrig oder ausgelastet

   * z. B. 10 Mbit/s Upload bei 1080p/4K-Videos
   * parallel laufende Uploads (Backups, andere Streams)

2. **On-the-fly-Transkodierung durch Synology Photos**

   * z. B. Originalvideo als 4K HEVC/H.265
   * NAS muss die Datei live in ein kompatibles Format/Bitrate umrechnen
   * bei schwächerer CPU kann das zu starkem Delay führen

3. **Mobile Netzqualität**

   * instabile LTE/5G-Verbindung
   * Drosselung/Tarif-Limits

4. **Speichermedien-/NAS-Last**

   * gleichzeitige IO-Last durch andere Dienste (Docker-Container, Backups, VMs)

### 7.4 Empfohlene Tests und Optimierungen

Zur Bewertung und Verbesserung der Performance sind u. a. folgende Maßnahmen sinnvoll:

1. **Test mit kleineren/komprimierten Testvideos**

   * z. B. ein kurzes 720p- oder 1080p-Video in gängigem H.264-Format
   * Test per DDNS aus dem Mobilnetz → Vergleich zu großen 4K-Dateien

2. **Transkodierungsfähigkeit des NAS prüfen**

   * In Synology Video Station/Photos prüfen, ob Hardware-Transkodierung verfügbar ist
   * Falls vorhanden, aktivieren und testen

3. **Upload-Bandbreite unter Last testen**

   * Parallel einen Speedtest von zuhause aus (Upstream) durchführen
   * Sicherstellen, dass keine laufenden Backups oder andere heavy Uploads die Leitung belegen

4. **Netzqualität unterwegs prüfen**

   * An einem Ort mit sicher gutem LTE/5G-Empfang testen
   * Vergleichstest im WLAN (z. B. Gast-WLAN bei Freunden) → hilft, Mobilfunk als Fehlerquelle einzugrenzen

5. **Alternative Streaming-Strategie für „wichtige“ Videos**

   * Für Videos, die sicher unterwegs gezeigt werden sollen, ggf. vorab eine „mobilfreundliche“ Version erzeugen (z. B. 1080p, moderate Bitrate) und diese gezielt verwenden.

## 8. Offene Punkte / Phase 2: Nginx Proxy Manager auf Port 443

In dieser Dokumentation wurde bewusst zunächst **Phase 1** abgeschlossen:

* Direkter, funktionierender und sicherer DDNS-Zugriff auf Synology Photos über Port 5001
* Photos-App nutzt wieder DDNS statt QuickConnect

**Offene Baustelle für ein eigenes Projekt („Phase 2“):**

* Konsolidierung aller externen Dienste über **Nginx Proxy Manager auf Port 443**
* Einrichten von `photos.mydomain.de` als Proxy-Host auf NPM
* Analyse und Behebung der aktuellen TLS-/SNI-Probleme auf 443 (TLS-Alert `unrecognized name` bei `curl https://192.168.88.106`)
* Zielbild: Nur Port 443 von außen offen, alle Dienste dahinter über Subdomains

---

Diese Dokumentation beschreibt den Stand nach erfolgreicher Wiederherstellung des externen Zugriffs auf Synology Photos über DDNS und Port 5001. Weitere Arbeiten an der NPM-/443-Konsolidierung werden in einem separaten Projekt dokumentiert.
