# Øvingsoppgave 2.2: Avansert SQL

**Mål:** Lære å bruke avanserte SQL-teknikker — Window Functions, Common Table Expressions (CTEs), og avanserte subqueries — for å utføre komplekse analyser på en relasjonsdatabase.

**Database:** `hobbyhuset` (samme som i øving 2.1)

---

## Kom i gang

### Forutsetninger

Denne øvingen bygger på `hobbyhuset`-databasen fra øving 2.1. Sørg for at databasen kjører og er populert med data fra `init-scripts/hobbyhuset.sql` og utvidelsene fra `test-scripts/hobbyhuset_utvidet_o5-6.sql`.

### Start databasen

```bash
docker-compose up -d
```

### Kjør utvidelsene (hvis du ikke allerede har gjort det)

```bash
docker-compose exec postgres psql -U admin -d hobbyhuset -f test-scripts/hobbyhuset_utvidet_o5-6.sql
```

### Koble til databasen

```bash
docker-compose exec postgres psql -U admin -d hobbyhuset
```


---

## Besvarelse

- Forklaringer og SQL skrives i `besvarelse-avansert-sql.md`


---

## Oppgave 1: Window Functions

*Window functions* (vindufunksjoner) utfører beregninger på et sett med rader som er relatert til den nåværende raden, uten å kollapse radene til én (slik `GROUP BY` gjør). De brukes til rangeringer, løpende totaler, og sammenligninger innenfor grupper.

**Generell syntaks:**
```sql
funksjon() OVER (
    [PARTITION BY kolonne]  -- Del opp i grupper
    [ORDER BY kolonne]      -- Sorter innenfor gruppen
)
```

**Vanlige vindufunksjoner:**

| Funksjon | Beskrivelse |
|----------|-------------|
| `RANK()` | Gir rang med hopp ved like verdier (1, 2, 2, 4) |
| `DENSE_RANK()` | Gir rang uten hopp ved like verdier (1, 2, 2, 3) |
| `ROW_NUMBER()` | Gir unikt løpenummer per rad |
| `SUM() OVER (...)` | Løpende sum |
| `AVG() OVER (...)` | Løpende eller partisjonert gjennomsnitt |

### Del 1: Forklar SQL-spørringene (skriv i `besvarelse-avansert-sql-sql.md`)

1.  **Spørring:**
    ```sql
    SELECT
        Fornavn,
        Etternavn,
        Årslønn,
        RANK() OVER (ORDER BY Årslønn DESC) AS Lønnsrangering
    FROM Ansatt;
    ```

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

### Del 2: Lag SQL-spørringer (skriv i `besvarelse-avansert-sql.sql`)

1.  **Rangering av varer per kategori:** Rangér alle varer etter pris (dyrest først) *innenfor* hver kategori. Resultatet skal vise varenavn, kategori, pris og rang.
2.  **Løpende sum:** Vis alle ordrer med ordredato og totalbeløp (`SUM(Pris * Antall)` fra `Ordrelinje`), og legg til en kolonne som viser den *løpende summen* av ordrebeløp sortert etter dato.
3.  **Prosentandel av kategoriprisen:** For hver vare, beregn hvor stor prosentandel varens pris utgjør av den totale prisen for alle varer i samme kategori. Avrund til to desimaler.

---

## Oppgave 2: Common Table Expressions (CTEs)

En CTE (Common Table Expression) er en midlertidig, navngitt resultattabell som defineres med `WITH`-nøkkelordet. CTEs gjør komplekse spørringer mer lesbare ved å dele dem opp i logiske steg.

**Generell syntaks:**
```sql
WITH navn_paa_cte AS (
    SELECT ...   -- Definer den midlertidige tabellen
)
SELECT *         -- Bruk den midlertidige tabellen
FROM navn_paa_cte
WHERE ...;
```

Du kan også definere **rekursive CTEs** for å traversere hierarkiske strukturer (f.eks. organisasjonskart):
```sql
WITH RECURSIVE hierarki AS (
    -- Basistilfelle: startpunkt
    SELECT AnsNr, Fornavn, LederAnsNr, 0 AS Nivå
    FROM Ansatt
    WHERE LederAnsNr IS NULL

    UNION ALL

    -- Rekursivt steg: finn alle underordnede
    SELECT A.AnsNr, A.Fornavn, A.LederAnsNr, H.Nivå + 1
    FROM Ansatt A
    JOIN hierarki H ON A.LederAnsNr = H.AnsNr
)
SELECT * FROM hierarki;
```

### Del 1: Forklar SQL-spørringen (skriv i `besvarelse-avansert-sql-sql.md`)

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

### Del 2: Lag SQL-spørringer (skriv i `besvarelse-avansert-sql.sql`)

1.  **Ansatte med over gjennomsnittslønn:** Bruk en CTE til å først beregne gjennomsnittslønnen for alle ansatte, og deretter liste opp alle ansatte som tjener mer enn dette gjennomsnittet. Vis navn, stilling og lønn.
2.  **Kategorier med flest varer:** Bruk en CTE til å telle antall varer i hver kategori, og deretter finne *kun* kategorien(e) med flest varer. Vis kategorinavn og antall.
3.  **Rekursiv CTE — Hierarki av ansatte:** Legg til en `LederAnsNr`-kolonne i `Ansatt`-tabellen og sett inn noen testverdier (f.eks. at ansatt 1 er leder for ansatt 2 og 3, og ansatt 2 er leder for ansatt 4). Skriv deretter en rekursiv CTE som finner alle ansatte som rapporterer til ansatt 1, direkte eller indirekte, og vis hvilket nivå i hierarkiet de befinner seg på.

---

## Oppgave 3: Avanserte Subqueries

En subquery (underspørring) er en `SELECT`-setning inni en annen SQL-setning. Subqueries kan brukes i `WHERE`, `FROM`, og `SELECT`-klausuler.

**Typer subqueries:**

| Type | Beskrivelse | Eksempel |
|------|-------------|---------|
| **Skalær** | Returnerer én enkelt verdi | `WHERE Pris > (SELECT AVG(Pris) FROM Vare)` |
| **Korrelert** | Refererer til den ytre spørringen | `WHERE Pris > (SELECT AVG(Pris) FROM Vare WHERE KatNr = V.KatNr)` |
| **I `FROM`** | Brukes som en midlertidig tabell | `FROM (SELECT ...) AS t` |
| **Med `EXISTS`** | Sjekker om subquery returnerer rader | `WHERE EXISTS (SELECT 1 FROM ...)` |

### Del 1: Forklar SQL-spørringene (skriv i `besvarelse-avansert-sql-sql.md`)

1.  **Spørring (Korrelert subquery):**
    ```sql
    SELECT V.Betegnelse, V.Pris
    FROM Vare V
    WHERE V.Pris > (
        SELECT AVG(Pris)
        FROM Vare
        WHERE KatNr = V.KatNr
    );
    ```

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

### Del 2: Lag SQL-spørringer (skriv i `besvarelse-avansert-sql.sql`)

1.  **Kunder som har bestilt en spesifikk vare:** Finn fornavn og etternavn på alle kunder som har bestilt varen med `VNr` = `'10820'`. Bruk en subquery med `IN`.
2.  **`EXISTS` — Kategorier med dyre varer:** Bruk `EXISTS` for å finne alle kategorier som har minst én vare med en pris over 1000 kr. Vis kategorinavn.
3.  **Varer dyrere enn gjennomsnittet:** Finn alle varer som er dyrere enn gjennomsnittsprisen for *alle* varer i databasen. Vis varenavn og pris, sortert fra dyrest til billigst.

---

## Ekstraoppgave (valgfri)

**Kombiner CTE og Window Function:** Bruk en CTE til å beregne totalt salgsbeløp per vare (sum av `Pris * Antall` fra `Ordrelinje`), og bruk deretter en window function til å rangere varene etter totalt salgsbeløp innenfor hver kategori. Vis topp 3 varer per kategori.

```sql
-- Hint: Bruk RANK() OVER (PARTITION BY ... ORDER BY ...) og filtrer på rang <= 3
```
