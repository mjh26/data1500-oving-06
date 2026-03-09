# Besvarelse: Avansert SQL

## Oppgave 1: Window Functions

### Del 1: Forklar SQL-spørringene

1.  **Spørring:**
    ```sql
    SELECT
        Fornavn,
        Etternavn,
        Årslønn,
        RANK() OVER (ORDER BY Årslønn DESC) AS Lønnsrangering
    FROM Ansatt;
    ```
    **Forklaring:**
    *   *... Skriv din forklaring her ...*
Velger alle ansatte, sortert på årslønn (høyest først) og hver ansatt får en rangering basert på lønnen.

2.  **Spørring:**
    ```sql
    SELECT
        V.Betegnelse,
        K.Navn AS Kategori,
        V.Pris,
        AVG(V.Pris) OVER (PARTITION BY K.Navn) AS GjennomsnittsprisForKategori
    FROM Vare V
    JOIN Kategori K ON V.KatNr = K.KatNr;
    ```
    **Forklaring:**
    *   *... Skriv din forklaring her ...*
Velger alle varer med kategori og pris, beregner gjennomsnittsprisen for hver kategori og gjennomsnittet på hver red uten å gruppere bort noe.

### Del 2: Lag SQL-spørringer

1.  **Rangering av varer per kategori:**
    ```sql
    -- Skriv din SQL-spørring her
    SELECT 
    V.Betegnelse,
    K.Navn AS Kategori,
    V.Pris,
    RANK() OVER (PARTITION BY K.Navn ORDER BY V.Pris DESC) AS PrisRang
    FROM Vare V
    JOIN Kategori K ON V.KatNr = K.KatNr;
    ```

2.  **Løpende sum av ordrebeløp:**
    ```sql
    -- Skriv din SQL-spørring her
    WITH OrdreTotal AS
    (SELECT 
    O.OrdreNr,
    O.OrdreDato,
    SUM(OL.prisprenhet * OL.Antall) AS Total
    FROM Ordre O
    JOIN Ordrelinje OL ON O.OrdreNr = OL.OrdreNr
    GROUP BY O.OrdreNr, O.OrdreDato)
    SELECT 
    OrdreNr,
    OrdreDato,
    Total, 
    SUM(Total) OVER (ORDER BY OrdreDato) AS LøpendeSum
    FROM OrdreTotal
    ORDER BY OrdreDato;
    ```

3.  **Prosentandel av kategoriprisen:**
    ```sql
    -- Skriv din SQL-spørring her
    SELECT
    V.Betegnelse,
    K.Navn AS Kategori,
    V.Pris,
    ROUND(V.Pris * 100.0 / SUM(V.Pris) OVER (PARTITION BY K.Navn),
    2)
    AS ProsentAvKategori
    FROM Vare V
    JOIN Kategori K ON V.KatNr = K.KatNr;
    ```

---

## Oppgave 2: Common Table Expressions (CTEs)

### Del 1: Forklar SQL-spørringen

1.  **Spørring:**
    ```sql
    WITH KunderPerPoststed AS (
        SELECT PostNr, COUNT(*) AS AntallKunder
        FROM Kunde
        GROUP BY PostNr
    )
    SELECT P.Poststed, KPP.AntallKunder
    FROM Poststed P
    JOIN KunderPerPoststed KPP ON P.PostNr = KPP.PostNr
    WHERE KPP.AntallKunder > 5
    ORDER BY KPP.AntallKunder DESC;
    ```
    **Forklaring:**
    *   *... Skriv din forklaring her ...*

### Del 2: Lag SQL-spørringer

1.  **Ansatte med over gjennomsnittslønn:**
    ```sql
    -- Skriv din SQL-spørring her
    ```

2.  **Kategorier med flest varer:**
    ```sql
    -- Skriv din SQL-spørring her
    ```

3.  **Rekursiv CTE - Hierarki av ansatte:**
    ```sql
    -- Skriv din SQL-spørring her (inkluder gjerne ALTER TABLE og testdata)
    ```

---

## Oppgave 3: Avanserte Subqueries

### Del 1: Forklar SQL-spørringene

1.  **Spørring (Correlated Subquery):**
    ```sql
    SELECT V.Betegnelse, V.Pris
    FROM Vare V
    WHERE V.Pris > (
        SELECT AVG(Pris)
        FROM Vare
        WHERE KatNr = V.KatNr
    );
    ```
    **Forklaring:**
    *   *... Skriv din forklaring her ...*

2.  **Spørring (Subquery i `FROM`):**
    ```sql
    SELECT Kategori, Gjennomsnittspris
    FROM (
        SELECT K.Navn AS Kategori, AVG(V.Pris) AS Gjennomsnittspris
        FROM Vare V
        JOIN Kategori K ON V.KatNr = K.KatNr
        GROUP BY K.Navn
    ) AS KategoriPriser
    WHERE Gjennomsnittspris > 100;
    ```
    **Forklaring:**
    *   *... Skriv din forklaring her ...*

### Del 2: Lag SQL-spørringer

1.  **Kunder som har bestilt en spesifikk vare:**
    ```sql
    -- Skriv din SQL-spørring her
    ```

2.  **`EXISTS` - Kategorier med dyre varer:**
    ```sql
    -- Skriv din SQL-spørring her
    ```

3.  **Varer dyrere enn gjennomsnittet:**
    ```sql
    -- Skriv din SQL-spørring her
    ```
