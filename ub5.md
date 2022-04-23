

# Relationale Datenbanksprache SQL

## DDL und DML


* [1) SQL Datentypen](#sql-datentypen)

* [2)  Create Table,Primary Key, Foreign Key](#create-table)

* [3) Alter Table, Drop Table](#alter-table)

* [4) Create Index, Create View](#create-ndex)

* [5) Data Manipulation Languange](#data-manipulation-languange-dml)

* [6) Geschachtelte Anfragen mit EXISTS, IN, ALL, ANY Klauseln ](#geschachtelte-anfragen-mit-exsts-n-all-any-klauseln)
 
* [7) Aggregatfunktionen mit AVG(), COUNT(), MIN(), MAX(), und SUM()](#aggregatfunktionen-mit-avg-count-mn-max-und-sum)

* [8) Joins: INNER -, RIGHT -, LEFT -, FULL [OUTER] JOIN ](#joins-nner---rght---left---full-outer-jon)

* [9) Änderungsoperationen: Einfügen, Löschen und Verändern](#nderungsoperationen-einfgen-lschen-und-verndern)
---
###Datenbank starten

``` python
from IPython.display import Markdown, display
path = "assets/_pgdata"
try:
    running_tests
except NameError:
    import os.path
    #!rm -rf ~/.pgdata
    if not os.path.exists(path):
        display(Markdown("# Datenbank wird initialisiert."))
        display(Markdown("### Datenbank wird extrahiert."))
        !tar -zx --touch --checkpoint=.50 -f assets/pgdata.tar.gz -C assets/
        # !pg_ctl -D $path initdb
        display(Markdown("### Datenbank initialisiert"))
    !chmod 700 $path
    display(Markdown("# Server wird (neu)gestartet."))
    if os.path.exists(path + "/postmaster.pid"):
        !pg_ctl -D $path restart
        display(Markdown("### Datenbank restart OK"))
    else:
        !pg_ctl -D $path start
        display(Markdown("### Datenbank start OK, verbinde..."))
display(Markdown("#### Einrichtung der Übungsdatenbank"))
!psql -c "DROP DATABASE IF EXISTS exercise_sheet;" postgres
!psql -c "CREATE DATABASE exercise_sheet;" postgres
display(Markdown("#### Verbindung mit Datenbank wird hergestellt..."))
%reload_ext sql
%sql postgresql://localhost/exercise_sheet
%sql ABORT;--NOOP
check=!pg_isready -h localhost -d exercise_sheet -q;echo $?
if check[0]=='0':
    # x = %sql SELECT 'OK' AS "status"
    display(x)
    display(Markdown("# Server OK! Es kann los gehen!"))
else:
    display(Markdown("# Server nicht OK! Versuche, den Kernel neu zu starten und die Zelle erneut auszuführen."))
```

###Datenbankschema initialisieren

``` python
from IPython.display import Markdown, display, Code
from urllib.request import urlopen
try:
    display(Markdown("#### Herunterladen der Schema Datei"))
    # url = '/home/jovyan/schema.sql'
    url = 'https://git.rwth-aachen.de/i5/teaching/dbis-raw/-/raw/main/schema.sql'
    url_open = urlopen(url)
    if ( url_open.getcode() == 200 ):
        display(Markdown("HTTP 200 OK"))
    schema = url_open.read().decode('utf-8')
except urllib.error.HTTPError as e:
    error = body = e.read().decode()
    display(Markdown("# Fehler: "))
    print ( error )
# print(schema)
display(Markdown("#### Schema import"))
!psql -c "$schema" exercise_sheet
products_exists = %sql SELECT TRUE FROM pg_attribute WHERE attrelid = 'products'::regclass AND attname = 'product_name'
if ( products_exists ):
    display(Markdown("# IMPORT OK! Es kann los gehen!"))
else:
    display(Markdown("# Fehler!"))
```

###  SQL Datentypen 

In einer Datenbank muss jede Spalte einen Namen und einen **Datentyp** haben. In jeder Spalte haben die Attributwerte einen bestimmten Datentyp.
#### Zeichentypen
- `CHAR(size)` or `CHARACTER(size)`             : Strings mit der festen Länge n
- `VARCHAR(size)` or `CHARACTER VARYING(size)`  : Strings mit der variablen Länge ( speichert Max. 255 Byte)
- `Text`                                        : unbegrenzt
#### Numerische Typen
- `INTEGER` or `INT` 
- `DECIMAL`
- `NUMERIC`
- `SERIAL` : Autoincrementing-Ganzzahl
---


##  <a name="CREATE TABLE"></a> CREATE TABLE

**Snytax:**
- CREATE TABLE <_Relationenname_> (_<_Spaltendefinition_>{, <_Spaltendefinition_>}_)
mit <_Spaltendefinition_> ::= <_Attributname_> <_Typ_> {<_Option_>}
und <_Option_> ::= DEFAULT <_Ausdruck_> | NOT NULL | UNIQUE | PRIMARY KEY | ...

**Beispiel 1:**
- Ein Relation "Student" im Graphen anlegen
- Jeder Student hat eine Name, Semester und Adresse
- Matrikelnummer besteht aus 6 Ziffern, Semester ist eine 2-stelliger Zahl, Adresse hat maximal die Laenge 50, Name hat maximal die Lange 20

``` python .noeval
CREATE TABLE Student( Name CHAR (20) NOT NULL, Semester INTEGER(2), Adresse VARCHAR (50))'''
```

Als ausführbarer Code:
``` python
 from IPython.display import display, Code
create_table_query = '''
    CREATE TABLE Student
    ( Name CHAR (20) NOT NULL,
    Semester INTEGER(2), 
    Adresse VARCHAR (50))
'''
# Ausführen des Befehls
create_table_test = %sql $create_table_query
# Ausgabe des Befehls mit Syntax-Highlighting
display(Code(create_table_query, language='sql'))
# Ausgabe der Ergebnisse
display(create_table_test)            
``` 
---
##PRIMARY KEY

Ein "Primary Key" ist eine Spalte oder eine Menge von Spalten, die jede Zeile eindeutig identifizieren lässt.
Spalten als Primary Key können beim anlegen (link to create) oder ändern (link to alter) einer Relation im Graphen festgelegt werden.

`Primary Key` ist auch eine _Einschränkung_ (link) und lässt sich auch als Kombination der `NOT NULL` und `UNIQUE` _Einschräankungen_ nachbilden

**Beispiel für _eindeutiger_ Pimary Key:**

- Studenten lassen sich mit ihrer 6-stelliger Matrikelnummer eindeutig identifizieren.

Da wir die "Student"-Relation schon im vorherigen Abschnitt angelegt haben, 
müssen wir eine änderung mit "ALTER TABLE" durchführen, um die Matrikelnummer als Primary Key hinzuzufügen.
Diese lernen wir erst im folgenden Kapitel.

Wäre die Relation "Student" nicht schon angelegt, 
dann hätten wir den Matrikelnummer als Primary Key beim anlegen der Relation wie folgt festlegen können:
``` python .noeval
CREATE TABLE Student (MatrNum INTEGER(6) PRIMARY KEY)
```


**Beispiel für _zusammengesetzter_ primary key:**
- Vorlesungen lassen sich mit Vorlesungsname und Jahr eindeutig identifizieren:
``` python .noeval
CREATE TABLE Vorlesung (VName VAR(20), Jahr INTEGER(4), PRIMARY KEY(VName, Jahr))
```

Als ausführbarer Code:
``` python
 from IPython.display import display, Code
    query = '''
    CREATE TABLE Vorlesung (VName VAR(20), 
    Jahr INTEGER(4), PRIMARY KEY(VName, Jahr))
    '''
# Ausführen des Befehls
query_test = %sql $query
# Ausgabe der Ergebnisse
display(query)            
``` 
---

##ALTER TABLE
Mit "ALTER TABLE" kann eine schon angelegte Relation modifiziert werden. Damit kann eine Spalte zur Relation hinzüfügt
oder gelöscht werden.

**neue Spalte hinzufügen:**
- Studenten lassen sich mit ihrer 6-stelliger Matrikelnummer eindeutig identifizieren.
``` python
from IPython.display import display, Code
 
    query = '''
    ALTER TABLE Student 
    ADD COLUMN MatrNum INT(6) PRIMARY KEY
    '''

query_test = %sql $query
display(query_test)            
``` 
**Eine Spalte löschen**
- Adressen der Studenten sollen nicht mehr gespeichert werden
``` python
from IPython.display import display, Code
 
    query = '''
    ALTER TABLE Student 
    DROP COLUMN Adresse
    '''

query_test = %sql $query
display(query_test)            
```
---
##FOREIGN KEY

Ein _Foreign Key_ ist eine Spalte oder eine Menge von Spalten in der Relation, 
die sich auf den Primary Key einer anderer Relation beziehen und somit die 2 Relationen verknüpfen.

**Beispiel:**
- Studenten können Vorlesungen in einem Jahr belegen und können eine Note bekommen.
``` python
from IPython.display import display, Code
 
    query = '''
    CREATE TABLE Belegung (MatrNum INTEGER(6), VName VAR(20), Jahr INT(4), Note DECIMAL(2),
    FOREIGN KEY (MatrNum) REFERENCES Student (MatrNum))
    '''

query_test = %sql $query
display(query_test)            
```
---
##DROP TABLE
Damit kann mann die Relationen im Graphen löschen.

- Lösche die Relation "Professor":
``` python
from IPython.display import display, Code
 
    query = '''
    DROP TABLE Professor
    '''

query_test = %sql $query
display(query_test)            
```
Da Diese Relation nicht im Graph existiert, liefert PostgreSQL ein Error. 
Dieses kann man wie folgt vermeiden:
``` python
from IPython.display import display, Code
 
    query = '''
    DROP TABLE IF EXISTS Professor
    '''

query_test = %sql $query
display(query_test)            
```
---
##CREATE INDEX
Falls kein Index angelegt ist, erfolgt die Suche nach Informationen innerhalb einer Relation sequentiell.
Indexe ermöglichen bei der Suche das Nutzen von Datenstrukturen wie B+ Bäume um den Zugriff zu beschleunigen.

**Syntax:**
>CREATE [UNIQUE] INDEX 
<_Indexname_>
ON <_Relationenname_> (<_Attributname_> [<_Ordnung_>]{, <_Attributname_> [<_Ordnung_>]})

wobei  <_Ordnung_> ::= 
ASC | DESC

>DROP INDEX <_Indexname_>


**Ein "SELECT" Beispiel aus dem Datenbank:**
- Finde die Bestellung "orders" welche an Timestamp '2020-8-16 10:31:23+01' "order_created_at" erstellt wurde.

**ohne Index:**

Hier nutzen wir "EXPLAIN"-Klausel um kosten für das "SELECT" Operation zu sehen
``` python
from IPython.display import display, Code
 
    query = '''
    EXPLAIN SELECT * FROM order WHERE order_created_at = '2020-8-16 10:31:23+01'
    '''

query_test = %sql $query
display(query_test)            
```
**mit index:**

Hier wird ein index Namens "time_index" für das "order_created_at"-Attribut der Relation "order" angelegt, 
um die Suche zu beschleunigen:
``` python
from IPython.display import display, Code
 
    query = '''
    CREATE INDEX time_index ON order(order_created_at)
    EXPLAIN SELECT * FROM order WHERE order_created_at = '2020-8-16 10:31:23+01'
    '''

query_test = %sql $query
display(query_test)            
```

Vergleiche kosten aus beiden Ausgaben

---
##CREATE VIEW
Erstellt eine Abfrage auf den Graphen und gibt es eine Name. 
Später können weitere Abfragen auf dieser View verweisen. 
Für den erstellten View wird keine Relation angelegt, 
sondern wird die gespeicherte Abfrage für jeden Verweis neu durchgeführt.

**Snytax:**
>CREATE VIEW <_Sichtname_> [(<_Attributname_>{, <_Attributname_>})] AS <_Subquery_>

>DROP VIEW <_Sicht -Name_>

**Beispiel:**
- Erstelle einen View, der die schon bezahlten 'Paid' Bestellungen "order" zurückgibt.
```python .noeval
CREATE VIEW paid_orders AS
SELECT * FROM orders WHERE order_status = 'Paid'
```
- Gib den Namen der Läden "store_name" von schon bezahlten 'Paid' Bestellungen an.
Nutze dabei das erstellte View.
```python .noeval
SELECT s.store_name AS "Store"
FROM paid_orders o
INNER JOIN stores s on s.store_id = o.order_store_id
```
- Gib den Namen der Käufer "user_full_name" von schon bezahlten 'Paid' Bestellungen an. 
Nutze dabei das erstellte View und lösche diese am Ende.
```python .noeval
SELECT u.user_full_name AS "Name"
FROM paid_orders o
INNER JOIN users u on u.user_id = o.order_store_id;

DROP VIEW paid_orders
```
Ist dasselbe wie:
```python .noeval
SELECT u.user_full_name AS "Name",
s.store_name AS "Store"
FROM orders o
INNER JOIN users u on u.user_id = o.order_user_id
INNER JOIN stores s on s.store_id = o.order_store_id
WHERE o.order_status = 'Paid'
```
---
## Data Manipulation Languange (DML)

DML beschäftigt sich mit allen Typen von Anfragen in SQL-Sprache. 
>Die Grundform ist wie folgt definiert: `SELECT`<_Liste von Attributsnamen_>
>                                        `FROM` <_ein oder mehrere Relationennamen_>
>                                        [`WHERE` <_Bedingung_>

In Bezug auf relationalen Algebra, wird mit `SELECT` - Klausel die Anfrage projektiert. Die Relationen, die für die Anfrage benötigt wird, werden durch `FROM`-Klausel dargestellt. Hier entspricht das `FROM` -Klausel das Kreuzprodukt von Relationen. Und das `WHERE`- Klausel beschreibt die Bedingungen für die Selektion.

**Beispiel:**
- Geben Sie die country ID von Deutschland.
``` python .noeval
    SELECT country_id
    FROM countries
    WHERE country_name = 'Germany'
```
---
###WHERE -Klausel
Nach dem `WHERE` können die folgenden Vergleichsoperatoren vorkommen:

- logische Verknüpfungen : AND ,OR, NOT
- arithmetische Operatoren : +, -, *, /, >, <, =
- Mengen Operatoren : IN, NOT IN, ANY, ALL, EXISTS or BETWEEN, LIKE, IS NULL

**Beispiel:**

 - Geben Sie den Namen der Produkte an, die in noch keinem Geschäft angeboten werden.
```   
    SELECT
    product_name AS "Product Name"
    FROM store_stock RIGHT JOIN products ON store_stock.product_id = products.product_id
    WHERE store_id IS NULL
```
**Beispiel:**
- Geben Sie die Produktnamen von Bücher, deren Preis größer als 15 euro sind.
```
  SELECT product_name AS "Product Name"
  FROM  products
  WHERE product_price > 15
  AND product_type = "Book"
```

---
### SELECT FROM- Klausel

>`SELECT DISTINCT` - Klausel nimmt nur die unterschiedlichen Variablen. Die Duplikaten werden nicht angenommen.
- Geben Sie die alle Geschäftsnamen in Asien.
 ```
 SELECT DISTINCT store_name 
 FROM countries NATURAL JOIN stores ON stores.store_country = countries.country_id
 WHERE continent = "Europe"
 ```


> `SELECT* FROM`- Klausel projektiert alle Spalten in der Relation.
- Geben Sie die alle Produkte.
```
SELECT* FROM products
```


### SQL vs. Relationale Algebra 

Man kann die Anfragen aus der relationalen Algebra auch mit SQL ausdrücken.

| Relationale<br/>Algebra | SQL                      |
|-------------------------|--------------------------|
| Vereiningung            | `UNION`                  |
| Differenz               | `EXCEPT`                 |
 | Kreuzprodukt           | `CROSS JOIN`             |
 | Selektion              | `WHERE`                  |
|Projektion               | `SELECT DISTINCT`        |
 | Umbenennung            | ` AS`                    |


---
##Geschachtelte Anfragen mit EXISTS, IN, ALL, ANY Klauseln
Diese sind Boolean Operatoren. Man kann in WHERE-Klausel mittels dieser Operatoren auch Unterabfragen erstellen.

###1- EXISTS
>Syntax: EXISTS(<_Subquery_>)

Testet Falls es eine Zeile in **Subquery** existiert


**Beispiel:**
- Namen der Läden (stores), deren Lagerbestand(current_stock) größer als 50 für ein Produkt ist.
``` python
from IPython.display import display, Code
 
    query = '''
    SELECT store_name
    FROM stores st
    WHERE EXISTS
        (SELECT 1 FROM store_stock ss 
        WHERE ss.store_id = st.store_id 
        AND current_stock > 50)
    '''

query_test = %sql $query
display(query_test)            
```
###2- IN
>Syntax: X IN (<_Subquery_>)

Testet, ob X einer der Werte aus **Subquery** ist.

**Beispiel:**
- Gib den Namen der Käufern (users), deren Bestellung bezahlt (Paid) oder versandt (Shipped) ist.
``` python
from IPython.display import display, Code
 
    query = '''
    SELECT user_full_name
    FROM users
    WHERE user_id IN
        (SELECT order_user_id
         FROM orders
         WHERE order_status = 'Paid' OR order_status = 'Shipped')
    '''

query_test = %sql $query
display(query_test)            
```
**Beispiel 2:**
- id der Bestellungen, deren status bezahlt oder versandt ist.
```python .noeval
SELECT order_id
FROM orders
WHERE order_status IN ('Paid', 'Shipped')
```
###3- ALL
>Syntax: X <_op_> ALL (<_Subquery_>)

Testet, ob alle Werte der Subquery die Voraussetzung erfüllen. op ist einer der Vergleichsoperatoren =, !=, <=, <, >=, >. 


**Beispiel:**
- Wie viel kostet das teuerste Produkt?
```python .noeval
SELECT product_name
FROM products
WHERE product_price >= ALL (SELECT product_price FROM products)
```
###4- ANY
>Syntax: X <_op_> ANY (<_Subquery_>)

Testet, ob ein Wert aus der Subquery die Voraussetzung erfüllt. 
op ist einer der Vergleichsoperatoren =, <>, <=, <, >=, >. 

X ANY = (<_Subquery_>) ist dasselbe wie X IN (<_Subquery_>)

---
##Aggregatfunktionen mit AVG(), COUNT(), MIN(), MAX(), und SUM().

Diese Funktionen führen eine Berechnung über eine Menge von Zeilen durch und liefern 
das Ergebnis als eine einzige Zeile zurück. 

Sie werden am meisten mit dem **GROUP BY**-Klausel innerhalb einer **SELECT**-Kalusel genutzt.
Nachdem der **GROUP BY**-Klausel das Ergebnis als Mengen von Zeilen liefert, wird auf jeder Ergebnismenge mit der Aggregatfunktion 
eine Berechnung durchgeführt.

###1- AVG()

###2- COUNT()

###3- MIN(), MAX()

###4- SUM()

---
##Gruppieren und Sortieren mit GROUP BY, HAVING, ORDER BY

Die allgemeine Syntax ist :
```
SELECT [...]
FROM [...]
WHERE[...]
GROUP BY <Liste von Atrributsnamen>
HAVING <Bedingung>
ORDER BY <Liste von Atrributsnamen>
```
###GROUP BY
Um eine Ergebnismenge zu gruppieren wird die **GROUP BY**- Klausel benutzt. Die Ergebnisrelation enthält ein Tupel für jede Gruppe. 

**Beispiel:**
- Geben Sie die Anzahl von Geschäften im jeden Land.
```
SELECT COUNT(store_id) AS NumberOfStores, store_country
FROM store
GROUP BY store_country
```


###HAVING
Die **HAVING**- Klausel gibt die Bedingung nach der Gruppierung.

**Beispiel:**
- Geben Sie die Anzahl von Geschäften im jeden Land, deren ID-Nummer größer als 50 sind.
```
  SELECT COUNT(store_id) AS NumberOfStores, store_country
  FROM store
  GROUP BY store_country
  HAVING COUNT(store_id) >50
```
Man kann nach der **HAVING**- Klausel alle Aggregatfunktionen benutzen.

**Beispiel:**

- Geben Sie den aktuellen Lagerbestand von allen Geschäfte, deren maximaler Lagerbestand größer als 400 oder minimaler Lagerbestand kleiner als 100 ist.

```
SELECT store_id, MAX(current_stock), MIN(current_stock)
FROM store_stock
GROUP BY store_id
HAVING MAX(current_stock)> 400 OR MIN(current_stock)<100
```



###ORDER BY
 Durch **ORDER BY** werden die gefilterte Ergebnisse nach einem oder mehreren Attributen sortiert.
 Die Reihenfolge werden durch ASC oder DESC bestimmt.

**Beispiel**
``` 
 SELECT* FROM products
 ORDER BY product_name ASC, product_price DESC
```
---
##Joins: INNER -, RIGHT -, LEFT -, FULL [OUTER] JOIN

**Beispiel für INNER JOIN:**

- bsp
>

![img.png](img.png)

---
##Änderungsoperationen: Einfügen, Löschen und Verändern