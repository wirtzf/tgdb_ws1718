# Tutorium - Grundlagen Datenbanken - Blatt 4

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `tutorium.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/datamodler_schema.png)

## Aufgaben

### Aufgabe 1
Um genauere Informationen und Prognosen mit Data Mining Werkzeugen zu schöpfen, ist es notwendig mehr Informationen über die registrierten Benutzer 
zu sammeln und zu speichern. Die in Zukunft gesammelten Informationen sollen in neuen Tabellen des bestehenden Datenbankmodells gespeichert werden. 
Dazu soll jedem Benutzer einen Erst- und Zweitwohnsitz zugeordnet werden. Jeder Wohnsitz besitzt eine eigene Adresse. Integriere in das bestehende 
Datenbankmodell Tabellen die den genauen Erst- und Zweitwohnsitz abbilden können. 
Beachte dazu die Normalisierungsformen bis 3NF - [Dokumentation](https://de.wikipedia.org/wiki/Normalisierung_(Datenbank)). 
Wie lautet deine SQL-Syntax um deine Erweiterung des Datenbankmodells zu implementieren?

#### Lösung
```sql
-- Tabelle für Wohnsitz anlegen
CREATE TABLE RESIDENCE 
  (
    RESIDENCE_ID NUMBER(38) NOT NULL,
    ADDRESS_ID NUMBER(38) NOT NULL,
	ACCOUNT_ID NUMBER(38) NOT NULL,
	COUNTRY_ID NUMBER(38) NOT NULL,
    FIRST_STREET VARCHAR(32) NOT NULL,
	SECOND_STREET VARCHAR(32) NOT NULL,
    CONSTRAINT PK_RESIDENCE PRIMARY KEY (RESIDENCE_ID),
    CONSTRAINT FK_ADDRESS FOREIGN KEY (ADDRESS_ID) REFERENCES ADDRESS(ADDRESS_ID)
	CONSTRAINT FK_ACCOUNT FOREIGN KEY (ACCOUNT_ID) REFERENCES ACCOUNT(ACCOUNT_ID)
	CONSTRAINT FK_COUNTRY FOREIGN KEY (COUNTRY_ID) REFERENCES COUNTRY(COUNTRY_ID)
  );
```

### Aufgabe 2
Als App Entwickler/in für Android und iOS möchtest du dich nicht darauf verlassen, dass die Adresse exakt richtig ist und überlegst in dem 
Datenbankmodell noch zwei zusätzliche Attribute (X und Y Koordinate) zur genauen GPS Lokalisierung einer Tankstelle aufzunehmen. 
Wie lautet deine SQL-Syntax um das Datenbankmodell auf die zwei Attribute zu erweitern?

#### Lösung
```sql
ALTER TABLE GAS_STATION
   ADD (GPS_POS_X NUMBER(10,5),
        GPS_POS_Y NUMBER(10,5));
```

### Aufgabe 3 - Für Enthusiasten
Welche Kunden haben im Jahr 2017 mehr als den Durchschnitt getank?

#### Lösung
```sql
Deine Lösung
```

### Aufgabe 4
Ermittle, warum du INSERT-Rechte auf die Tabelle `SCOTT.EMP` und UPDATE-Rechte auf die Tabelle `SCOTT.DEPT` besitzt. 
Beantworte dazu schrittweise die Aufgaben von 4.1 bis 4.4.

#### Aufgabe 4.1
Wurden die Tabellen-Rechte direkt an dich bzw. an `PUBLIC` vergeben?

##### Lösung
```sql
SELECT *
FROM all_tab_privs
WHERE table_schema = 'SCOTT';
```

#### Aufgabe 4.2
Welche Rollen besitzt du direkt?

##### Lösung
```sql
SELECT * 
FROM user_role_privs;
```

#### Aufgabe 4.3
Welche Rollen haben die Rollen?

##### Lösung
```sql
SELECT *
FROM role_role_privs 
WHERE granted_role = 'FH_TRIER';
```

#### Aufgabe 4.4
Haben die Rollen Rechte an `SCOTT.EMP` oder `SCOTT.DEPT`?

##### Lösung
```sql
SELECT role_role_privs 
FROM dict??? 
WHERE role_role_privs??? LIKE '%SCOTT%'; 

```

### Aufgabe 5 - Für Enthusiasten
Es soll für jede Tankstelle der Umsatz einzelner Jahre aufgelistet werden auf Basis der Daten, die Benutzer durch ihre Quittungen eingegeben haben. 
Sortiere erst nach Jahr und anschließend nach der Tankstelle. Beispiel:

| Jahr  | Anbieter  | Straße            | PLZ   | Stadt | Land          | Umsatz    |
| ----- | --------- | ----------------- | ----- | ----- | --------------| --------- |
| 2017  | Esso      | Triererstraße 15  | 54292 | Trier | Deutschland   | 54784.14  |
| 2017  | Shell     | Zurmainerstraße 1 | 54292 | Trier | Deutschland   | 67874.78  |
| 2016  | Esso      | Triererstraße 15  | 54292 | Trier | Deutschland   | 57412.66  |
| 2016  | Shell     | Zurmainerstraße 1 | 54292 | Trier | Deutschland   | 72478.42  |

#### Lösung
```sql
Deine Lösung
```


