
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

CHAR, VARCHAR, DECIMAL, INTEGER /INT



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
Indexe ermöglichen bei der Suche das Nutzen von Datenstrukturen wie B+ Bäume um den Zugriff zu beschleunigen

mit Beispiel Database:
SELECT * FROM Relation WHERE Spalte = 'xxxx'

mit vs ohne index mit EXPLAIN

---
##Create View
Erstellt eine Abfrage auf den Graphen und gibt es eine Name. 
Später können weitere Abfragen auf dieser View verweisen. 
Für den erstellten View wird keine Relation angelegt, 
sondern wird die gespeicherte Abfrage für jeden Verweis neu durchgeführt.

**Snytax:**
- CREATE VIEW <_Sichtname_> [(<_Attributname_>{, <_Attributname_>})] AS <_Subquery_>

**Beispiel:**
- Erstelle einen View, der die Vorlesungen des Jahres 2020 zurückgibt.
```python .noeval
CREATE VIEW Vorlesungen_2020 AS
SELECT * FROM Vorlesung WHERE Jahr = 2020
```
-Nutze dieses View um




