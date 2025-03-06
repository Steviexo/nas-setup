# Paperless-ngx Dokumentenmanagementsystem

## Projektbeschreibung

Dieses Projekt implementiert ein digitales Dokumentenmanagementsystem auf Basis von [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx), das auf einem Synology NAS-System betrieben wird. Paperless-ngx ermöglicht das Scannen, Archivieren und Abrufen von Dokumenten über eine benutzerfreundliche Weboberfläche. Ziel ist es, eine zentrale Lösung zur Verwaltung und Sicherung von Dokumenten zu schaffen, die auch von unterwegs sicher zugänglich ist.

## Inhaltsverzeichnis

- [Systemumgebung](#systemumgebung)
- [Einrichtung](#einrichtung)
- [VPN-Zugriff](#vpn-zugriff)
- [Dokumentenimport](#dokumentenimport)
- [Benutzerverwaltung](#benutzerverwaltung)
- [Tests und Validierung](#tests-und-validierung)
- [Problemlösungen und Optimierungen](#problemlösungen-und-optimierungen)
- [Abschluss und Fazit](#abschluss-und-fazit)

## Systemumgebung

- **NAS-System**: Synology NAS mit Docker-Unterstützung
- **Betriebssystem**: Synology DSM
- **Paperless-ngx**: Docker-Container
- **VPN-Lösung**: OpenVPN über das NAS
- **DDNS**: Dynamische DNS-Domain zur externen Erreichbarkeit des NAS

## Einrichtung

1. **Docker-Compose-Datei erstellen**: Konfiguration der Paperless-ngx-Umgebung mit definierten Volumes und Benutzerrechten.
2. **Environment-Variablen setzen**: Konfiguration der zugelassenen Hosts (`PAPERLESS_ALLOWED_HOSTS`) und Aktivierung der rekursiven Verarbeitung von Dokumenten (`PAPERLESS_CONSUME_RECURSIVE=true`).
3. **Starten des Paperless-ngx Containers**: Überprüfung der Logs auf mögliche Fehler.

Detaillierte Anweisungen findest du in der Datei `Projektdokumentation.md`.

## VPN-Zugriff

OpenVPN-Server auf dem NAS eingerichtet, um einen sicheren Fernzugriff auf Paperless-ngx zu ermöglichen. Weitere Details und Konfigurationen sind in der Dokumentation beschrieben.

## Dokumentenimport

Dokumente, die sich in Unterverzeichnissen befinden, werden durch die Verwendung der `PAPERLESS_CONSUME_RECURSIVE`-Variable automatisch importiert. Alternativ können Dokumente manuell in den `consume`-Ordner verschoben werden.

## Benutzerverwaltung

Mehrere Benutzer können eingerichtet werden, allerdings haben alle Benutzer standardmäßig Zugriff auf alle Dokumente. Zukünftige Erweiterungen könnten die Trennung der Dokumentenbereiche nach Benutzern ermöglichen.

## Tests und Validierung

Nach der Einrichtung wurden verschiedene Tests durchgeführt, um sicherzustellen, dass alle Funktionen wie VPN-Zugriff, Dokumentenimport und Benutzerzugriff wie erwartet funktionieren.

## Problemlösungen und Optimierungen

- Korrektur von Konfigurationsfehlern
- Anpassung von Dateiberechtigungen
- Vermeidung von doppelten Dokumentenimporten

## Abschluss und Fazit

Das Projekt zur Implementierung von Paperless-ngx auf einem Synology NAS wurde erfolgreich abgeschlossen. Die Lösung ermöglicht eine zentrale, sichere und effiziente Verwaltung von Dokumenten, die auch von unterwegs zugänglich ist.

## Zusätzliche Konfigurationen

Für die Einrichtung eines Reverse Proxys auf einer Synology NAS, siehe [Reverse Proxy Setup](REVERSE_PROXY_SETUP.md).

## Projektdokumentation

Wir haben eine umfassende Projektdokumentation erstellt, die beschreibt, wie wir ein Problem mit der Warteschlangenverwaltung in Paperless-NGX nach einem unerwarteten Neustart des Containers gelöst haben. Diese Dokumentation enthält detaillierte Schritte zur Identifikation und Bereinigung von Duplikaten in der Warteschlange, die durch den Neustart verursacht wurden.

### Inhalt der Dokumentation:

- **Problemstellung**: Beschreibung des Problems nach einem Neustart des Containers.
- **Lösungsschritte**: Schritt-für-Schritt-Anleitung zur Identifikation der relevanten Tabelle, Überprüfung auf Duplikate, und Bereinigung der Warteschlange.
- **Ergebnis**: Details zu den Verbesserungen nach der Bereinigung der Warteschlange.
- **Weiterführende Schritte**: Empfehlungen zur Vermeidung ähnlicher Probleme in der Zukunft.

Du findest die vollständige Dokumentation [hier](./Paperless-ngx_clear-duplicates-in-queue.md).


---

**Autor**: ChatGPT  
**Datum**: 20. August 2024
