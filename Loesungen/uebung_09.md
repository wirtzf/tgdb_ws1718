# Tutorium - Grundlagen Datenbanken - Blatt 9

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `tutorium.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/datamodler_schema.png)

### Aufgabe 1
Wo liegen die Vor- und Nachteile eines Trigger in Vergleich zu einer Prozedur?

Der Trigger überwacht automatisch ob Tabellen aktualisiert oder Datensätze eingefügt werden, und kann direkt
auf eine konsistente Datenbank achten.

#### Lösung
Deine Lösung

### Aufgabe 2
Wo drin unterscheidet sich der `Row Level Trigger` von einem `Statement Trigger`?

#### Lösung
Deine Lösung

### Aufgabe 3
Schaue dir den folgenden PL/SQL-Code an. Was macht er?

```sql
CREATE SEQUENCE seq_account_id //autoinc wert für die account_id erstellen
START WITH 1000  //startwert
INCREMENT BY 1  //erhöhungsfaktor
MAXVALUE 99999999 //maximalwert
CYCLE
CACHE 20;

CREATE OR REPLACE TRIGGER BIU_ACCOUNT  //Trigger erstellen oder überschreiben
BEFORE INSERT OR UPDATE OF account_id ON account //vor INSERT oder UPDATE der account_id
FOR EACH ROW
DECLARE

BEGIN
  IF UPDATING('account_id') THEN   //siehe Fehlermeldung
    RAISE_APPLICATION_ERROR(-20001, 'Die Account-ID darf nicht verändert oder frei gewählt werden!');
  END IF;

  IF INSERTING THEN
    :NEW.account_id := seq_account_id.NEXTVAL;  //account_id wird automatisch mit der nächsten sequence gefüllt
  END IF;
END;
/
```

#### Lösung
Deine Lösung

### Aufgabe 4
Verbessere den Trigger aus Aufgabe 2 so, dass
+ wenn versucht wird einen Datensatz mit `NULL` Werten zu füllen, die alten Wert für alle Spalten, die als `NOT NULL` gekennzeichnet sind, behalten bleiben.
+ es nicht möglich ist, das die Werte für `C_DATE` und `U_DATE` in der Zukunkt liegen
+ `U_DATE` >= `C_DATE` sein muss
+ der erste Buchstabe jedes Wortes im Vor- und Nachnamen groß geschrieben wird
+ die Account-ID aus einer `SEQUENCE` entnommen wird

Nutze die Lösung der Aufgabe 2, Aufgabenblatt 8 um die Aufgabe zu lösen. Dort solltest du einige Hilfestellungen finden.

#### Lösung
```sql
CREATE SEQUENCE seq_account_id
START WITH 1000
INCREMENT BY 1
MAXVALUE 99999999
CYCLE
CACHE 20;


CREATE OR REPLACE TRIGGER BIU_ACCOUNT
BEFORE INSERT OR UPDATE OF account_id ON account
FOR EACH ROW
DECLARE

BEGIN
  IF UPDATING('account_id') THEN
    RAISE_APPLICATION_ERROR(-20001, 'Die Account-ID darf nicht verändert oder frei gewählt werden!');
  END IF;
  
  IF (INSERTING OR UPDATING)('c_date') or (INSERTING OR UPDATING)('u_date') THEN
	IF DATE(:NEW.c_date) > DATE(NOW()) THEN
		RAISE_APPLICATION_ERROR(-20011, 'Das c_date darf nicht neuer als das aktuelle Datum sein!');
	ELSIF DATE(:NEW.u_date) > DATE(NOW()) THEN
		RAISE_APPLICATION_ERROR(-20011, 'Das u_date darf nicht neuer als das aktuelle Datum sein!');
	ELSIF DATE(:NEW.u_date) < DATE(:NEw.c_date) THEN
		RAISE_APPLICATION_ERROR(-20011, 'Das u_date darf nicht kleiner als das c_date sein!');
  END IF;
  
  IF (INSERTING OR UPDATING)('surname') or (INSERTING OR UPDATING)('forename') THEN
	IF (TRUE <> REGEXP_LIKE (:NEW.surname, '^[a-z]+[a-zA-Z0-9._%-]$'))THEN
		RAISE_APPLICATION_ERROR(-20011, 'Der erste Buchstabe des Nachnamen muss klein sein!');
	ELSIF (TRUE <> REGEXP_LIKE (:NEW.forename, '^[a-z]+[a-zA-Z0-9._%-]$'))THEN
		RAISE_APPLICATION_ERROR(-20011, 'Der erste Buchstabe des Fornamens muss klein sein!');
  END IF;

  IF INSERTING THEN
    :NEW.account_id := seq_account_id.NEXTVAL;
  END IF;
END;
/
```

### Aufgabe 5
Angenommen der Steuersatz in Deutschland sinkt von 19% auf 17%.
+ Aktualisiere den Steuersatz von Deutschland und
+ alle Quittungen die nach dem `01.10.2017` gespeichert wurden.

#### Lösung
```sql
UPDATE country
SET duty_amount = 17
WHERE UPPER(country_name) = 'DEUTSCHLAND';

UPDATE receipt
SET duty_amount
WHERE receipt_date > '01.10.2017';
```

### Aufgabe 6
Liste alle Hersteller auf, die LKW's produzieren und verknüpfe diese ggfl. mit den Eigentümern.

#### Lösung
```sql
SELECT pr.producer_name, acc surname
FROM producer pr
INNER JOIN vehicle ve ON ve.producer_id=pr.producer_id
INNER JOIN vehicle_type vt ON ve.vehicle_type_id=vt.vehicle_type_id AND vehicle_type_name = 'LKW'
INNER JOIN acc_vehic accv ON accv.vehicle_id=ve.vehicle_id
INNER JOIN account acc ON accv.account_id=acc.account_id;

```


























