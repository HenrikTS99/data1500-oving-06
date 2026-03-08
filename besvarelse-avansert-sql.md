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
    Rangerer ansatt etter lønn (1-12)

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
    Viser vare, kategori og prisen, så gjennomsnittprisen til kategorien.

### Del 2: Lag SQL-spørringer

1.  **Rangering av varer per kategori:**
    ```sql
    SELECT v.Betegnelse, k.Navn as Kategori, v.Pris,
    RANK() OVER (PARTITION BY k.Navn ORDER BY v.pris DESC) AS Prisranger
    FROM Vare v
    JOIN Kategori k on v.katNr = K.KatNr;

    ```

2.  **Løpende sum av ordrebeløp:**
    ```sql
    SELECT o.ordrenr,
        o.ordredato,
        SUM(ol.prisprenhet * ol.antall) AS totalbeløp,
        SUM(SUM(ol.prisprenhet * ol.antall)) OVER (ORDER BY o.ordredato) AS løpende_summen
    FROM ordre o
    JOIN ordrelinje ol ON o.ordrenr = ol.ordrenr
    GROUP BY o.ordrenr, o.ordredato
    ORDER BY o.ordredato;
    ```

3.  **Prosentandel av kategoriprisen:**
    ```sql
    SELECT
    V.Betegnelse,
    K.Navn AS Kategori,
    V.Pris,
    ROUND(V.Pris / SUM(V.Pris) OVER (PARTITION BY K.Navn) * 100, 2) AS prosentAndel,
    SUM(V.Pris) OVER (PARTITION BY K.Navn) AS SumprisForKategori
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
