# 02 - CTE, czyli WITH

## Czym jest CTE?

CTE to nazwany fragment zapytania, ktory istnieje tylko na czas wykonania jednego query.

CTE zapisujemy przez `WITH`.

Najprosciej:

```text
CTE = tymczasowy wynik z nazwa, uzywany w jednym zapytaniu
```

Albo:

```text
WITH pozwala napisac trudne query krok po kroku.
```

## Podstawowa skladnia

```sql
WITH nazwa_cte AS (
    SELECT ...
)
SELECT ...
FROM nazwa_cte;
```

Przyklad:

```sql
WITH paid_orders AS (
    SELECT
        order_id,
        customer_id,
        total_amount
    FROM course.orders
    WHERE status = 'paid'
)
SELECT *
FROM paid_orders;
```

Co tu sie dzieje?

```text
1. paid_orders to nazwany wynik SELECT-a.
2. Glowny SELECT czyta z paid_orders tak, jakby to byla tabela.
3. paid_orders nie zostaje zapisane w bazie.
4. paid_orders istnieje tylko w ramach tego jednego query.
```

## CTE nie jest tabela ani view

CTE:

```text
istnieje tylko w jednym zapytaniu
```

Tabela:

```text
istnieje w bazie jako obiekt z danymi
```

View:

```text
istnieje w bazie jako zapisane query
```

CTE nie tworzy trwalego obiektu w bazie.

## Po co uzywac CTE?

CTE pomaga, gdy:

- query robi sie dlugie,
- masz kilka etapow liczenia,
- chcesz najpierw policzyc agregacje, a potem uzyc jej dalej,
- chcesz uniknac powtarzania tego samego subquery,
- chcesz, zeby zapytanie bylo czytelniejsze,
- chcesz debugowac query krok po kroku.

CTE nie jest potrzebne do kazdego prostego query.

## Najpierw ustal grain

Zanim napiszesz CTE albo window function, ustal:

```text
Co oznacza jeden wiersz wyniku?
```

To nazywa sie grain.

Przyklad:

- `course.order_items` ma grain: jedna pozycja zamowienia,
- `course.orders` ma grain: jedno zamowienie,
- raport miesieczny ma grain: jeden miesiac,
- raport klienta ma grain: jeden klient.

To jest wazne, bo bardzo latwo policzyc cos na zlym poziomie szczegolow.

Przyklad: jezeli chcesz policzyc wartosc zamowienia na podstawie pozycji, najpierw musisz zejsc do grainu:

```text
jeden wiersz per order_id
```

```sql
WITH order_values AS (
    SELECT
        o.customer_id,
        o.order_id,
        o.order_date,
        SUM(oi.quantity * oi.unit_price) AS order_value
    FROM course.orders o
    JOIN course.order_items oi
        ON o.order_id = oi.order_id
    GROUP BY
        o.customer_id,
        o.order_id,
        o.order_date
)
SELECT *
FROM order_values
ORDER BY customer_id, order_date;
```

Ten CTE ma jeden cel:

```text
zamienic pozycje zamowien na jeden wiersz per zamowienie
```

To jest bardzo czesty wzorzec w transformacjach danych.

## Jedno CTE powinno miec jedno zadanie

Dobra praktyka:

```text
jedno CTE = jeden krok logiki
```

Przyklad:

- `order_values` - liczy wartosc zamowienia,
- `monthly_revenue` - agreguje zamowienia do miesiecy,
- `revenue_with_previous` - dodaje wartosc z poprzedniego miesiaca.

Slabsze podejscie:

```text
jedno ogromne CTE, ktore jednoczesnie czysci dane, laczy tabele, liczy agregacje, segmentuje klientow i robi finalny raport
```

CTE ma pomagac czytac query jak pipeline.

## Przyklad bez CTE

Zadanie:

```text
Pokaz klientow, ktorzy wydali wiecej niz srednia laczna wartosc zamowien per klient.
```

Bez CTE:

```sql
SELECT
    c.customer_id,
    c.customer_name,
    SUM(o.total_amount) AS total_revenue
FROM course.customers c
JOIN course.orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name
HAVING SUM(o.total_amount) > (
    SELECT
        AVG(customer_total)
    FROM (
        SELECT
            customer_id,
            SUM(total_amount) AS customer_total
        FROM course.orders
        GROUP BY customer_id
    ) customer_totals
);
```

To dziala, ale jest trudniejsze do czytania.

## To samo z CTE

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(total_amount) AS total_revenue
    FROM course.orders
    GROUP BY customer_id
),
average_customer_total AS (
    SELECT
        AVG(total_revenue) AS average_total_revenue
    FROM customer_totals
)
SELECT
    c.customer_id,
    c.customer_name,
    ct.total_revenue
FROM course.customers c
JOIN customer_totals ct
    ON c.customer_id = ct.customer_id
JOIN average_customer_total act
    ON ct.total_revenue > act.average_total_revenue
ORDER BY ct.total_revenue DESC;
```

Logika:

```text
1. customer_totals liczy laczna sprzedaz per klient.
2. average_customer_total liczy srednia z tych lacznych wartosci.
3. Glowny SELECT pokazuje klientow powyzej sredniej.
```

## CTE vs subquery

Subquery:

```sql
SELECT *
FROM (
    SELECT ...
) alias;
```

CTE:

```sql
WITH alias AS (
    SELECT ...
)
SELECT *
FROM alias;
```

Rownica praktyczna:

```text
Subquery siedzi w srodku query.
CTE wyciaga krok na gore i nadaje mu nazwe.
```

CTE bardzo czesto robi to samo, co subquery, ale jest wygodniejsze przy kilku krokach.

## Kiedy subquery, a kiedy CTE?

Uzyj subquery, gdy:

- logika jest krotka,
- subquery jest proste,
- uzywasz go tylko raz,
- query nadal jest czytelne.

Uzyj CTE, gdy:

- query ma kilka etapow,
- subquery jest dlugie,
- chcesz nazwac etap,
- chcesz uzyc wyniku kilka razy,
- chcesz latwiej debugowac zapytanie.

## CTE z agregacja

```sql
WITH customer_sales AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(o.total_amount) AS total_revenue,
        COUNT(o.order_id) AS orders_count
    FROM course.customers c
    JOIN course.orders o
        ON c.customer_id = o.customer_id
    GROUP BY
        c.customer_id,
        c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_revenue,
    orders_count
FROM customer_sales
WHERE total_revenue > 200
ORDER BY total_revenue DESC;
```

W CTE robimy agregacje.

W glownym `SELECT` filtrujemy gotowy wynik agregacji.

## CTE z HAVING

```sql
WITH high_value_customers AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(o.total_amount) AS total_revenue
    FROM course.customers c
    JOIN course.orders o
        ON c.customer_id = o.customer_id
    GROUP BY
        c.customer_id,
        c.customer_name
    HAVING SUM(o.total_amount) > 200
)
SELECT *
FROM high_value_customers
ORDER BY total_revenue DESC;
```

`HAVING` nadal sluzy do filtrowania po agregacji.

## Kilka CTE w jednym query

Mozesz miec kilka CTE oddzielonych przecinkami.

```sql
WITH orders_summary AS (
    SELECT
        customer_id,
        COUNT(order_id) AS orders_count,
        SUM(total_amount) AS total_revenue
    FROM course.orders
    GROUP BY customer_id
),
paid_orders_summary AS (
    SELECT
        customer_id,
        COUNT(order_id) AS paid_orders_count
    FROM course.orders
    WHERE status = 'paid'
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(os.orders_count, 0) AS orders_count,
    COALESCE(pos.paid_orders_count, 0) AS paid_orders_count,
    COALESCE(os.total_revenue, 0) AS total_revenue
FROM course.customers c
LEFT JOIN orders_summary os
    ON c.customer_id = os.customer_id
LEFT JOIN paid_orders_summary pos
    ON c.customer_id = pos.customer_id
ORDER BY total_revenue DESC;
```

## CTE moze korzystac z poprzedniego CTE

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(total_amount) AS total_revenue
    FROM course.orders
    GROUP BY customer_id
),
average_total AS (
    SELECT
        AVG(total_revenue) AS average_revenue
    FROM customer_totals
)
SELECT
    ct.customer_id,
    ct.total_revenue,
    at.average_revenue
FROM customer_totals ct
CROSS JOIN average_total at
WHERE ct.total_revenue > at.average_revenue;
```

Drugie CTE `average_total` korzysta z pierwszego CTE `customer_totals`.

## Dlaczego CROSS JOIN?

`average_total` zwraca jeden wiersz z jedna srednia.

`CROSS JOIN` dokleja te jedna wartosc do kazdego klienta.

To pozwala porownac kazdego klienta do jednej globalnej sredniej.

## CTE jako pipeline

CTE sa bardzo wygodne do pisania transformacji krok po kroku.

Przyklad:

```sql
WITH order_values AS (
    SELECT
        o.customer_id,
        o.order_id,
        o.order_date,
        SUM(oi.quantity * oi.unit_price) AS order_value
    FROM course.orders o
    JOIN course.order_items oi
        ON o.order_id = oi.order_id
    GROUP BY
        o.customer_id,
        o.order_id,
        o.order_date
),
monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', order_date) AS revenue_month,
        SUM(order_value) AS revenue
    FROM order_values
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
    revenue_month,
    revenue
FROM monthly_revenue
ORDER BY revenue_month;
```

Czytamy to tak:

```text
1. order_values - jeden wiersz per zamowienie z policzona wartoscia.
2. monthly_revenue - jeden wiersz per miesiac.
3. final SELECT - pokaz wynik.
```

## CTE + window function

CTE bardzo czesto przygotowuje grain, a window function dodaje analityczne wyliczenie.

Przyklad: miesieczna sprzedaz i porownanie z poprzednim miesiacem.

```sql
WITH monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', order_date) AS revenue_month,
        SUM(total_amount) AS revenue
    FROM course.orders
    GROUP BY DATE_TRUNC('month', order_date)
),
revenue_with_previous AS (
    SELECT
        revenue_month,
        revenue,
        LAG(revenue) OVER (
            ORDER BY revenue_month
        ) AS previous_revenue
    FROM monthly_revenue
)
SELECT
    revenue_month,
    revenue,
    previous_revenue,
    revenue - previous_revenue AS absolute_growth
FROM revenue_with_previous
ORDER BY revenue_month;
```

Logika:

```text
1. Najpierw robimy jeden wiersz per miesiac.
2. Potem window function porownuje miesiac z poprzednim miesiacem.
```

To jest jeden z najwazniejszych wzorcow analitycznego SQL.

## CTE z CASE

```sql
WITH customer_activity AS (
    SELECT
        c.customer_id,
        c.customer_name,
        COUNT(o.order_id) AS orders_count,
        COALESCE(SUM(o.total_amount), 0) AS total_revenue
    FROM course.customers c
    LEFT JOIN course.orders o
        ON c.customer_id = o.customer_id
    GROUP BY
        c.customer_id,
        c.customer_name
)
SELECT
    customer_id,
    customer_name,
    orders_count,
    total_revenue,
    CASE
        WHEN orders_count = 0 THEN 'no_orders'
        WHEN total_revenue >= 300 THEN 'high_value'
        ELSE 'standard'
    END AS customer_segment
FROM customer_activity
ORDER BY total_revenue DESC;
```

CTE przygotowuje dane, a glowny `SELECT` dodaje logike biznesowa.

## CTE z anti joinem

```sql
WITH customers_without_orders AS (
    SELECT
        c.customer_id,
        c.customer_name
    FROM course.customers c
    LEFT JOIN course.orders o
        ON c.customer_id = o.customer_id
    WHERE o.order_id IS NULL
)
SELECT
    customer_id,
    customer_name,
    'customer_without_order' AS issue_type
FROM customers_without_orders
ORDER BY customer_id;
```

## CTE z UNION ALL

```sql
WITH customers_without_orders AS (
    SELECT
        c.customer_id AS object_id,
        c.customer_name AS object_name
    FROM course.customers c
    LEFT JOIN course.orders o
        ON c.customer_id = o.customer_id
    WHERE o.order_id IS NULL
),
products_without_sales AS (
    SELECT
        p.product_id AS object_id,
        p.product_name AS object_name
    FROM course.products p
    LEFT JOIN course.order_items oi
        ON p.product_id = oi.product_id
    WHERE oi.order_item_id IS NULL
)
SELECT
    'customer_without_order' AS issue_type,
    object_id,
    object_name
FROM customers_without_orders

UNION ALL

SELECT
    'product_without_sale' AS issue_type,
    object_id,
    object_name
FROM products_without_sales

ORDER BY issue_type, object_id;
```

## CTE do debugowania trudnego query

Jesli query jest trudne, nie pisz wszystkiego naraz.

Najpierw napisz pierwszy krok:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS total_revenue
FROM course.orders
GROUP BY customer_id;
```

Jesli dziala, opakuj go w CTE:

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(total_amount) AS total_revenue
    FROM course.orders
    GROUP BY customer_id
)
SELECT *
FROM customer_totals;
```

Potem dodawaj kolejne kroki.

## Nazewnictwo CTE

Dobre nazwy:

```text
customer_totals
order_item_summary
products_without_sales
monthly_sales
high_value_customers
```

Slabe nazwy:

```text
x
a
tmp
data
query1
```

CTE ma poprawiac czytelnosc, dlatego nazwa powinna mowic, co jest w srodku.

## Najczestsze bledy

### Brak przecinka miedzy CTE

Zle:

```sql
WITH first_cte AS (
    SELECT ...
)
second_cte AS (
    SELECT ...
)
SELECT ...
```

Dobrze:

```sql
WITH first_cte AS (
    SELECT ...
),
second_cte AS (
    SELECT ...
)
SELECT ...
```

### Proba uzycia CTE w kolejnym query

CTE istnieje tylko dla jednego zapytania.

To nie zadziala:

```sql
WITH customer_totals AS (
    SELECT customer_id, SUM(total_amount) AS total_revenue
    FROM course.orders
    GROUP BY customer_id
)
SELECT *
FROM customer_totals;

SELECT *
FROM customer_totals;
```

Drugie zapytanie nie zna `customer_totals`.

### CTE bez sensu

Nie kazde query potrzebuje CTE.

To nie upraszcza zapytania:

```sql
WITH all_customers AS (
    SELECT *
    FROM course.customers
)
SELECT *
FROM all_customers;
```

## Czy CTE przyspiesza query?

CTE jest przede wszystkim narzedziem czytelnosci i organizacji query.

Nie nalezy zakladac, ze CTE zawsze przyspiesza zapytanie.

Wydajnosc sprawdza sie przez:

```sql
EXPLAIN ANALYZE
```

## WITH RECURSIVE

`WITH RECURSIVE` to bardziej zaawansowany rodzaj CTE, ktory moze odwolac sie sam do siebie.

Uzywa sie go np. do:

- struktur drzewiastych,
- hierarchii,
- kategorii nadrzednych i podrzednych,
- generowania kolejnych krokow.

Prosty przyklad:

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numbers
    WHERE n < 5
)
SELECT *
FROM numbers;
```

Wynik:

```text
1
2
3
4
5
```

Na tym etapie najwazniejsze jest zwykle `WITH`. `WITH RECURSIVE` to temat dodatkowy.

## Recursive CTE jako calendar spine

Jednym z praktycznych zastosowan `WITH RECURSIVE` jest zbudowanie kalendarza.

To przydaje sie, gdy chcesz pokazac wszystkie dni w raporcie, nawet takie, w ktorych nie bylo zamowien.

```sql
WITH RECURSIVE calendar_dates AS (
    SELECT DATE '2026-01-01' AS calendar_date

    UNION ALL

    SELECT calendar_date + 1
    FROM calendar_dates
    WHERE calendar_date < DATE '2026-01-07'
)
SELECT *
FROM calendar_dates;
```

Wynik:

```text
2026-01-01
2026-01-02
2026-01-03
2026-01-04
2026-01-05
2026-01-06
2026-01-07
```

W PostgreSQL czesto da sie to zrobic prosciej przez `generate_series`, ale recursive CTE pokazuje ogolny, przenoszalny mechanizm.

Najwazniejsze elementy recursive CTE:

- anchor, czyli pierwszy wiersz,
- recursive step, czyli sposob generowania kolejnych wierszy,
- stop condition, czyli warunek zatrzymania.

Bez warunku zatrzymania mozna przypadkowo stworzyc nieskonczona rekurencje.

## CTE vs VIEW

CTE:

```text
istnieje tylko w jednym query
```

VIEW:

```text
jest zapisanym obiektem w bazie
```

Jesli logika jest potrzebna tylko w jednym query, CTE wystarczy.

Jesli chcesz uzywac tej logiki wielokrotnie jako obiektu w bazie, rozwaz `VIEW`.

## Najwazniejsze do zapamietania

- CTE zapisujemy przez `WITH`.
- CTE to nazwany fragment query.
- CTE istnieje tylko w ramach jednego zapytania.
- CTE nie tworzy tabeli w bazie.
- CTE pomaga rozbijac trudne query na kroki.
- Przed CTE warto ustalic grain, czyli co oznacza jeden wiersz.
- Dobre CTE ma jedno zadanie.
- Kilka CTE oddzielamy przecinkami.
- Kolejne CTE moze korzystac z poprzedniego.
- CTE dobrze laczy sie z window functions.
- CTE czesto zastepuje trudne subquery w `FROM`.
- CTE poprawia czytelnosc, ale nie gwarantuje lepszej wydajnosci.
- Do stalej logiki uzywanej wielokrotnie lepszy moze byc `VIEW`.
