# Paperless-NGX: Warteschlangenmanagement und Bereinigung nach einem Neustart

## Projektübersicht

Dieses Projekt dokumentiert die Lösung eines Problems, das bei der Nutzung von Paperless-NGX in einer Docker-Umgebung auftrat. Nach einem Neustart des Containers stellte sich heraus, dass die Warteschlange für den Import von Dokumenten massiv anwuchs, da viele Duplikate entstanden. Diese Dokumentation beschreibt die Schritte, die unternommen wurden, um die Warteschlange zu bereinigen und den Importprozess wieder in Gang zu bringen.

## Problemstellung

Nach dem Neustart des Paperless-NGX Containers wurden doppelt so viele Dokumente in der Warteschlange angezeigt. Es stellte sich heraus, dass der Neustart zu Duplikaten in der Warteschlange führte. Die Herausforderung bestand darin, diese Duplikate effizient zu entfernen, ohne den Importprozess zu beeinträchtigen.

## Lösungsschritte

### 1. Zugang zur Datenbank

Zuerst wurde Zugang zur PostgreSQL-Datenbank von Paperless-NGX benötigt, um die Warteschlange direkt zu bearbeiten.

```bash
docker exec -it <paperless-container-name> /bin/bash
psql -U paperless

### 2. Identifikation der relevanten Tabelle

Die Tabelle `documents_paperlesstask` wurde als die relevante Tabelle identifiziert, die die Warteschlangenaufgaben enthält.

```sql
\dt
SELECT * FROM documents_paperlesstask LIMIT 10;


### Schritt 3: Überprüfung auf Duplikate

```markdown
### 3. Überprüfung auf Duplikate

Es wurde eine SQL-Abfrage ausgeführt, um zu überprüfen, ob Duplikate in der Warteschlange vorhanden sind.

```sql
SELECT task_file_name, COUNT(*)
FROM documents_paperlesstask
WHERE status = 'PENDING'
GROUP BY task_file_name
HAVING COUNT(*) > 1;


### Schritt 4: Bereinigung der Duplikate

```markdown
### 4. Bereinigung der Duplikate

Eine SQL-Abfrage wurde verwendet, um die Duplikate zu entfernen und nur eine Instanz jeder Datei in der Warteschlange zu behalten.

```sql
DELETE FROM documents_paperlesstask
WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY task_file_name ORDER BY id) AS rnum
    FROM documents_paperlesstask
    WHERE status = 'PENDING'
  ) t
  WHERE t.rnum > 1
);


### Schritt 5: Überprüfung der Bereinigung

```markdown
### 5. Überprüfung der Bereinigung

Nach der Bereinigung wurde die verbleibende Anzahl der `PENDING`-Aufgaben überprüft, um den Erfolg der Operation sicherzustellen.

```sql
SELECT COUNT(*) FROM documents_paperlesstask WHERE status = 'PENDING';


### Schritt 6: Ergebnis

```markdown
### 6. Ergebnis

Durch die Bereinigung der Duplikate reduzierte sich die Anzahl der Aufgaben in der Warteschlange von etwa 29.000 auf ca. 6.000. Der Importprozess konnte danach erfolgreich fortgesetzt werden.

## Fazit

Diese Dokumentation beschreibt einen effektiven Ansatz zur Verwaltung und Bereinigung der Warteschlange von Paperless-NGX nach einem unerwarteten Container-Neustart. Durch gezielte SQL-Abfragen konnten Duplikate identifiziert und entfernt werden, wodurch der Importprozess wieder stabilisiert wurde.

## Weiterführende Schritte

- **Regelmäßige Überwachung**: Es wird empfohlen, die Warteschlange regelmäßig zu überwachen, um ähnliche Probleme in Zukunft zu vermeiden.
- **Backup-Strategie**: Implementiere eine Backup-Strategie für die Datenbank, um sicherzustellen, dass Daten bei künftigen Problemen schnell wiederhergestellt werden können.
- **Optimierung**: Überlege, ob du weitere Optimierungen am Importprozess vornehmen kannst, um die Effizienz zu steigern und Ausfälle zu minimieren.
