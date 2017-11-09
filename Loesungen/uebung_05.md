# Tutorium - Grundlagen Datenbanken - Blatt 5

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `01_tutorium.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/datamodler_schema.png)

## Aufgaben

### Aufgabe 1
Erstelle mit Dia oder einem anderen Werkzeug eine Abbilung der Mengen, die durch `INNER JOIN`, `RIGHT JOIN`, `LEFT JOIN` und `OUTER JOIN` gemeint sind.

#### Lösung
Grafiken schön zu sehen bei den Links!!!
INNER JOIN: 	 Liefert die Schnittmenge zweier Tabellen (https://www.w3schools.com/sql/sql_join_inner.asp) 
LEFT JOIN:		 Liefert die Schnittmenge zweier Tabellen plus zusätzlich alle anderen Einträge der linken Tabelle. (https://www.w3schools.com/sql/sql_join_left.asp) 
RIGHT JOIN:		 Liefert die Schnittmenge zweier Tabellen plus zusätzlich alle anderen Einträge der rechten Tabelle. (https://www.w3schools.com/sql/sql_join_right.asp) 
FULL OUTER JOIN: Liefert die Schnittmenge zweier Tabellen plus zusätzlich alle anderen Einträge aus beiden Tabellen. (https://www.w3schools.com/sql/sql_join_full.asp)
Prinzipiell sind LEFT- und RIGHT- auch OUTER JOINS, OUTER kann jedoch weggelassen werden.


### Aufgabe 2
Welche Personen haben kein Fahrzeug? Löse dies einmal mit `LEFT JOIN` und `RIGHT JOIN`.

#### Lösung
```sql
--LEFT JOIN:
SELECT forename, surname
FROM account acc
LEFT JOIN acc_vehic av ON acc.account_id=av.account_id AND av.vehicle_id is NULL;

--RIGHT JOIN:
SELECT forename, surname
FROM acc_vehic av 
RIGHT JOIN account acc ON av.account_id=acc.account_id AND av.vehicle_id is NULL;
```
