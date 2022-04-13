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