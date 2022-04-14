[Übung5DML.md](https://github.com/byzakyz/dbis-hiwi/files/8485049/Ubung5DML.md)


``` python
 from IPython.display import display, Code
# SQL Befehl
hello_world_query = '''
    SELECT 'Hello World'
'''
# Ausführen des Befehls
hello_world_test = %sql $hello_world_query
# Ausgabe des Befehls mit Syntax-Highlighting
display(Code(hello_world_query, language='sql'))
# Ausgabe der Ergebnisse
display(hello_world_test)            
``` 

### SQL Datentypen
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


## Create Table

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
##Primary Key

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

##Alter Table
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
##Foreign Key

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
##Drop Table
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
##Create Index
Falls kein Index angelegt ist, erfolgt die Suche nach Informationen innerhalb einer Relation sequentiell.
Indexe ermöglichen bei der Suche das Nutzen von Datenstrukturen wie B+ Bäume um den Zugriff zu beschleunigen.

**Syntax:**
>CREATE [UNIQUE] INDEX 
<_Indexname_>
ON <_Relationenname_> (<_Attributname_> [<_Ordnung_>]{, <_Attributname_> [<_Ordnung_>]})

wobei  <_Ordnung_> ::= 
ASC | DESC


**Ein "SELECT" Beispiel aus dem Datenbank:**
- Finde "order" welche an Timestamp '2020-8-16 10:31:23+01' erstellt wurde.

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
##Create View
Erstellt eine Abfrage auf den Graphen und gibt es eine Name. 
Später können weitere Abfragen auf dieser View verweisen. 
Für den erstellten View wird keine Relation angelegt, 
sondern wird die gespeicherte Abfrage für jeden Verweis neu durchgeführt.

**Snytax:**
- CREATE VIEW <_Sichtname_> [(<_Attributname_>{, <_Attributname_>})] AS <_Subquery_>

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
Nutze dabei das erstellte View.
```python .noeval
SELECT u.user_full_name AS "Name"
FROM paid_orders o
INNER JOIN users u on u.user_id = o.order_store_id
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
### Data Manipulation Languange (DML)

DML beschäftigt sich mit allen Typen von Anfragen in SQL-Sprache. 
>Die Grundform ist wie folgt definiert: `SELECT`<_Liste von Attributsnamen_>
>                                        `FROM` <_ein oder mehrere Relationennamen_>
>                                        [`WHERE` <_Bedingung_>

In Bezug auf relationalen Algebra, wird mit `SELECT` - Klausel die Anfrage projektiert. Die Relationen, die für die Anfrage benötigt wird, werden durch `FROM`-Klausel dargestellt. Hier entspricht das `FROM` -Klausel das Kreuzprodukt von Relationen. Und das `WHERE`- Klausel beschreibt die Bedingungen für die Selektion.

####Beispiel
Gegeben sei folgende relationale Datenbankschema:

DesignerIn (<ins>Kuerzel</ins>, Vorname, Nachname) <p>
Schuch(<ins>PID</ins>, Typ, Modell, Preis, DKuerzel)<p>
>Geben Sie die Nachnamen bei denen die Vorname "Lukas" sind.
``` 
    SELECT Nachname 
    FROM designerIn
    WHERE Vorname = 'Lukas'
```
###WHERE -Klausel
Nach dem `WHERE` können die folgenden Vergleichsoperatoren vorkommen:

- logische Verknüpfungen : AND ,OR, NOT
- arithmetische Operatoren : +, -, *, /, >, <, =
- Mengen Operatoren : IN, NOT IN, ANY, ALL, EXISTS or BETWEEN, LIKE, IS NULL

#### Beispiel

>Geben Sie die Modelle von Sandalen, die ihren Preisen größer als 500 sind.
```
   SELECT Modell
   FROM Schuch
   WHERE Preis > 500
   AND Typ = 'Sandalen'
```


### SELECT FROM- Klausel

>`SELECT DISTINCT` - Klausel nimmt nur die unterschiedlichen Variablen. Die Duplikaten werden nicht angenommen.

> `SELECT* FROM`- Klausel projektiert alle Spalten in der Relation.

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




