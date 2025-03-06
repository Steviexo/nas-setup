# Projektdokumentation für die Einrichtung von Paperless-ngx und den Import von Dokumenten

---

## Projektname: Dokumentenmanagementsystem mit Paperless-ngx

### Projektbeschreibung:
Dieses Projekt zielt darauf ab, eine effiziente, digitale Dokumentenverwaltungslösung mit Paperless-ngx auf einem NAS-System zu implementieren. Paperless-ngx ermöglicht das Scannen, Archivieren und Abrufen von Dokumenten in einer benutzerfreundlichen Oberfläche. Bestehende Dokumente sollen in das System integriert werden, um einen schnellen Zugriff zu gewährleisten.

---

## 1. Projektziele

- **Einrichtung von Paperless-ngx** auf einem NAS-System.
- **VPN-Zugriff** auf Paperless-ngx für sicheren Zugriff von unterwegs.
- **Import vorhandener Dokumente** aus einem bestehenden Ordner mit Unterverzeichnissen.
- **Zugriffsbeschränkungen und Benutzerverwaltung**: Einrichten mehrerer Benutzer, die auf separate Dokumentenbereiche zugreifen können.

## 2. Systemumgebung

- **NAS-System**: Synology NAS mit Docker-Unterstützung
- **Betriebssystem**: Synology DSM
- **Paperless-ngx**: Docker-Container
- **VPN-Lösung**: OpenVPN über das NAS
- **DDNS**: Verwendung einer dynamischen DNS-Domain zur Erreichbarkeit des NAS von außerhalb

## 3. Einrichtungsschritte

### 3.1 Einrichtung von Paperless-ngx

1. **Docker-Compose-Datei erstellen**:
   - Definition der Paperless-ngx-Umgebung.
   - Angabe des Volumes für die Dokumentenablage und den `consume`-Ordner.
   - Konfiguration der Benutzer- und Gruppen-ID.

2. **Environment-Variablen konfigurieren**:
   - `PAPERLESS_ALLOWED_HOSTS`: Konfiguration der zugelassenen Hosts (z.B. localhost, DDNS-Domain).
   - `consume`-Ordner: `/volume2/media/Dokumente:/usr/src/paperless/consume`.
   - **Wichtige Variable**: `PAPERLESS_CONSUME_RECURSIVE=true`, um sicherzustellen, dass auch Dokumente in Unterverzeichnissen automatisch erkannt und verarbeitet werden.

3. **Paperless-Container starten**:
   - Überprüfen der Logs auf Fehler und korrekte Einrichtung.

### 3.2 VPN-Zugriff konfigurieren

1. **OpenVPN-Server auf dem NAS einrichten**:
   - Import der Konfigurationsdateien auf die mobilen Geräte.
   - Test des Zugriffs auf die Paperless-ngx-Weboberfläche (http://192.168.178.32:8010) über VPN.

2. **Portsicherung**:
   - Deaktivieren nicht mehr benötigter Portfreigaben (Port 500, 5001, 80, 443).

### 3.3 Import vorhandener Dokumente

1. **Problematik**:
   - Vorhandene Dokumente sind in Unterverzeichnissen gespeichert, die von Paperless-ngx standardmäßig nicht automatisch verarbeitet werden.

2. **Lösung**:
   - Anstatt alle Dateien manuell zu kopieren oder Skripte zu nutzen, wurde die Environment-Variable `PAPERLESS_CONSUME_RECURSIVE` auf `true` gesetzt, um die rekursive Verarbeitung von Dokumenten in Unterverzeichnissen durch Paperless-ngx zu ermöglichen.

3. **Manueller Import (optional)**:
   - Alternativ können Dateien manuell in den `consume`-Ordner verschoben oder eine Dateiübertragungssoftware (wie FileStation) verwendet werden, falls die rekursive Verarbeitung aus bestimmten Gründen nicht genutzt werden soll.

### 3.4 Benutzerverwaltung und Zugriffsbeschränkungen

1. **Einrichten weiterer Benutzer**:
   - Weitere Benutzer in Paperless-ngx einrichten.
   - Überprüfung der Zugriffsrechte: Standardmäßig haben alle Benutzer Zugriff auf alle Dokumente.

2. **Getrennte Dokumentenbereiche**:
   - Aktuell keine Möglichkeit, getrennte Dokumentenbereiche pro Benutzer einzurichten. Überlegungen für zukünftige Erweiterungen.

## 4. Tests und Validierung

- **VPN-Zugriffstest**:
   - Erfolgreicher Zugriff auf die Paperless-ngx-Weboberfläche von einem mobilen Gerät über VPN.
  
- **Dokumentenimporttest**:
   - Test mit verschiedenen Dateitypen im `consume`-Ordner, um sicherzustellen, dass alle Dokumente korrekt importiert werden, einschließlich derjenigen, die in Unterverzeichnissen gespeichert sind.
   - Überprüfung des Logs auf mögliche Fehler während des Imports.

- **Benutzerzugriffstest**:
   - Sicherstellen, dass alle Benutzer korrekt eingerichtet sind und die entsprechenden Dokumente einsehen können.

## 5. Problemlösungen und Optimierungen

- **Fehlerbehebung bei Bad Request-Fehler**: Korrektur von Tippfehlern in den `PAPERLESS_ALLOWED_HOSTS`.
- **Optimierung der Dateiberechtigungen**: Sicherstellen, dass der Paperless-Benutzer ausreichende Berechtigungen für den Zugriff auf die Dateien hat.
- **Eindeutige Zuordnung von Dateien**: Skript zur Vermeidung von doppelten Importen verwenden.

## 6. Abschluss und Fazit

Das Projekt zur Einrichtung von Paperless-ngx auf dem NAS wurde erfolgreich abgeschlossen. Alle bestehenden Dokumente, einschließlich solcher in Unterverzeichnissen, wurden dank der `PAPERLESS_CONSUME_RECURSIVE`-Variable korrekt importiert. Der Zugriff über VPN ermöglicht eine sichere Verwaltung der Dokumente von unterwegs. Weitere Erweiterungen und Verbesserungen, wie die Implementierung getrennter Benutzerbereiche, könnten in zukünftigen Projektschritten in Angriff genommen werden.

---

**Dokument erstellt von:** ChatGPT, basierend auf den durchgeführten Schritten und Konfigurationen.  
**Datum:** 20. August 2024
