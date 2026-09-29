# Zadania 03 - Window functions

W tych zadaniach uzywaj funkcji okna.

## Zadanie 1

Pokaz wszystkie zamowienia i dodaj numer zamowienia klienta.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `customer_order_number`.

Numeruj zamowienia osobno dla kazdego klienta, od najstarszego do najnowszego.

## Zadanie 2

Pokaz najnowsze zamowienie kazdego klienta.

Wynik powinien zawierac:

- `customer_id`,
- `order_id`,
- `order_date`,
- `total_amount`.

Uzyj `ROW_NUMBER()` i CTE.

## Zadanie 3

Pokaz zamowienia z rankingiem wedlug wartosci `total_amount`.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `total_amount`,
- `revenue_rank`,
- `revenue_dense_rank`.

Uzyj `RANK()` i `DENSE_RANK()`.

## Zadanie 4

Pokaz zamowienia klientow i poprzednia kwote zamowienia tego samego klienta.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `previous_order_amount`.

Uzyj `LAG()`.

## Zadanie 5

Pokaz zamowienia klientow i roznice wzgledem poprzedniego zamowienia klienta.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `previous_order_amount`,
- `amount_difference`.

## Zadanie 6

Pokaz zamowienia klientow i date kolejnego zamowienia tego samego klienta.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `next_order_date`.

Uzyj `LEAD()`.

## Zadanie 7

Pokaz narastajaca sprzedaz klienta po kolejnych zamowieniach.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `running_customer_revenue`.

## Zadanie 8

Pokaz kazde zamowienie oraz laczna wartosc zamowien danego klienta.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `total_amount`,
- `customer_total_revenue`.

Nie uzywaj `GROUP BY`.

## Zadanie 9

Pokaz procentowy udzial kazdego zamowienia w calkowitej sprzedazy.

Wynik powinien zawierac:

- `order_id`,
- `total_amount`,
- `all_orders_revenue`,
- `revenue_percent`.

Zaokraglij procent do 2 miejsc po przecinku.

## Zadanie 10

Policz sprzedaz per kraj klienta, a nastepnie dodaj ranking krajow wedlug `total_revenue`.

Wynik powinien zawierac:

- `country`,
- `total_revenue`,
- `country_rank`.

Uzyj CTE i funkcji okna.

## Zadanie 11

Znajdz pierwsze zamowienie kazdego klienta.

Wynik powinien zawierac:

- `customer_id`,
- `order_id`,
- `order_date`,
- `total_amount`.

Uzyj `ROW_NUMBER()` i CTE.

## Zadanie 12

Pokaz produkty z rankingiem sprzedanych sztuk w ramach kategorii.

Wynik powinien zawierac:

- `category`,
- `product_id`,
- `product_name`,
- `units_sold`,
- `rank_in_category`.

Najpierw policz `units_sold` per produkt, a potem dodaj ranking.

## Zadanie 13

Pokaz zamowienia klientow i srednia kroczaca z aktualnego oraz maksymalnie dwoch poprzednich zamowien klienta.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `three_order_moving_avg`.

Uzyj:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

## Zadanie 14

Podziel klientow na 4 grupy wedlug lacznej wartosci zamowien.

Wynik powinien zawierac:

- `customer_id`,
- `total_revenue`,
- `revenue_quartile`.

Uzyj `NTILE(4)`.

## Zadanie 15

Pokaz pierwsza i ostatnia kwote zamowienia kazdego klienta przy kazdym zamowieniu.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `first_order_amount`,
- `last_order_amount`.

Uzyj `FIRST_VALUE` i `LAST_VALUE`.

Pamietaj o jawnej ramce dla `LAST_VALUE`.

## Zadanie 16

Pokaz zamowienia klientow i policz w jednym zapytaniu:

- poprzednia kwote zamowienia,
- nastepna kwote zamowienia,
- numer zamowienia klienta.

Uzyj named window.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `total_amount`,
- `previous_order_amount`,
- `next_order_amount`,
- `customer_order_number`.

## Zadanie 17

Przygotuj deduplikacje klientow po emailu.

Zasady:

- duplikat rozpoznaj po `LOWER(email)`,
- zostaw najnowszy rekord wedlug `signup_date`,
- przy remisie zostaw wiekszy `customer_id`,
- pomin rekordy z `email IS NULL`.

Uzyj `ROW_NUMBER()` i CTE.

Wynik powinien zawierac:

- `customer_id`,
- `customer_name`,
- `email`,
- `country`,
- `signup_date`.
