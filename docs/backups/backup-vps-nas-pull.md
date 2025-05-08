# 💽 VPS-Pull-Backup mit DSM und rsync

Dieses Dokument beschreibt die Konfiguration eines **automatisierten Pull-Backups** vom VPS zum NAS. Der Backup-Vorgang basiert auf einem lokalen Backup-Skript auf dem VPS und einer geplanten DSM-Aufgabe, die regelmäßig die Sicherungen abholt. Das Setup folgt der 3-2-1-Backup-Regel.

---

## ✅ Zielsetzung

* VPS sichert Nextcloud lokal (Datenbank + Dateien)
* NAS holt Sicherungen automatisch per rsync
* Sicherer SSH-Zugriff NAS → VPS (keybasiert)
* Wöchentliche Planung via DSM-Aufgabenplaner

---

## ⚙️ Voraussetzungen

| Komponente   | Details                                                                         |
| ------------ | ------------------------------------------------------------------------------- |
| VPS          | Debian 12, Benutzer `stevie`, Nextcloud installiert                             |
| NAS          | Synology DSM 7.2, SSH aktiviert, Benutzer mit SSH-Zugriff (z. B. `adminstevie`) |
| SSH          | Keybasierte Authentifizierung vom NAS auf den VPS                               |
| Speicherziel | `/volume5/VPSbackup` auf dem NAS                                                |

---

## 📂 Aufbau & Ablauf

1. VPS erstellt unter `/root/vps-backup/<Datum>/` folgende Dateien:

   * `db.sql` (MySQL-Dump)
   * `config.tar.gz` (Nextcloud-Konfiguration)
   * `data.tar.gz` (optional: Nextcloud-Daten)

2. NAS führt wöchentlich ein Skript aus:

   * Verbindet sich per SSH
   * Ruft aktuellen Tagesordner mit rsync ab
   * Speichert ihn lokal in `/volume5/VPSbackup/<Datum>/`

---

## 📜 NAS-Pull-Skript (Beispiel)

Pfad: `/volume1/homes/adminstevie/nextcloud-pull.sh`

```bash
#!/bin/bash
# NAS holt VPS-Backup ab

VPS_USER=stevie
VPS_HOST=194.59.205.164
VPS_PATH=/root/vps-backup
NAS_TARGET=/volume5/VPSbackup

# heutiges Datum
DATE=$(date +"%Y-%m-%d")

# Zielordner vorbereiten
mkdir -p "$NAS_TARGET/$DATE"

# Backup vom VPS holen
rsync -az "$VPS_USER@$VPS_HOST:$VPS_PATH/$DATE/" "$NAS_TARGET/$DATE/"
```

> 🔒 Tipp: Achte darauf, dass `id_ed25519` auf dem NAS die Berechtigung `600` hat.

---

## 🛠️ DSM-Aufgabenplaner

1. DSM → Systemsteuerung → Aufgabenplaner
2. „Benutzerdefiniertes Skript“ → `adminstevie`
3. Zeitplan: wöchentlich, Sonntag, 04:00 Uhr
4. Skriptpfad: `/volume1/homes/adminstevie/nextcloud-pull.sh`

---

## 🔄 Wartung & Kontrolle

* **Testlauf** manuell durchführbar: `sh /volume1/homes/adminstevie/nextcloud-pull.sh`
* **Logfiles** bei Bedarf hinzufügen (z. B. per `>> /var/log/vps-pull.log`)
* **Alte Backups löschen** via DSM-Task oder ergänzendes Skript

---

## 🔗 Weiterführende Links

* [Nextcloud VPS Setup auf GitHub](https://github.com/Steviexo/vps-setup)
* [Offizielle rsync-Doku](https://download.samba.org/pub/rsync/rsync.html)
* [DSM Aufgabenplaner](https://kb.synology.com/de-de/DSM/help/DSM/AdminCenter/system_taskScheduler)

---

✍ **Letzte Aktualisierung:** Mai 2025
