# MySQL-Tutorial: Deutsche Städte
## Voraussetzungen
* MySQL Server 8.0 und MySQL Workbench CE müssen installiert sein
* Die Dateien german_cities.csv und lebendgeborene_2023.csv aus diesem Repository (siehe Datenquellen).


## Vorbereitung

>1. Kopiere die Datei "german_cities.csv" in das folgende Verzeichnis:
```plain
C:\ProgramData\MySQL\MySQL Server 8.0\Uploads
````
>2. Starte MySQL Workbench CE

## Datenbank anlegen
>3. Gebe diesen Befehl ein, um eine Datenbank zu erstellen:
```sql
CREATE DATABASE deutsche_staedte
```
>4. Wähle im Navigator-Fenster „Schemas“ und klicke auf das Aktualisieren-Symbol.

![alt text](https://github.com/devrimdevelopment/mysql-tutorial/blob/main/image-1.png?raw=true)
#### Nun sollte die angelegte Datenbank sichtbar sein.
---
## Datenbank füttern
>5. Um mit der angelegten Datenbank arbeiten zu können, muss sie wie folgt zur Nutzung ausgewählt werden:
```sql
USE deutsche_staedte
```
>6. Nun kann eine Table angelegt werden. Entsprechend der bereitgestellen .csv-Datei sollten auch gleich die Datentypen wie folgt definiert werden:
```sql
CREATE TABLE german_cities (
    id INT PRIMARY KEY,
    name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    district VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    state VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    population INT,
    lat FLOAT,
    lon FLOAT,
    area FLOAT
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
###### (Das Attribut **CHARACTER SET** ermöglicht mit der Kennung **utf8mb4** die saubere Darstellung von Umlauten, während das Attribut **COLLATE** eine robuste Sortierfunktionalität gewährleistet)
>7. Schlussendlich kann auf die bereitgestellte .csv-Datei verwiesen werden, um dessen Inhalt in die Datenbank einzuspeisen:
```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/german_cities.csv'
INTO TABLE german_cities
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(id, name, district, state, population, @lat, @lon, @area)
SET lat = NULLIF(@lat, ''),
    lon = NULLIF(@lon, ''),
    area = NULLIF(@area, '');
```
### Die Datenbank ist nun bereit.
- - - 
#### Beispielabfrage:
```sql
-- Wählt die Top 10 der bevölkerungsreichsten Städte in Nordrhein-Westfalen aus --
SELECT name, district, population FROM german_cities
WHERE state = 'Nordrhein-Westfalen'
ORDER BY population DESC
LIMIT 10;
```
#### Syntaxerklärung:
| SQL-Teil | Bedeutung |
| :--- | :--- |
| `SELECT name, population` | Wählt die anzuzeigenden Spalten aus. |
| `FROM german_cities` | Gibt die Zieltabelle an. |
| `WHERE state = 'Nordrhein-Westfalen` | Filtert die Ergebnisse auf das Bundesland Nordrhein-Westfalen.|
| `ORDER BY population` | Sortiert die Ergebnisse basierend auf der Spalte `population`. |
| `DESC` | **DESCending** (absteigend), d.h., die größten Werte (höchste Bevölkerung) stehen oben. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |
---
---
# Der JOIN-Befehl
Mit JOIN verbindet man Daten aus zwei oder mehr Tabellen in deiner Datenbank, um sie in einem einzigen Ergebnisdatensatz abzufragen und anzuzeigen.
Zu diesem Zweck werden wir nun eine weitere Tabelle im Workbench anlegen:
>8. Kopiere die Datei "lebendgeborene_2023.csv" in das folgende Verzeichnis: C:\ProgramData\MySQL\MySQL Server 8.0\Uploads
>9. Gebe diesen Befehl ein:
```sql
CREATE TABLE lebendgeborene_2023 (
    ags_code VARCHAR(8) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL PRIMARY KEY,
    region_name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    clean_name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    region_typ VARCHAR(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    lebendgeborene VARCHAR(10)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
#### Nun müsste nach Betätigen des Buttons zum Aktualisieren eine weitere Tabelle sichtbar sein. Diese muss nun mit Daten gefüttert werden.
>10. Dafür sollte nun folgender Befehlsfolge ausgeführt werden.
```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/lebendgeborene_2023.csv'
INTO TABLE lebendgeborene_2023
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(ags_code, region_name, clean_name, region_typ, lebendgeborene);
```

---
#### Beispielabfrage mit INNER JOIN:
```sql
-- Geburten in kreisfreien Städten --
SELECT g.name, g.state, g.population, l.lebendgeborene
FROM german_cities g
INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE l.region_typ = 'kreisfreie Stadt'
ORDER BY l.lebendgeborene DESC
LIMIT 10;
```
#### Syntaxerklärung:
| SQL-Teil | Bedeutung |
|----------|-----------|
| `SELECT g.name, g.state, g.population, l.lebendgeborene` | Wählt die Spalten `name`, `state`, `population` aus der Tabelle `german_cities` und `lebendgeborene` aus `lebendgeborene_2023` für die Anzeige aus. |
| `FROM german_cities g` | Gibt die erste Zieltabelle `german_cities` an, mit dem Alias `g` für kürzere Schreibweise. |
| `INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name` | Verknüpft `german_cities` mit `lebendgeborene_2023` (Alias `l`), indem nur Zeilen angezeigt werden, wo der Stadtname (`g.name`) mit `l.clean_name` übereinstimmt. |
| `WHERE l.region_typ = 'kreisfreie Stadt'` | Filtert die Ergebnisse, sodass nur Zeilen mit `region_typ = 'kreisfreie Stadt'` aus `lebendgeborene_2023` berücksichtigt werden. |
| `ORDER BY l.lebendgeborene DESC` | Sortiert die Ergebnisse absteigend (`DESC`) nach der Spalte `lebendgeborene`, sodass Städte mit den meisten Geburten oben stehen. |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen, um nur die Top 10 Städte anzuzeigen. |


#### Beispielabfrage mit INNER JOIN:
```sql
-- Geburtenrate pro 1.000 Einwohner --
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000
FROM german_cities g
INNER JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```
#### Syntaxerklärung:
| SQL-Teil | Bedeutung und Funktion |
| :--- | :--- |
| **`SELECT g.name, g.state, ... AS distance_km`** | Wählt Spalten, berechnet die Geburtenrate (`geburten_pro_1000`) und die **Entfernung zu München** (`distance_km`) in Kilometern. |
| **`6371`** | Die **Konstante für den Erdradius** in **Kilometern**. Multipliziert mit dem Ergebnis der ACOS-Funktion ergibt sich die Entfernung. |
| **`RADIANS(Wert)`** | Wandelt die geografischen Koordinaten (Grad) in **Radiant** um (notwendig für trigonometrische Funktionen in SQL). |
| **`COS(radians(48.137154))`** | Die **festen Koordinaten des Zielpunkts (München)** werden für die Berechnung verwendet. |
| **`ACOS(...)`** | Die **Arcus-Cosinus-Funktion**. Sie berechnet den Winkel zwischen den beiden Punkten auf der Kugeloberfläche. |
| **`FROM german_cities g LEFT JOIN ...`** | Die linke Zieltabelle ist `german_cities g`. Der **LEFT JOIN** behält alle Städte bei, auch wenn keine Geburten-Daten vorhanden wären (was durch den WHERE-Filter aber ausgeschlossen wird). |
| **`WHERE g.state = 'Bayern' AND ... AND (...) < 100`** | **Filterung:** Schließt nur Städte in **Bayern** ein, die **<100 km** von München entfernt sind und Geburten-Daten haben. |
| **`ORDER BY geburten_pro_1000 DESC`** | Sortiert **absteigend** nach der Geburtenrate. |
| **`LIMIT 10`** | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |

#### Beispielabfrage mit LEFT JOIN:
```sql
-- Top 10 kreisfreie Städte in Bayern nach Geburtenrate --
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000
FROM german_cities g
LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE g.state = 'Bayern' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```
#### Syntaxerklärung:
| SQL-Teil | Bedeutung |
|----------|-----------|
| `SELECT g.name, g.state, g.population, l.lebendgeborene, ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000` | Wählt die Spalten `name`, `state`, `population` aus `german_cities`, `lebendgeborene` aus `lebendgeborene_2023` und berechnet die Geburtenrate pro 1.000 Einwohner, gerundet auf zwei Dezimalstellen, als `geburten_pro_1000`. |
| `FROM german_cities g` | Gibt die erste Zieltabelle `german_cities` an, mit dem Alias `g` für kürzere Schreibweise. |
| `LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name` | Verknüpft `german_cities` mit `lebendgeborene_2023` (Alias `l`), indem alle Städte aus `german_cities` angezeigt werden, auch wenn keine Geburten-Daten vorliegen (`l.lebendgeborene` kann `NULL` sein). Verknüpfung über `g.name = l.clean_name`. |
| `WHERE g.state = 'Bayern' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL` | Filtert auf Städte in Bayern (`g.state = 'Bayern'`), kreisfreie Städte (`l.region_typ = 'kreisfreie Stadt'`) und stellt sicher, dass Geburten-Daten vorhanden sind (`l.lebendgeborene IS NOT NULL`). |
| `ORDER BY geburten_pro_1000 DESC` | Sortiert die Ergebnisse absteigend (`DESC`) nach der berechneten Geburtenrate (`geburten_pro_1000`). |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen, um die Top 10 Städte anzuzeigen. |

#### Beispielabfrage mit mit RIGHT JOIN:
```sql 
-- Top 10 kreisfreie Städte in Hessen nach Geburtenrate --
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000
FROM lebendgeborene_2023 l
RIGHT JOIN german_cities g ON l.clean_name = g.name
WHERE g.state = 'Hessen' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```
#### Syntaxerklärung:
| SQL-Teil | Bedeutung |
|----------|-----------|
| `SELECT g.name, g.state, g.population, l.lebendgeborene, ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000` | Wählt die Spalten `name`, `state`, `population` aus `german_cities`, `lebendgeborene` aus `lebendgeborene_2023` und berechnet die Geburtenrate pro 1.000 Einwohner, gerundet auf zwei Dezimalstellen, als `geburten_pro_1000`. |
| `FROM lebendgeborene_2023 l` | Gibt die erste Zieltabelle `lebendgeborene_2023` an, mit dem Alias `l`. |
| `RIGHT JOIN german_cities g ON l.clean_name = g.name` | Verknüpft `lebendgeborene_2023` mit `german_cities` (Alias `g`), indem alle Städte aus `german_cities` angezeigt werden, auch wenn keine Geburten-Daten vorliegen (`l.lebendgeborene` kann `NULL` sein). Verknüpfung über `l.clean_name = g.name`. |
| `WHERE g.state = 'Hessen' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL` | Filtert auf Städte in Hessen (`g.state = 'Hessen'`), kreisfreie Städte (`l.region_typ = 'kreisfreie Stadt'`) und stellt sicher, dass Geburten-Daten vorhanden sind (`l.lebendgeborene IS NOT NULL`). |
| `ORDER BY geburten_pro_1000 DESC` | Sortiert die Ergebnisse absteigend (`DESC`) nach der berechneten Geburtenrate (`geburten_pro_1000`). |
| `LIMIT 10` | Begrenzt die Ausgabe auf die ersten 10 Zeilen, um die Top 10 Städte anzuzeigen. |
---
---
## Fortgeschrittene Beispielabfrage: Geografischer Radius
```sql
-- Geburtenrate in kreisfreien Städten Bayerns im Umkreis von 100 km um München
SELECT g.name, g.state, g.population, l.lebendgeborene,
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000,
       ROUND(
           6371 * acos(
               cos(radians(48.137154)) * cos(radians(g.lat)) * 
               cos(radians(g.lon) - radians(11.576124)) + 
               sin(radians(48.137154)) * sin(radians(g.lat))
           ), 2
       ) AS distance_km
FROM german_cities g
LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE g.state = 'Bayern' 
  AND l.region_typ = 'kreisfreie Stadt' 
  AND l.lebendgeborene IS NOT NULL
  AND (
      6371 * acos(
          cos(radians(48.137154)) * cos(radians(g.lat)) * 
          cos(radians(g.lon) - radians(11.576124)) + 
          sin(radians(48.137154)) * sin(radians(g.lat))
      )
  ) < 100
ORDER BY geburten_pro_1000 DESC
LIMIT 10;
```
## Syntaxerklärung: Haversine-Formel für die Distanzberechnung
Die Haversine-Formel wird zur Berechnung der **kürzesten Entfernung zwischen zwei Punkten auf einer Kugeloberfläche** verwendet. Sie berechnet hier die Distanz jeder Stadt (`g.lat`, `g.lon`) zum Zentrum von München.

| SQL-Teil | Bedeutung und Funktion |
| :--- | :--- |
| **`SELECT g.name, g.state, g.population, l.lebendgeborene, ROUND(...) AS geburten_pro_1000, ROUND(...) AS distance_km`** | Wählt Spalten, berechnet die Geburtenrate pro 1.000 Einwohner und die Entfernung zu München (in km). |
| **`6371`** | Die **Konstante für den Erdradius** in **Kilometern**. Sie wird am Ende mit dem errechneten Winkel multipliziert. |
| **`6371 * acos(...)`** | Berechnet die Entfernung zwischen München (48.137154, 11.576124) und der Stadt (g.lat, g.lon) mit der Haversine-Formel. |
| **`FROM german_cities g`** | Gibt die Zieltabelle german_cities an, mit Alias g. |
| **`WHERE g.state = 'Bayern' AND l.region_typ = 'kreisfreie Stadt' AND l.lebendgeborene IS NOT NULL AND (...) < 100`** | Filtert auf Bayern, kreisfreie Städte, vorhandene Geburten-Daten und Städte <100 km von München. |
| **`RADIANS(Wert)`** | Wandelt die geografischen Koordinaten (Grad) in **Radiant** um. Dies ist notwendig für die trigonometrischen Funktionen (`COS`, `SIN`) in SQL. |
| **`COS(radians(48.137154))`** | Die festen Koordinaten des Zielpunkts (**München**). Sie werden in Radiant umgewandelt und in die Formel integriert. |
| **`COS(radians(g.lat))`** | Die dynamische Koordinate (Breitengrad der aktuellen Stadt aus `german_cities`). |
| **`ACOS(...)`** | Die **Arcus-Cosinus-Funktion**. Sie berechnet den Winkel zwischen den beiden Punkten auf der Kugeloberfläche. |
| **`... AS distance_km`** | Speichert das Ergebnis der gesamten komplexen Berechnung (`6371 * ACOS(...)`) als neue Spalte **`distance_km`**. |
| **`AND (...) < 100`** | Die gleiche komplexe Berechnung wird als **Filterbedingung** in der `WHERE`-Klausel verwendet, um nur Städte anzuzeigen, deren Distanz **kleiner als 100 km** ist. |
| **`ORDER BY geburten_pro_1000 DESC`** | Sortiert absteigend nach Geburtenrate.
| **`LIMIT 10`** | Begrenzt die Ausgabe auf die ersten 10 Zeilen. |

### Fortgeschrittene Beispielabfrage 2
```sql
-- Top 5 kreisfreie Städte nahe Berlin mit Geburtenrate ≥ 8
SELECT g.name, g.state, g.population, l.lebendgeborene,
       -- 1. Berechnung der Geburtenrate pro 1.000 Einwohner
       ROUND((l.lebendgeborene / g.population) * 1000, 2) AS geburten_pro_1000,
       -- 2. Berechnung der Entfernung von Berlin in Kilometern (Haversine-Formel)
       ROUND(
           6371 * acos(
               cos(radians(52.520008)) * cos(radians(g.lat)) * cos(radians(g.lon) - radians(13.404954)) + 
               sin(radians(52.520008)) * sin(radians(g.lat))
           ), 2
       ) AS distance_km
FROM german_cities g
LEFT JOIN lebendgeborene_2023 l ON g.name = l.clean_name
WHERE l.region_typ = 'kreisfreie Stadt' 
  AND l.lebendgeborene IS NOT NULL
  -- 3. Filterung: Schränkt die Ergebnisse auf Städte mit Rate >= 8 ein
  AND (l.lebendgeborene / g.population) * 1000 >= 8 
ORDER BY distance_km ASC
LIMIT 5;
```
#### Erweiterte Syntaxerklärung zur Abfrage

Diese Abfrage filtert Städte nach einer hohen Geburtenrate und sortiert das Ergebnis nach der geografischen Entfernung zu Berlin.
| SQL-Teil | Bedeutung und Funktion |
| :--- | :--- |
| **`Feste Koordinaten`** | Die in der Haversine-Formel verwendeten Werte 52.520008 (Lat) und 13.404954 (Lon) definieren den Ausgangspunkt Berlin. Die Distanz wird von jeder Stadt zu diesem festen Punkt berechnet. |
| **`SELECT g.name, g.state, ... AS distance_km`** | Wählt die anzuzeigenden Spalten aus. Es werden die berechneten Felder **`geburten_pro_1000`** (Rate) und **`distance_km`** (Entfernung) als separate Spalten in den Ergebnisdatensatz aufgenommen. |
| **`6371 * acos(...)`** | **Haversine-Formel:** Berechnet die kürzeste Entfernung in **Kilometern** zwischen dem Zentrum von Berlin (`52.520008` (Lat), `13.404954` (Lon)) und jeder Stadt in der Tabelle (`g.lat`, `g.lon`). Die Konstante `6371` ist der Erdradius in km. |
| **`FROM german_cities g LEFT JOIN ...`** | **Verknüpfung:** Die Zieltabelle ist `german_cities g`. Der **LEFT JOIN** verknüpft diese mit `lebendgeborene_2023 l`, wobei alle Städte aus `german_cities` als potenziell beibehaltene Zeilen betrachtet werden. |
| **`WHERE ... AND (l.lebendgeborene / g.population) * 1000 >= 8`** | **Statistische Filterung:** Stellt sicher, dass nur **kreisfreie Städte** mit **vorhandenen Geburten-Daten** in das Ergebnis aufgenommen werden, deren Geburtenrate **größer oder gleich 8** pro 1.000 Einwohner ist. |
| **`ORDER BY distance_km ASC`** | **Sortierrichtung:** Sortiert das Ergebnis **aufsteigend (`ASC`)** nach der berechneten Entfernung zu Berlin. Die geografisch nächstgelegenen Städte erscheinen somit zuerst. |
| **`LIMIT 5`** | **Begrenzung:** Beschränkt die Ausgabe auf die **fünf** Städte, die die Kriterien erfüllen und Berlin am nächsten liegen. |

## Datenquellen und Lizenz

Die Rohdaten für dieses Tutorial stammen vom deutschen **Open-Data-Portal GovData.de**.

Die Daten auf GovData werden typischerweise unter einer der folgenden Lizenzen bereitgestellt:

* **DL-Zero ("Datenlizenz Deutschland – Zero – Version 2.0")**: Die Daten dürfen **uneingeschränkt** genutzt werden, ohne dass ein Quellhinweis erforderlich ist (obwohl dieser im Sinne der Transparenz empfohlen wird).
* **DL-DE/BY ("Datenlizenz Deutschland – Namensnennung – Version 2.0")**: Die Daten dürfen genutzt werden, jedoch **muss die Quelle genannt werden**.

**Hinweis zu den CSV-Dateien:**
Die im Repository bereitgestellten CSV-Dateien (`german_cities.csv` und `lebendgeborene_2023.csv`) wurden mit einem Skript **angepasst und bereinigt**, um sie für dieses MySQL-Tutorial optimal nutzbar zu machen.

---
