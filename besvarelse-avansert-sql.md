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
    Lager en CTE med postnummer og antall kunder på dette postnummeret, så blir det brukt til å beregne antall kunder per postnummer, der hvor det er over 5 kunder, og viser antall kunder.
    Denne spørringen er lik som:  

    ```sql
    SELECT 
        p.poststed,
        COUNT(*) AS AntallKunder
    FROM Kunde k JOIN poststed p ON k.postnr = p.postnr
    GROUP BY p.postnr
    HAVING COUNT(*) > 5 ORDER BY count(*) DESC;

    ```

### Del 2: Lag SQL-spørringer

1.  **Ansatte med over gjennomsnittslønn:**
    ```sql
    WITH Gjennomsnittslønn AS (
        SELECT AVG(Årslønn) as lønn
        FROM ansatt
    )
    SELECT ansnr, fornavn, Årslønn
    FROM ansatt a
    JOIN Gjennomsnittslønn gl ON a.Årslønn > gl.lønn;
    ```

    Uten CTE
    ```sql
    WITH Gjennomsnittslønn AS (
        SELECT AVG(Årslønn) as lønn
        FROM ansatt
    )
    SELECT ansnr, fornavn, Årslønn
    FROM ansatt a
    WHERE Årslønn > (SELECT lønn FROM Gjennomsnittslønn);
    ```

2.  **Kategorier med flest varer:**
    ```sql
    WITH VarerPerKategori AS (
        SELECT katnr, COUNT(vnr) AS antallVarer
        FROM vare
        GROUP BY katNr
    )
    SELECT k.navn, vpk.antallVarer
    FROM kategori k 
    JOIN VarerPerKategori vpk ON k.katnr = vpk.katnr
    WHERE vpk.antallVarer = (
        SELECT MAX(antallVarer) FROM VarerPerKategori
    );

    ```
    Uten CTE
    ```sql
    SELECT k.navn, COUNT(v.vnr) AS antallVarer
    FROM kategori k
    JOIN vare v ON k.katnr = v.katnr
    GROUP BY k.navn
    HAVING COUNT(v.vnr) = (
        SELECT MAX(antall)
        FROM (
            SELECT COUNT(vnr) AS antall
            FROM vare
            GROUP BY katnr
        ) AS tmp
    )
    ORDER BY antallVarer DESC;
    ```

3.  **Rekursiv CTE - Hierarki av ansatte:**
    ```sql
    -- Skriv din SQL-spørring her (inkluder gjerne ALTER TABLE og testdata)
    ALTER TABLE ansatt
        ADD COLUMN LederAnsNr INT;

    UPDATE ansatt SET LederAnsNr = 1 WHERE ansnr IN (2, 3);
    UPDATE ansatt SET LederAnsNr = 2 WHERE ansnr IN (6, 7);
    UPDATE ansatt SET LederAnsNr = 6 WHERE ansnr IN (11, 12);
    UPDATE ansatt SET LederAnsNr = NULL WHERE ansnr = 1;  -- top manager

    WITH RECURSIVE hieraki AS (
        SELECT AnsNr, Fornavn, LederAnsNr, 1 AS Nivå
        FROM Ansatt
        WHERE LederAnsNr = 1

        UNION ALL 

        -- Rekursivt steg: finn alle underordnede
        SELECT A.ANsNr, A.Fornavn, A.LederAnsNr, H.Nivå + 1
        FROM Ansatt A
        JOIN hieraki H ON A.LederAnsNr = H.AnsNr
    )
    SELECT * FROM hieraki;
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
