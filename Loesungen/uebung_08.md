# Tutorium - Grundlagen Datenbanken - Blatt 8

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `tutorium.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/datamodler_schema.png)

### Aufgabe 1
Erstelle eine Prozedur, die das anlegen von Benutzern durch übergabe von Parametern ermöglicht.

#### Lösung
```sql
--geil wäre ein Array für die Parameter, aber noch nicht gelernt wie und ob es überhaupt geht
CREATE OR REPLACE PROCEDURE CreateUser(surname IN VARCHAR2, forename IN VARCHAR2, email IN VARCHAR2, c_date IN DATE, u_date IN DATE) 
AS
v_id 	account.account_id%TYPE;
BEGIN
  SELECT MAX(account_id) +1 INTO v_id FROM account;
  
  INSERT INTO account 
  VALUES(v_id, surname, forename, email, c_date, u_date);
  
  DBMS_OUTPUT.PUT_LINE('Account mit der ID :'||v_id||' angelegt!');
END;
/

--Prozedur aufruf
EXEC CreateUser('Wirtz','Fabian', 'wirtzf@hs-trier.de', SYSDATE, SYSDATE);
```

### Aufgabe 2
Erstelle eine Prozedur, die das erstellen von Quittungen ermöglicht.  Fange entsprechende übergebene Parameter auf `NULL` oder ` ` ab und gebe eine Meldung aus. 
Ergänze eventuell Parameter die `NULL` sind mit Informationen die sich durch Abfragen abrufen lassen. Berücksichtige die Fehlerbehandlung und mögliche 
Constraints die gebrochen werden könnten!

#### Lösung
```sql
CREATE OR REPLACE PROCEDURE CreateReceipt(in_acc_id IN NUMBER, 
											in_acc_vehic_id IN NUMBER,
											in_duty_amaount IN NUMBER,
											in_gas_id IN NUMBER,
											in_gas_station_id IN NUMBER,
											in_price_l IN NUMBER,
											in_kilometer IN NUMBER,
											in_liter IN NUMBER,
											in_receipt_date IN DATE 
											) 
AS
v_reid 	receipt.receipt_id%TYPE;
v_acc_id account.account_id%TYPE;
v_acc_vehic_id acc_vehic.acc_vehic_id%TYPE;
v_gas_station_id gas_station.gas_station_id%TYPE;
v_duty_amount country.duty_amount%TYPE;
v_gas_id gas.gas_id%TYPE;
BEGIN
	IF (in_acc_id IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: Account_ID ist NULL');
	END IF;
	
	IF (in_acc_vehic_id IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: Acc_Vehic_ID ist NULL');
	END IF;
	
	/*IF (in_gas_id IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: GAS_ID ist NULL');
	END IF;*/
	
	IF (in_gas_station_id IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: GAS_Station ist NULL');
	END IF;
	
	IF (in_price_l IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: PRICE_L ist NULL');
	END IF;
	
	IF (in_kilometer IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: Kilometer ist NULL');
	END IF;
	
	IF (in_liter IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: Liter ist NULL');
	END IF;
	
	IF (in_receipt_date IS NULL)THEN
		RAISE_APPLICATION_ERROR(-20001, ' Fehler: Receipt_Date ist NULL');
	END IF;
	
	--ACCOUNT_ID
	BEGIN
		SELECT account_id INTO v_acc_id
		FROM account
		WHERE account_id = in_acc_id;
	EXCEPTION
		 WHEN NO_DATA_FOUND THEN
			 RAISE_APPLICATION_ERROR(-20001, 'Fehler: Konnte Account_ID '||in_acc_id||' nicht finden!');
	END;
	
	--ACC_VEHIC_ID
	BEGIN
		SELECT acc_vehic_id INTO v_acc_vehic_id
		FROM acc_vehic
		WHERE acc_vehic_id = in_acc_vehic_id;
	EXCEPTION
		WHEN NO_DATA_FOUND THEN
			RAISE_APPLICATION_ERROR(-20001, 'Fehler: Konnte Acc_Vehic_ID '||in_acc_vehic_id||' nicht finden!');
	END;

	--GAS_Station
	BEGIN
		SELECT gas_station_id INTO v_gas_station_id
		FROM gas_station
		WHERE gas_station_id = in_gas_station_id;
	EXCEPTION
		WHEN NO_DATA_FOUND THEN
			RAISE_APPLICATION_ERROR(-20001, 'Fehler: Konnte gas_station_id '||v_gas_station_id||' nicht finden!');
	END;
	
	--Duty_Amount
	BEGIN
		IF (in_duty_amaount IS NULL)THEN
			BEGIN
				SELECT duty_amount INTO v_duty_amount
				FROM country c
				INNER JOIN gas_station gs ON gs.country_id = c.country_id
				WHERE gs.gas_station_id = v_gas_station_id;
			EXCEPTION
				WHEN NO_DATA_FOUND THEN 
					RAISE_APPLICATION_ERROR(-20001, 'Fehler: Konnte Dury_Amount nicht abfragen!');
			END;
		END IF;
	END;
	
	-- PRICE_L
	IF(in_price_l<0) THEN
		 RAISE_APPLICATION_ERROR(-20001, 'Fehler: Der Preis pro Liter kann nicht negativ sein!');
	END IF;
	
	-- Liter
	IF(in_liter<0) THEN
		 RAISE_APPLICATION_ERROR(-20001, 'Fehler: Die Anzahl an getankten Liter muss größer 0 sein!');
	END IF;
	
	--GAS
	IF(in_gas_id IS NULL) THEN
		BEGIN
			SELECT default_gas_id INTO v_gas_id
			FROM vehicle v
			INNER JOIN acc_vehic accv ON accv.vehicle_id = v.vehicle_id
			WHERE accv.acc_vehic_id = v_acc_vehic_id;
		EXCEPTION
			WHEN NO_DATA_FOUND THEN
				RAISE_APPLICATION_ERROR(-20001, 'Fehler: Konnte gas_id '||v_gas_id||' nicht finden!');
		END;
	END IF;
	
  SELECT MAX(NVL(receipt_id, 0)) +1 INTO v_reid FROM receipt;
  
  INSERT INTO receipt 
  VALUES(v_reid, v_acc_id, v_acc_vehic_id, v_duty_amount, v_gas_id, v_gas_station_id, in_price_l, in_kilometer, in_liter, in_receipt_date, SYSDATE, SYSDATE);
  
  DBMS_OUTPUT.PUT_LINE('Receipt mit der ID :'||v_reid||' angelegt!');
  
EXCEPTION
	WHEN OTHERS THEN
		RAISE_APPLICATION_ERROR(-20001, 'IRGEND WAS IST SCHIEF GELAUFEN');
END;
/

--Ausfrühren der Prozedur
EXEC CreateReceipt(10, 2, NULL, NULL, 7, 1, 222, 42, SYSDATE );
```












