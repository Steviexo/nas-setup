# Paperless-ngx Reverse Proxy Setup auf Synology NAS

## Übersicht

Dieses Dokument beschreibt den Prozess der Einrichtung eines Reverse Proxys auf einer Synology NAS für die Anwendung Paperless-ngx, die in einem Docker-Container läuft. Es wird auch beschrieben, wie auftretende Probleme behoben wurden.

## Voraussetzungen

- Synology NAS mit DSM 6.2 oder höher
- Docker-Paket installiert
- Paperless-ngx in einem Docker-Container konfiguriert und lauffähig
- Zugriff auf DSM und die Netzwerkkonfiguration der Synology NAS
- Eine Subdomain (z.B. `paperless.steviexo.synology.me`) über Synology DDNS
- Grundkenntnisse im Umgang mit Docker und Netzwerken

## Schritte zur Einrichtung

### 1. **Einrichtung des Docker-Containers**

Stelle sicher, dass der Paperless-ngx-Container läuft und auf Port 8010 über HTTP erreichbar ist. Dies kann durch den direkten Aufruf der URL im Browser (`http://<IP-der-NAS>:8010`) überprüft werden.

### 2. **Erstellung und Zuweisung eines SSL-Zertifikats**

- Gehe zu **Systemsteuerung** > **Sicherheit** > **Zertifikat**.
- Erstelle ein neues Let's Encrypt-Zertifikat für die Subdomain `paperless.steviexo.synology.me`.
- Weisen Sie das Zertifikat der Subdomain und dem entsprechenden Dienst zu.

### 3. **Einrichtung des Reverse Proxys**

- Gehe zu **Systemsteuerung** > **Anwendungsportal** > **Reverse Proxy**.
- Erstelle eine neue Regel mit folgenden Einstellungen:
  - **Quelle:**
    - **Protokoll:** HTTPS
    - **Hostname:** `paperless.steviexo.synology.me`
    - **Port:** 443
  - **Ziel:**
    - **Protokoll:** HTTP
    - **Hostname:** `192.168.178.32` (IP der NAS)
    - **Port:** 8010
- **Benutzerdefinierte Kopfzeilen:**
  - Füge eine Kopfzeile hinzu:
    - **Name:** `Host`
    - **Wert:** `paperless.steviexo.synology.me`

### 4. **Überprüfung der Umgebungsvariablen**

Falls Paperless-ngx auf Anfragen vom Reverse Proxy mit einem "400 Bad Request"-Fehler reagiert, überprüfe die Umgebungsvariablen des Docker-Containers:

- Stelle sicher, dass die Umgebungsvariable `PAPERLESS_URL` oder `PAPERLESS_BASE_URL` (je nach Konfiguration) auf `https://paperless.steviexo.synology.me` gesetzt ist.

### 5. **Firewall-Konfiguration**

Stelle sicher, dass die Firewall auf der Synology NAS so konfiguriert ist, dass sie den Zugriff auf die relevanten Ports (443 für HTTPS und 8010 für HTTP) zulässt. Dies kann durch das Erstellen spezifischer Regeln geschehen.

### 6. **Fehlerbehebung**

Falls Probleme auftreten:
- Überprüfe die Logs des Paperless-ngx-Containers und des DSM-Webservers.
- Stelle sicher, dass alle DNS-Einträge korrekt gesetzt sind und die Subdomain auf die öffentliche IP der NAS verweist.
- Vergewissere dich, dass keine Verweigerungsregeln in der Firewall aktiv sind, die den Zugriff blockieren könnten.

## Schlussfolgerung

Nachdem die Umgebungsvariable korrekt gesetzt wurde, funktionierte der Reverse Proxy wie erwartet, und Paperless-ngx war über die Subdomain `https://paperless.steviexo.synology.me` erreichbar. Diese Dokumentation bietet eine umfassende Anleitung, wie man ähnliche Setups auf einer Synology NAS einrichtet und Fehler behebt.

## Lizenz

Dieses Projekt steht unter der MIT-Lizenz. Weitere Informationen finden Sie in der LICENSE-Datei.
