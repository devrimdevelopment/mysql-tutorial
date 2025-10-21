# MySQL-Tutorial: Deutsche Städte

## Vorbereitung

>1. Kopiere die Datei "german_cities.csv" in das folgende Verzeichnis:
C:\ProgramData\MySQL\MySQL Server 8.0\Uploads
>2. Starte MySQL Workbench CE

## Datenbank anlegen
>3. Gebe diesen Befehl ein:
```code
CREATE DATABASE deutsche_staedte
```
>3. Wählt unterhalb des Navigator-Fensters "Schemas" und dann das Aktualisieren-Symbol drücken.

![alt text](https://github.com/devrimdevelopment/mysql-tutorial/blob/main/image-1.png?raw=true)
#### Nun sollte die angelegte Datenbank sichtbar sein.
---
## Datenbank füttern
>4. Um mit der angelegten Datenbank arbeiten zu können, muss sie wie folgt zur Nutzung ausgewählt werden:
```code
USE deutsche_staedte
```
>5. Nun kann eine Table angelegt werden. Entsprechend der bereitgestellen .csv-Datei sollten auch gleich die Datentypen wie folgt definiert werden:
```code
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
);
```
>6. Schlussendlich kann auf die bereitgestellte .csv-Datei verwiesen werden, um dessen Inhalt in die Datenbank einzuspeisen:
```code
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
