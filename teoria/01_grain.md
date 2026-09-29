# 01 - Grain danych

## Czym jest grain?

`Grain` to poziom szczegolowosci danych w wyniku zapytania.

Inaczej: grain odpowiada na pytanie:

```text
Co oznacza jeden wiersz w tym wyniku?
```

Przyklady:

- jeden wiersz = jeden klient,
- jeden wiersz = jedno zamowienie,
- jeden wiersz = jedna pozycja zamowienia,
- jeden wiersz = jeden miesiac i jeden kraj,
- jeden wiersz = jeden produkt i jedna kategoria.

To jest jedna z najwazniejszych rzeczy w SQL, bo wiele bledow w raportach wynika nie ze skladni, tylko z pomieszania grainu.

## Grain w tabelach kursowych

W naszym schemacie:

```text
course.customers
```

Grain:

```text
jeden wiersz = jeden klient
```

```text
course.orders
```

Grain:

```text
jeden wiersz = jedno zamowienie
```

```text
course.order_items
```

Grain:

```text
jeden wiersz = jedna pozycja zamowienia
```

```text
course.products
```

Grain:

```text
jeden wiersz = jeden produkt
```

## Dlaczego grain jest wazny?

Jezeli laczysz tabele o roznej szczegolowosci, liczba wierszy moze sie zmienic.

Przyklad:

```sql
SELECT
    o.order_id,
    o.total_amount,
    oi.order_item_id,
    oi.quantity,
    oi.unit_price
FROM course.orders o
JOIN course.order_items oi
    ON o.order_id = oi.order_id;
```

Tutaj zaczynamy od tabeli `orders`, gdzie jeden wiersz oznacza jedno zamowienie.

Po dolaczeniu `order_items` wynik ma juz inny grain:

```text
jeden wiersz = jedna pozycja zamowienia
```

Dlatego proste `SUM(o.total_amount)` po takim joinie moze zawyzyc wynik, bo ta sama kwota zamowienia powtorzy sie tyle razy, ile zamowienie ma pozycji.

## Przyklad bledu grainu

To zapytanie moze dac zly wynik:

```sql
SELECT
    SUM(o.total_amount) AS total_revenue
FROM course.orders o
JOIN course.order_items oi
    ON o.order_id = oi.order_id;
```

Problem:

- `orders.total_amount` jest na grainie zamowienia,
- `order_items` jest na grainie pozycji zamowienia,
- po joinie jedno zamowienie moze wystapic kilka razy.

Bezpieczniej liczyc wartosc z pozycji:

```sql
SELECT
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM course.order_items oi;
```

Albo najpierw przygotowac dane na odpowiednim grainie.

## Zmiana grainu przez GROUP BY

`GROUP BY` zmienia grain wyniku.

Przyklad:

```sql
SELECT
    customer_id,
    COUNT(order_id) AS orders_count,
    SUM(total_amount) AS total_revenue
FROM course.orders
GROUP BY customer_id;
```

Grain wyniku:

```text
jeden wiersz = jeden klient
```

Tabela `orders` ma grain zamowienia, ale po `GROUP BY customer_id` wynik ma grain klienta.

## Grain a CTE

CTE pomaga pisac query etapami. Najlepiej, gdy kazdy CTE ma jasny grain.

Przyklad:

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        COUNT(order_id) AS orders_count,
        SUM(total_amount) AS total_revenue
    FROM course.orders
    GROUP BY customer_id
)
SELECT
    *
FROM customer_totals;
```

Grain CTE `customer_totals`:

```text
jeden wiersz = jeden klient
```

## Grain jako kontrola poprawnosci query

Przed napisaniem zapytania warto zapytac:

1. Jaki ma byc jeden wiersz wyniku?
2. Z jakiej tabeli startuje query?
3. Czy join zmieni liczbe wierszy?
4. Czy po joinie agreguje dane na dobrym poziomie?
5. Czy `COUNT`, `SUM`, `AVG` licza to, co faktycznie chcemy?

## Najczestsze bledy

Blad 1:

```text
Liczenie SUM(total_amount) po joinie orders + order_items.
```

Mozliwy efekt:

```text
zawyzone przychody
```

Blad 2:

```text
COUNT(customer_id) po joinie customers + orders.
```

Mozliwy efekt:

```text
liczba klientow liczona wiele razy
```

Poprawka:

```sql
COUNT(DISTINCT c.customer_id)
```

Blad 3:

```text
Brak GROUP BY na poziomie, na ktorym ma byc raport.
```

Mozliwy efekt:

```text
raport pokazuje dane na zlym poziomie szczegolowosci
```

## Najwazniejsza zasada

Zanim napiszesz query, ustal grain wyniku.

```text
Jeden wiersz w moim wyniku oznacza...
```

To zdanie bardzo czesto chroni przed bledami w raportach.
