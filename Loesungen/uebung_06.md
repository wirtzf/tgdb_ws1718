# Tutorium - Grundlagen Datenbanken - Blatt 6

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `01_tutorium.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/datamodler_schema.png)

## Data-Dictionary-Views
![Data-Dictionary-Views](./img/constraint_schema.png)

## Aufgaben

### Aufgabe 1
Wie heißt der Primary Key Contraint der Tabelle `VEHICLE` und für welche Spalten wurde er angelegt?

#### Lösung
```sql
P = Primary Key
R = Foreign Key
U = Unique
C = CHECK

SELECT constraint_name, constraint_type, column_name
FROM user_constraints NATURAL JOIN user_cons_columns
WHERE table_name = 'VEHICLE' AND constraint_type='P';

ODER:

SELECT ucc.constraint_name, ucc.column_name, ucc.position
FROM user_cons_columns ucc
WHERE constraint_name IN (SELECT constraint_name 
						  FROM user_constraints
						  WHERE table_name LIKE 'VEHICLE'
						  AND constraint_type = 'P'
						  );
```

### Aufgabe 2
Für welche Spalte**n** der Tabelle `ACC_VEHIC` wurde ein Foreign Key angelegt und auf welche Spalte/n in welcher Tabelle wird er referenziert?

#### Lösung
```sql
SELECT uc.constraint_name, uc.constraint_type, ucc.table_name, ucc.column_name
FROM user_constraints uc
INNER JOIN user_cons_columns ucc ON uc.r_constraint_name=ucc.constraint_name
WHERE uc.table_name = 'ACC_VEHIC' AND uc.constraint_type='R';  
```

### Aufgabe 3
Erstelle einen Check Constraint für die Tabelle `ACCOUNT`, dass der Wert der Spalte `U_DATE` nicht älter sein kann als `C_DATE`.

#### Lösung
```sql
ALTER TABLE ACCOUNT 
ADD CONSTRAINT c_ucdate CHECK (U_DATE >= C_DATE); 
```

### Aufgabe 4
Erstelle einen Check Constraint der überprüft, ob der erste Buchstabe der Spalte `GAS_NAME` der Tabelle `GAS` groß geschrieben ist.

#### Lösung
```sql
ALTER TABLE GAS
ADD constraint c_initcap CHECK (gas_name=initcap(gas_name));

ODER:

ALTER TABLE gas
ADD constraint c_gas_name CHECK (REGEXP_LIKE(gas_name, '^[A-Z].*$', 'c'));
```

### Aufgabe 5
Erstelle einen Check Contraint der überprüft, ob der Wert der Spalte `IDENTICATOR` der Tabelle `ACC_VEHIC` eins von diesen möglichen Fahrzeugkennzeichenmustern entspricht. Nutze Reguläre Ausdrücke.

+ B:AB:5000
+ TR:MP:1
+ Y:123456
+ THW:98765
+ MZG:XZ:96

#### Lösung
```sql
ALTER TABLE ACC_VEHIC
ADD constraint c_kennzeichen CHECK (REGEXP_LIKE(identicator, '^[A-Z]{1,3}:([A-Z]{1,2}:[1-9]{0,3}|[1-9][0,9]{0,5})$', 'c'));
```

### Aufgabe 6 - Wiederholung
Liste für alle Personen den Verbrauch an Kraftstoff auf (Missachte hier die unterschiedlichen Kraftstoffe). Dabei ist interessant, wie viel Liter die einzelne Person getankt hat und wie viel Euro sie für Kraftstoffe ausgegeben hat.

#### Lösung
```sql
COLUMN SURNAME FORMAT a15
COLUMN foreNAME FORMAT a15

SELECT acc.surname, acc.forename,
		(SELECT SUM(re.price_l*re.liter*1+re.duty_amount)
		 FROM receipt re
		 WHERE re.account_id = acc.account_id
		 GROUP BY re.account_id
		)"Ausgaben",
		(SELECT SUM(re.liter)
		 FROM receipt re
		 WHERE re.account_id = acc.account_id
		)"Getankte Liter"
FROM account acc;
```

### Aufgabe 7 - Wiederholung
Liste die Tankstellen absteigend sortiert nach der Kundenanzahl über alle Jahre.

#### Lösung
```sql
SELECT TO_CHAR(r.C_DATE, 'YYYY') "Jahr",
	   p.provider_name "Provider",
	   gs.street "Straße",
	   a.plz "PLZ",
	   a.city "Stadt",
	   COUNT(r.account_id) "Anzahl"
FROM gas_station gs
INNER JOIN provider p ON (p.provider_id=gs.provider_id)
INNER JOIN address a ON (a.address_id=gs.address_id)
INNER JOIN receipt r ON (r.gas_station_id=gs.gas_station_id)
GROUP BY r.c_date, p.provider_name, gs.street, a.plz, a.city;
```

### Aufgabe 8 - Wiederholung
Erweitere das Datenbankmodell um ein Fahrtenbuch, sowie es Unternehmen für ihren Fuhrpark führen. Dabei ist relevant, welche Person an welchem Tag ab wie viel Uhr ein Fahrzeug für die Reise belegt, wie viele Kilometer zurück gelegt wurden und wann die Person das Fahrzeug wieder abgibt.

Berücksichtige bitte jegliche Constraints!

#### Lösung
```sql
CREATE TABLE LBOOK(
	LBOOK_ID 		NUMBER(38) NOT NULL,
	ACCOUNT_ID 		NUMBER(38) NOT NULL,
	ACC_VEHIC_ID 	NUMBER(38) NOT NULL,
	B_DATE 			DATE NOT NULL,
	KILOMETER 		NUMBER(7,3) NOT NULL,
	S_DATE 			DATE NOT NULL
);

ALTER TABLE LBOOK
ADD constraint p_lbook PRIMARY KEY (LBOOK_ID);

ALTER TABLE LBOOK
ADD constraint r_account FOREIGN KEY (ACCOUNT_ID) REFERENCES ACCOUNT(ACCOUNT_ID);

ALTER TABLE LBOOK
ADD constraint r_acc_vehic FOREIGN KEY (ACC_VEHIC_ID) REFERENCES ACC_VEHIC(ACC_VEHIC_ID);

ALTER TABLE LBOOK
ADD constraint c_date CHECK(S_DATE >= B_DATE);
```
