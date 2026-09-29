# Zadania 02 - CTE, czyli WITH

W tych zadaniach uzywaj CTE, czyli konstrukcji `WITH`.

CTE pomaga rozbic dluzsze query na nazwane kroki. Nie tworzy stalej tabeli w bazie danych.

## Zadanie 1

Utworz CTE o nazwie `paid_orders`, ktore zawiera tylko zamowienia ze statusem `paid`.

Nastepnie pobierz z niego:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`.

Wynik posortuj po `order_date`.

## Zadanie 2

Utworz CTE `customer_sales`, ktore policzy laczna sprzedaz per klient.

Wynik koncowy powinien zawierac:

- `customer_id`,
- `total_revenue`.

Pokaz tylko klientow, dla ktorych `total_revenue` jest wieksze niz `200`.

## Zadanie 3

Utworz CTE `customer_sales`, ktore policzy:

- `customer_id`,
- `orders_count`,
- `total_revenue`.

Nastepnie polacz wynik z tabela `course.customers`, zeby pokazac:

- `customer_id`,
- `customer_name`,
- `orders_count`,
- `total_revenue`.

Wynik posortuj po `total_revenue` malejaco.

## Zadanie 4

Utworz dwa CTE:

- `paid_orders` - zamowienia ze statusem `paid`,
- `cancelled_orders` - zamowienia ze statusem `cancelled`.

Nastepnie jednym wynikiem pokaz:

- `status_group`,
- `orders_count`,
- `total_revenue`.

Uzyj `UNION ALL`.

## Zadanie 5

Utworz CTE `customer_totals`, ktore policzy laczna wartosc zamowien per klient.

Nastepnie pokaz klientow, ktorzy wydali wiecej niz srednia laczna wartosc zamowien per klient.

Wynik powinien zawierac:

- `customer_id`,
- `customer_name`,
- `total_revenue`.

Wynik posortuj po `total_revenue` malejaco.

## Zadanie 6

Utworz CTE `order_items_totals`, ktore policzy wartosc pozycji zamowienia per `order_id`.

Wartosc pozycji licz jako:

```text
quantity * unit_price
```

Nastepnie przygotuj raport porownujacy kwote z `course.orders.total_amount` do sumy pozycji.

Wynik powinien zawierac:

- `order_id`,
- `status`,
- `order_total_amount`,
- `items_total_amount`,
- `difference_amount`.

Pokaz wszystkie zamowienia, rowniez te bez pozycji.

## Zadanie 7

Utworz CTE `customers_without_orders`, ktore znajdzie klientow bez zamowien.

Uzyj anti joina:

```sql
LEFT JOIN ... WHERE ... IS NULL
```

Wynik powinien zawierac:

- `customer_id`,
- `customer_name`,
- `country`.

## Zadanie 8

Przygotuj raport kontrolny skladajacy sie z 3 czesci polaczonych przez `UNION ALL`.

Wynik powinien zawierac:

- `issue_type`,
- `object_id`,
- `object_name`,
- `metric_value`.

Raport ma pokazac:

1. klientow bez zamowien,
2. produkty bez sprzedazy,
3. klientow, ktorzy maja laczna wartosc zamowien wieksza niz `300`.

W pierwszej i drugiej czesci uzyj anti joina.

## Zadanie 9

Utworz CTE `product_sales`, ktore policzy sprzedaz per produkt.

Wynik koncowy powinien zawierac:

- `product_id`,
- `product_name`,
- `category`,
- `units_sold`,
- `total_revenue`.

Pokaz tylko produkty, ktore sprzedaly sie w liczbie wiekszej niz `1`.

Uzyj `HAVING`.

## Zadanie 10

Utworz CTE `monthly_sales`, ktore policzy miesieczna sprzedaz per kraj klienta.

Wynik powinien zawierac:

- `sales_month`,
- `country`,
- `orders_count`,
- `total_revenue`.

Zasady:

- miesiac wylicz z `orders.order_date`,
- uzyj `DATE_TRUNC`,
- polacz `orders` z `customers`,
- pokaz tylko grupy, gdzie `total_revenue` jest wieksze niz `100`.

## Zadanie 11

Utworz CTE `customer_segments`, ktore nada klientom segment:

- `with_orders`, jezeli klient ma przynajmniej jedno zamowienie,
- `without_orders`, jezeli klient nie ma zadnego zamowienia.

Wynik powinien zawierac:

- `customer_id`,
- `customer_name`,
- `customer_segment`.

## Zadanie 12

Napisz krotka odpowiedz:

1. Czym CTE rozni sie od zwyklej subquery?
2. Czym CTE rozni sie od `VIEW`?
