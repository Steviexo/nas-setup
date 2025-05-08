# ☁️ Hyper Backup zur Nextcloud (WebDAV-Ziel)

Dieses Dokument beschreibt die Einrichtung eines **Hyper Backup-Ziels auf einer selbstgehosteten Nextcloud-Instanz**, die auf einem VPS betrieben wird. Das Ziel folgt der 3-2-1-Backup-Regel mit einer externen Kopie getrennt vom lokalen Netzwerk.

---

## ✅ Zielsetzung

* Sicherung ausgewählter NAS-Ordner und Anwendungen
* Nutzung von Nextcloud (WebDAV) als externes Ziel
* App-spezifisches Passwort zur Absicherung (2FA aktiv)
* Wöchentliche Sicherung mit Versionierung

---

## 🧩 Systemübersicht

| Komponente            | Beschreibung                    |
| --------------------- | ------------------------------- |
| NAS                   | Synology DSM 7.2                |
| Zielsystem            | Nextcloud auf VPS (Debian 12)   |
| Übertragungsweg       | WebDAV (HTTPS)                  |
| Verbindungssicherheit | TLS + App-spezifisches Passwort |

---

## 📁 Zielstruktur (Nextcloud)

1. In der Nextcloud Weboberfläche einloggen (z. B. `https://nextcloud.queerheliophelia.de`)

2. Ordner erstellen: `NAS-Backup`

3. WebDAV-Adresse:

   ```
   https://nextcloud.queerheliophelia.de/remote.php/dav/files/<USERNAME>/NAS-Backup
   ```

   Ersetze `<USERNAME>` durch deinen Nextcloud-Benutzernamen

4. Erstelle ein **App-spezifisches Passwort** unter `Einstellungen → Sicherheit`

---

## 🔧 Hyper Backup-Konfiguration (Beispiel)

* Sicherungstyp: WebDAV
* Serveradresse:

  ```
  https://nextcloud.queerheliophelia.de/remote.php/dav/files/nextstevie/NAS-Backup
  ```
* Benutzername: `nextstevie`
* Passwort: App-spezifisches Passwort (nicht das Standardpasswort)

### 📁 Gesicherte Daten

* **Ordner**:

  * `/volume1/docker` ✅
  * `/volume1/homes`
  * `/volume1/PlexMediaServer`
  * `/volume2/media/Dokumente`
  * `/volume2/Obsidian-notes`

* **Anwendungen (Pakete)**:

  * Hyper Backup
  * Note Station
  * Synology Contacts
  * Synology Drive Server
  * Synology Photos (optional)

### 🔄 Versionierung

* Art: „Intelligente Versionen“
* Anzahl Versionen: 2

### 📅 Zeitplan

* **Häufigkeit**: Wöchentlich
* **Tag/Uhrzeit**: Sonntag, 03:00 Uhr

---

## 🛡️ Tipps & Wartung

* **Speicherplatzbedarf regelmäßig prüfen** in Nextcloud
* **Wiederherstellungstest** jährlich durchführen (lokal)
* App-Passwort sichern und dokumentieren
* Bei Benutzerwechsel neuen Nextcloud-Nutzer oder App-Passwort anlegen

---

## 🔗 Weiterführende Links

* [Synology Hyper Backup – Offizielle Anleitung](https://kb.synology.com/de-de/DSM/help/HyperBackup/hyperbackup)
* [Nextcloud WebDAV-Dokumentation](https://docs.nextcloud.com/server/latest/user_manual/files/access_webdav.html)
* [VPS Setup auf GitHub](https://github.com/Steviexo/vps-setup)

---

✍ **Letzte Aktualisierung:** Mai 2025
