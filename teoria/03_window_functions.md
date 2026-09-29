# 03 - Window functions

## Czym sa window functions?

Window functions to funkcje, ktore licza wynik dla kazdego wiersza, ale moga patrzec na inne wiersze z tej samej grupy.

Najprosciej:

```text
GROUP BY zwija wiele wierszy do jednego.
Window function zostawia wiersze i dokleja dodatkowe wyliczenie.
```

Przyklad biznesowy:

```text
Pokaz kazde zamowienie i obok laczna sprzedaz klienta.
```

Klasyczne `GROUP BY` pokazaloby jeden wiersz per klient.

Window function moze pokazac kazde zamowienie i dodatkowo sume klienta.

## Podstawowa skladnia

```sql
funkcja(...) OVER (
    PARTITION BY kolumna_grupujaca
    ORDER BY kolumna_sortujaca
    ROWS BETWEEN ... AND ...
)
```

Najwazniejsze elementy:

- `OVER` uruchamia funkcje okna,
- `PARTITION BY` dzieli dane na grupy,
- `ORDER BY` ustawia kolejnosc wierszy w grupie,
- `ROWS BETWEEN` okresla, ktore wiersze z okna funkcja ma widziec.

## GROUP BY vs window function

`GROUP BY`:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS total_revenue
FROM course.orders
GROUP BY customer_id;
```

Wynik:

```text
jeden wiersz per customer_id
```

Window function:

```sql
SELECT
    order_id,
    customer_id,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY customer_id
    ) AS customer_total_revenue
FROM course.orders;
```

Wynik:

```text
jeden wiersz per zamowienie, plus suma klienta
```

## ROW_NUMBER

`ROW_NUMBER()` nadaje kolejne numery wierszom.

Przyklad: ponumeruj zamowienia kazdego klienta od najnowszego.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date DESC, order_id DESC
    ) AS order_number
FROM course.orders;
```

Logika:

```text
PARTITION BY customer_id - osobna numeracja dla kazdego klienta.
ORDER BY order_date DESC - najnowsze zamowienie dostaje numer 1.
```

## Latest record per group

Bardzo czesty przypadek w data engineeringu:

```text
Pokaz najnowsze zamowienie kazdego klienta.
```

```sql
WITH numbered_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        total_amount,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM course.orders
)
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount
FROM numbered_orders
WHERE rn = 1;
```

To jest jeden z najwazniejszych wzorcow:

```text
ROW_NUMBER w CTE, potem WHERE rn = 1.
```

## RANK i DENSE_RANK

`RANK()` i `DENSE_RANK()` sa podobne do `ROW_NUMBER()`, ale obsluguja remisy.

```sql
SELECT
    order_id,
    customer_id,
    total_amount,
    RANK() OVER (
        ORDER BY total_amount DESC
    ) AS revenue_rank,
    DENSE_RANK() OVER (
        ORDER BY total_amount DESC
    ) AS revenue_dense_rank
FROM course.orders;
```

Roznica:

- `ROW_NUMBER` zawsze nadaje unikalny numer,
- `RANK` daje ten sam ranking przy remisie, ale zostawia dziury,
- `DENSE_RANK` daje ten sam ranking przy remisie i nie zostawia dziur.

Przyklad:

```text
kwoty: 300, 300, 200

ROW_NUMBER: 1, 2, 3
RANK:       1, 1, 3
DENSE_RANK: 1, 1, 2
```

## LAG

`LAG()` pozwala pobrac wartosc z poprzedniego wiersza.

Przyklad: pokaz poprzednia kwote zamowienia klienta.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    LAG(total_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS previous_order_amount
FROM course.orders;
```

`LAG` jest przydatny do:

- porownania z poprzednim rekordem,
- liczenia roznic,
- sledzenia zmian statusu,
- analizy historii.

## LEAD

`LEAD()` dziala podobnie, ale patrzy na nastepny wiersz.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    LEAD(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS next_order_date
FROM course.orders;
```

## Running total

Running total to suma narastajaca.

Przyklad: narastajaca sprzedaz klienta po kolejnych zamowieniach.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_customer_revenue
FROM course.orders;
```

Ta czesc:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

znaczy:

```text
wez wszystkie poprzednie wiersze w oknie i aktualny wiersz
```

Dzieki temu powstaje suma narastajaca.

## Window frames

Window frame mowi, ktore wiersze funkcja okna ma brac pod uwage.

Najczestsze przyklady:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

od poczatku okna do aktualnego wiersza.

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

dwa poprzednie wiersze plus aktualny wiersz.

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

cale okno od pierwszego do ostatniego wiersza.

Warto pamietac:

```text
ROWS liczy fizyczne wiersze.
RANGE grupuje rekordy o takich samych wartosciach ORDER BY.
```

Na tym etapie najczesciej uzywaj `ROWS`, bo jest latwiejsze do zrozumienia.

## Moving average

Moving average to srednia kroczaca.

Przyklad: srednia z aktualnego i maksymalnie dwoch poprzednich zamowien klienta.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    ROUND(
        AVG(total_amount) OVER (
            PARTITION BY customer_id
            ORDER BY order_date, order_id
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ),
        2
    ) AS three_order_moving_avg
FROM course.orders
ORDER BY customer_id, order_date, order_id;
```

To jest przydatne do wygladzania trendow.

## Procent udzialu w calosci

Window functions pozwalaja porownac wiersz do sumy globalnej.

```sql
SELECT
    order_id,
    total_amount,
    SUM(total_amount) OVER () AS all_orders_revenue,
    ROUND(
        total_amount / SUM(total_amount) OVER () * 100,
        2
    ) AS revenue_percent
FROM course.orders;
```

`OVER ()` oznacza:

```text
patrz na wszystkie wiersze jako jedno okno
```

## NTILE i PERCENT_RANK

`NTILE` dzieli wynik na grupy podobnej wielkosci.

Przyklad: podziel klientow na kwartyle wedlug lacznej sprzedazy.

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(total_amount) AS revenue
    FROM course.orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    revenue,
    NTILE(4) OVER (
        ORDER BY revenue DESC
    ) AS revenue_quartile,
    PERCENT_RANK() OVER (
        ORDER BY revenue
    ) AS percent_rank
FROM customer_revenue
ORDER BY revenue DESC;
```

Znaczenie:

- `NTILE(4)` dzieli rekordy na 4 grupy,
- `PERCENT_RANK()` pokazuje wzgledna pozycje od `0` do `1`.

## FIRST_VALUE i LAST_VALUE

`FIRST_VALUE` pobiera pierwsza wartosc z okna.

`LAST_VALUE` pobiera ostatnia wartosc z okna, ale trzeba uwazac na domyslna ramke.

Przyklad:

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    FIRST_VALUE(total_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS first_order_amount,
    LAST_VALUE(total_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_order_amount
FROM course.orders
ORDER BY customer_id, order_date, order_id;
```

Pulapka:

```text
LAST_VALUE bez pelnej ramki czesto zwraca wartosc z aktualnego wiersza, a nie prawdziwa ostatnia wartosc klienta.
```

Dlatego przy `LAST_VALUE` zwykle dopisujemy:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

## Named windows

Jesli kilka funkcji okna ma ten sam `PARTITION BY` i `ORDER BY`, mozna nazwac okno.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    LAG(total_amount) OVER customer_timeline AS previous_order_amount,
    LEAD(total_amount) OVER customer_timeline AS next_order_amount,
    ROW_NUMBER() OVER customer_timeline AS customer_order_number
FROM course.orders
WINDOW customer_timeline AS (
    PARTITION BY customer_id
    ORDER BY order_date, order_id
)
ORDER BY customer_id, order_date, order_id;
```

To zmniejsza powtarzanie kodu.

## Deduplikacja przez ROW_NUMBER

To jeden z najczestszych wzorcow w praktycznej pracy data engineera.

Kroki:

```text
1. Wybierz klucz duplikatu, np. LOWER(email).
2. Wybierz zasade, ktory rekord zostaje, np. najnowszy signup_date.
3. Nadaj ROW_NUMBER().
4. Zostaw tylko rn = 1.
```

Przyklad:

```sql
WITH ranked_customers AS (
    SELECT
        c.*,
        ROW_NUMBER() OVER (
            PARTITION BY LOWER(email)
            ORDER BY signup_date DESC, customer_id DESC
        ) AS duplicate_rank
    FROM course.customers c
    WHERE email IS NOT NULL
)
SELECT
    customer_id,
    customer_name,
    email,
    country,
    signup_date,
    acquisition_channel
FROM ranked_customers
WHERE duplicate_rank = 1;
```

Logika:

```text
Dla kazdego emaila zostaw najnowszy rekord.
Jesli jest remis, zostaw rekord z wiekszym customer_id.
```

W PostgreSQL istnieje tez krotszy zapis `DISTINCT ON`, ale `ROW_NUMBER` jest bardziej przenoszalny miedzy rozne silniki SQL.

## Window function po agregacji

Mozna najpierw zrobic agregacje, a potem funkcje okna na wyniku agregacji.

Przyklad: ranking krajow po sprzedazy.

```sql
WITH sales_by_country AS (
    SELECT
        c.country,
        SUM(o.total_amount) AS total_revenue
    FROM course.customers c
    JOIN course.orders o
        ON c.customer_id = o.customer_id
    GROUP BY c.country
)
SELECT
    country,
    total_revenue,
    RANK() OVER (
        ORDER BY total_revenue DESC
    ) AS country_rank
FROM sales_by_country;
```

## Dialect notes

Rozne silniki SQL maja skroty do tych samych problemow.

### FILTER w PostgreSQL

PostgreSQL pozwala pisac warunkowe agregacje przez `FILTER`.

```sql
SELECT
    c.country,
    COUNT(*) AS all_orders,
    COUNT(*) FILTER (WHERE o.status = 'paid') AS paid_orders,
    SUM(o.total_amount) FILTER (WHERE o.status = 'paid') AS paid_revenue
FROM course.orders o
JOIN course.customers c
    ON o.customer_id = c.customer_id
GROUP BY c.country;
```

To jest czytelniejsza wersja:

```sql
COUNT(CASE WHEN o.status = 'paid' THEN 1 END)
```

W innych silnikach mozesz spotkac np. `COUNTIF` albo `count_if`.

### DISTINCT ON w PostgreSQL

PostgreSQL ma `DISTINCT ON`, ktore moze zastapic niektore przypadki `ROW_NUMBER`.

Przyklad: najnowsze zamowienie klienta.

```sql
SELECT DISTINCT ON (customer_id)
    customer_id,
    order_id,
    order_date,
    total_amount
FROM course.orders
ORDER BY customer_id, order_date DESC, order_id DESC;
```

To jest krotsze, ale mniej przenoszalne niz `ROW_NUMBER()`.

### QUALIFY w hurtowniach

PostgreSQL nie ma `QUALIFY`.

Dlatego w PostgreSQL filtrujemy wynik window function przez CTE:

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC
        ) AS rn
    FROM course.orders
)
SELECT *
FROM ranked
WHERE rn = 1;
```

W niektorych hurtowniach, np. BigQuery, Snowflake albo Databricks SQL, mozna spotkac:

```sql
SELECT *
FROM orders
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date DESC
) = 1;
```

Drabinka logiczna:

```text
WHERE filtruje wiersze przed grupowaniem.
HAVING filtruje grupy po agregacji.
QUALIFY filtruje wyniki funkcji okna.
```

## Najczestsze bledy

### Mylenie GROUP BY z window function

Jesli chcesz zmniejszyc liczbe wierszy, uzyj `GROUP BY`.

Jesli chcesz zostawic wiersze i dodac wyliczenie, uzyj window function.

### Brak ORDER BY przy ROW_NUMBER

To jest technicznie mozliwe, ale zwykle bez sensu biznesowego.

```sql
ROW_NUMBER() OVER (PARTITION BY customer_id)
```

Bez `ORDER BY` nie mowisz bazie, ktory rekord ma byc pierwszy.

### RANK zamiast ROW_NUMBER przy deduplikacji

Do deduplikacji zwykle chcesz dokladnie jeden rekord.

Uzyj:

```sql
ROW_NUMBER()
```

a nie:

```sql
RANK()
```

`RANK` moze zostawic wiecej niz jeden rekord przy remisie.

### Filtrowanie po aliasie window function w tym samym SELECT

To zwykle nie zadziala:

```sql
SELECT
    order_id,
    ROW_NUMBER() OVER (...) AS rn
FROM course.orders
WHERE rn = 1;
```

Najpierw policz `rn` w CTE, potem filtruj:

```sql
WITH numbered AS (
    SELECT
        order_id,
        ROW_NUMBER() OVER (...) AS rn
    FROM course.orders
)
SELECT *
FROM numbered
WHERE rn = 1;
```

## Najwazniejsze do zapamietania

- Window functions dzialaja przez `OVER`.
- `PARTITION BY` tworzy grupy, ale nie zwija wierszy.
- `ORDER BY` w oknie ustala kolejnosc liczenia.
- `ROWS BETWEEN` okresla ramke okna.
- `ROW_NUMBER` jest podstawowym narzedziem do deduplikacji i wyboru najnowszego rekordu.
- `RANK` i `DENSE_RANK` obsluguja remisy.
- `LAG` patrzy na poprzedni wiersz.
- `LEAD` patrzy na nastepny wiersz.
- `FIRST_VALUE` i `LAST_VALUE` pobieraja wartosci z poczatku i konca okna.
- Przy `LAST_VALUE` trzeba uwazac na ramke.
- Agregacje okienne pozwalaja pokazac sume/srednia bez utraty szczegolow.
