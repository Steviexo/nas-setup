# 📖 NAS-Setup – Dokumentation & Best Practices

Willkommen im Repository **nas-setup**! Dieses Repository dient als **Wissenssammlung und Best Practices für mein Synology NAS**. Hier dokumentiere ich meine Erfahrungen, um:

✅ **Mein eigenes Wissen zu strukturieren & dokumentieren**\
✅ **Anderen (auch nicht-tech-affinen) Nutzern verständliche Anleitungen bereitzustellen**\
✅ **Meine Fortschritte als zukünftige Admin sichtbar zu machen**

Dieses Repository wächst mit meinen Erfahrungen und ist eine zentrale Anlaufstelle für meine NAS-Konfiguration. Hier sammle ich bewährte Methoden, hilfreiche Anleitungen und Lösungswege für häufige Probleme, um meinen NAS effizient zu verwalten und stetig zu optimieren.

## 🖥 Technische Ausstattung

Dieses Repository basiert auf einem **Synology DS918+ NAS** mit folgender Ausstattung:

- **Modell**: Synology DS918+
- **Prozessor**: Intel Celeron J3455, 4 Kerne @ 1,5 GHz (Burst bis 2,3 GHz)
- **RAM**: 8 GB DDR3L (erweiterbar)
- **Festplatten**: 4 Einschübe (aktuelles Setup: Seagate Exos X16 ST12000NM001G 12TB; HGST Ultrastar HE12 12TB HDD SATA 6Gb/s 4KN SE 7200Rpm HUH721212ALN604 in separaten Raids)
- **Betriebssystem**: DSM (DiskStation Manager) 7.2
- **Zusätzliche Erweiterungen**: /

Falls du ein anderes NAS verwendest, können sich einige Konfigurationsschritte unterscheiden.

## 📂 Verzeichnisstruktur & Unterthemen

```
nas-setup/
├── docs/                 # Dokumentation zu diesem Repository
│   ├── README.md         # Einführung und Übersicht
│   ├── architecture.md   # Technische Architektur
│   ├── faq.md            # Häufig gestellte Fragen
│   ├── installation.md   # Installationsanleitung
│   ├── docker/           # Alles rund um Docker
│   │   ├── paperless-ngx.md  # Dokumentation zu Paperless-NGX
│   │   ├── nextcloud.md      # Einrichtung von Nextcloud
├── templates/            # Vorlagen für Konfigurationen oder Anleitungen
│   ├── docker-setup.md   # Beispiel-Vorlage für Docker-Setup
├── LICENSE               # Lizenzinformationen
├── CONTRIBUTING.md       # Hinweise für Beiträge zum Repository
└── README.md             # Diese Datei
```

## 📌 Enthaltene Themen

### 🔹 **Docker & Anwendungen**

- **[Paperless-NGX](docs/docker/paperless-ngx.md)**: Dokumentenmanagement automatisieren
- **[Nextcloud](docs/docker/nextcloud.md)**: Private Cloud & Dateiverwaltung

### 🔹 **NAS-Konfiguration & Administration**

- **Systemhärtung & Sicherheit** (Geplant)
- **RAID & Speichermanagement** (Geplant)

### 🔹 **Backup & Recovery**

- **[Hyper Backup zu Nextcloud (WebDAV)](docs/backups/hyper-backup-nextcloud.md)**: Sicherung von NAS-Daten auf VPS-Cloudziel
- **[VPS-Pull per rsync](docs/backups/backup-vps-nas-pull.md)**: Automatisiertes Abholen von Backups per DSM-Aufgabe

### 🚧 **In Arbeit / Geplante Inhalte**

- **Netzwerk & Firewall**
- **Backup-Strategien**

---

## 📝 Warum dieses Repository?

Ich nutze GitHub primär als **persönliches Dokumentations- und Wissensmanagement-Tool**. Dieses Repository hilft mir dabei, …

- meine **NAS-Konfiguration & Best Practices** zu dokumentieren
- anderen (auch ohne tiefes technisches Wissen) Lösungen für ähnliche Probleme bereitzustellen
- meine **Erfahrungen als zukünftiger Systemadministrator** sichtbar zu machen

Falls du Fragen hast oder Feedback geben möchtest, freue ich mich über deine Nachricht! 😊

---

## 🚀 Verwendung

Hier findest du alle wichtigen Informationen zur Nutzung dieses Repositories.

- **Installation**: Lies die [Installationsanleitung](docs/installation.md) für erste Schritte.
- **FAQ**: Häufige Fragen werden in der [FAQ](docs/faq.md) beantwortet.
- **Architektur**: Eine technische Übersicht findest du in [architecture.md](docs/architecture.md).

## 🤝 Mitwirken

Falls du Änderungen oder Verbesserungen beisteuern möchtest, lies die [CONTRIBUTING.md](CONTRIBUTING.md).

## 🔗 Weiterführende Informationen

- [Offizielle Dokumentation](#)
- [GitHub Issues](#)
- [Projekt-Template](https://github.com/steviexo/project-template)

---

✍ **Letzte Aktualisierung:**&#x20;
