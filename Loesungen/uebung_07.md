# Tutorium - Grundlagen Datenbanken - Blatt 7

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `tutorium.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/datamodler_schema.png)

### Aufgabe 1

Analysiere den untenstehenden anonymen PL/SQL-Codeblock. Was macht er?
Passe den Codeblock so an, dass nicht nur die ID des Benutzers ausgegeben wird, sondern auch der Vor- und Nachname, als auch die Anzahl seiner Fahrzeuge.

```sql
DECLARE
  v_account_id account.account_id%TYPE; //Variable Deklarieren

BEGIN
  SELECT MAX(a.account_id) INTO v_account_id
  FROM account a								//die account_id in die Variable schreiben
  WHERE a.surname LIKE 'P%';

  //Ausgabe
  DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat die ID ' || v_account_id);

EXCEPTION
  WHEN NO_DATA_FOUND //wenn Keine daten bei SQL statment zurückgeliefert werden
    THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde kein Benutzer gefunden');
  WHEN OTHERS  //bei allen anderen Fehler
    THEN DBMS_OUTPUT.PUT_LINE ('Folgender unerwarteter Fehler ist aufgetreten: ');
  RAISE;
END;
/
```

#### Lösung
```sql
DECLARE
	v_account_id account.account_id%TYPE;
	v_forename account.forename%TYPE;
	v_surname account.surname%TYPE;
	v_Fahrzeuge NUMBER;

BEGIN
	
	SELECT MAX(a.account_id), a.forename, a.surname, COUNT(av.vehicle_id) INTO v_account_id, v_forename, v_surname, v_Fahrzeuge
	FROM account a
	INNER JOIN Acc_Vehic av ON a.account_id=av.account_id AND av.vehicle_id IN (SELECT vehicle_id FROM vehicle)
	WHERE a.surname LIKE '%P'
	GROUP BY av.vehicle_id, a.account_id, a.forename, a.surname;
	

	DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat die ID ' || v_account_id);
	DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat den Vornamen ' || v_forename);
	DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat d Nachnamen ' || v_surname);
	DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat ' || v_Fahrzeuge ||' Fahrzeuge');

	EXCEPTION
	  WHEN NO_DATA_FOUND
		THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde kein Benutzer gefunden');
	  WHEN OTHERS
		THEN DBMS_OUTPUT.PUT_LINE ('Folgender unerwarteter Fehler ist aufgetreten: ');
	  RAISE;
END;
/
```

### Aufgabe 2
Schreibe einen anonymen PL/SQL-Codeblock, der die Tankstelle mit der kleinsten ID 
auflistet mit Informationen über den Anbieter und der Addresse. Implementiere ein `IF-ELSE` Konstrukt, 
dass wenn eine Tankstelle mehr Kundenbesuch erziehlt hat, als alle anderen im Durchschnitt, die Tankstelle als gut 
Besucht gekennzeichnet wird in der Ausgabe. Andernfalls wird die Tankstelle als schlecht Besucht gekennzeichnet.

#### Lösung
```sql
//kleinste ID = 1 für den rest aber mehrere -> Cursor
DECLARE
	v_gs_id gas_station.gas_station_id%TYPE;
	v_pn provider.provider_name%TYPE;
	v_plz address.plz%TYPE;
	v_city address.city%TYPE;


BEGIN
	
	SELECT gs.gas_station_id, pr.provider_name, ad.plz, ad.city INTO v_gs_id, v_pn, v_plz, v_city
	FROM gas_station gs
	INNER JOIN provider pr ON gs.provider_id=pr.provider_id AND gs.gas_station_id=(SELECT MIN(gs.gas_station_id) FROM gas_station gs)
	INNER JOIN address ad ON gs.address_id = ad.address_id;	

	DBMS_OUTPUT.PUT_LINE('Die Tankstelle mit der kleinsten ID ' || v_gs_id);
	DBMS_OUTPUT.PUT_LINE('... ' || v_pn);
	DBMS_OUTPUT.PUT_LINE('... ' || v_plz);
	DBMS_OUTPUT.PUT_LINE('... ' || v_city);

	EXCEPTION
	  WHEN NO_DATA_FOUND
		THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde kein Benutzer gefunden');
	  WHEN OTHERS
		THEN DBMS_OUTPUT.PUT_LINE ('Folgender unerwarteter Fehler ist aufgetreten: ');
	  RAISE;
END;
/

//Cursor
DECLARE
	v_avg NUMBER;
	v_anzgs NUMBER;
	v_anzrec Number;
BEGIN
	SELECT COUNT(receipt_id) INTO v_anzrec
	FROM receipt;
	
	SELECT COUNT(gas_station_id)INTO v_anzgs
	FROM receipt;
	
	v_avg:=v_anzrec/v_anzgs;
	
	DBMS_OUTPUT.PUT_LINE (v_avg);

  FOR CUR_GS IN ( 
				SELECT gs.gas_station_id, pr.provider_name, ad.plz, ad.city, COUNT(re.receipt_id) AS "AZ"
				FROM gas_station gs
				INNER JOIN provider pr ON gs.provider_id=pr.provider_id
				INNER JOIN address ad ON gs.address_id = ad.address_id
				INNER JOIN receipt re ON gs.gas_station_id = re.gas_station_id
				GROUP BY gs.gas_station_id, re.receipt_id, pr.provider_name, ad.plz, ad.city	
				) LOOP
		
		IF (CUR_GS.AZ>v_avg) THEN
			DBMS_OUTPUT.PUT_LINE ('Gut Besucht! Gas Station: '||CUR_GS.gas_station_id||' '||CUR_GS.provider_name||' '||CUR_GS.plz||' '||CUR_GS.city);
		ELSE
			DBMS_OUTPUT.PUT_LINE ('Schlecht Besucht! Gas Station: '||CUR_GS.gas_station_id||' '||CUR_GS.provider_name||' '||CUR_GS.plz||' '||CUR_GS.city);
		END IF;
		
  END LOOP;
END;
/
```

### Aufgabe 3
Analysiere den untenstehenden anonymen PL/SQL-Code. Was macht er?
Passe den Codeblock so an, dass für jede Tankstelle alle Kunden die dort einmal tanken, waren ausgegeben werden.

```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE('Liste alle Tankstellen aus Deutschland');
  DBMS_OUTPUT.PUT_LINE('____________________________________________');
  FOR rec_gs IN (  SELECT p.provider_name, gs.street, a.plz, a.city, c.country_name
                    FROM gas_station gs
                      INNER JOIN address a ON (a.address_id = gs.address_id)
                      INNER JOIN provider p ON (gs.provider_id = p.provider_id)
                      INNER JOIN country c ON (gs.country_id = c.country_id)
                    WHERE c.country_name LIKE 'Deutschland') LOOP
    DBMS_OUTPUT.PUT_LINE('++ ' || rec_gs.provider_name || ' ++ ' || rec_gs.street || ' ++ ' || rec_gs.plz || ' ++ ' || rec_gs.city || ' ++ ' || rec_gs.country_name);
  END LOOP;
END;
/
```

#### Lösung
```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE('Liste alle Tankstellen aus Deutschland');
  DBMS_OUTPUT.PUT_LINE('____________________________________________');
  FOR rec_gs IN (  SELECT p.provider_name, gs.street, a.plz, a.city, c.country_name, gs.gas_station_id
                    FROM gas_station gs
                      INNER JOIN address a ON (a.address_id = gs.address_id)
                      INNER JOIN provider p ON (gs.provider_id = p.provider_id)
                      INNER JOIN country c ON (gs.country_id = c.country_id)
                    WHERE c.country_name LIKE 'Deutschland') LOOP
				
    DBMS_OUTPUT.PUT_LINE('++ ' || rec_gs.provider_name || ' ++ ' || rec_gs.street || ' ++ ' || rec_gs.plz || ' ++ ' || rec_gs.city || ' ++ ' || rec_gs.country_name);

	FOR cur_Kd IN (	SELECT acc.surname, acc.forename
					FROM receipt re
					INNER JOIN account acc ON re.account_id=acc.account_id
					AND re.gas_station_id = rec_gs.gas_station_id
					)LOOP
		DBMS_OUTPUT.PUT_LINE(cur_Kd.surname||' ++ '||cur_Kd.forename);
	END LOOP;
	
  END LOOP;
END;
/
```

### Aufgabe 4
Schreibe einen anonymen PL/SQL-Codeblock, der alle deine Fahrzeuge auflistet und die dazugehörigen Belege inkl. Betrag, der ausgegeben wurde für jeden Tankvorgang.

#### Lösung
```sql
/*Wieso kann eine receipt_id zwei vehicle ID haben?*/
SELECT acc.forename, acc.account_id, ve.vehicle_id, re.receipt_id
FROM receipt re
INNER JOIN account acc ON re.account_id=acc.account_id
INNER JOIN acc_vehic av ON acc.account_id=av.account_id
INNER JOIN vehicle ve ON av.vehicle_id=ve.vehicle_id;

DECLARE
BEGIN
  FOR cur_vh IN (  	SELECT pr.producer_name,ve.version, ve.vehicle_id
					FROM vehicle ve
					INNER JOIN acc_vehic av ON ve.vehicle_id = av.vehicle_id
					INNER JOIN producer pr ON ve.producer_id = pr.producer_id
					INNER JOIN account acc ON av.account_id = acc.account_id
					WHERE acc.forename = 'Max'
				) LOOP
			
    DBMS_OUTPUT.PUT_LINE('Fahrzeug: ' || cur_vh.producer_name || ' Version: ' || cur_vh.version);

	FOR cur_Beleg IN (	SELECT re.receipt_id
						FROM receipt re
						INNER JOIN vehicle ve ON ve.vehicle_id = cur_vh.vehicle_id
					 )LOOP
	
	DBMS_OUTPUT.PUT_LINE('Beleg: ' || cur_Beleg.receipt_id);
	
		FOR cur_price IN (	SELECT re.liter, re.price_l
							FROM receipt re
							WHERE re.receipt_id = cur_Beleg.receipt_id
						 )LOOP
		
		DBMS_OUTPUT.PUT_LINE('Rechnungsbetrag: ' || cur_price.liter*cur_price.price_l);
		
		END LOOP;
	END LOOP;
	
  END LOOP;
END;
/
```
