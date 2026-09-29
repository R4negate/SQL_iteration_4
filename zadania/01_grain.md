# Zadania 01 - Grain danych

W tych zadaniach skup sie na tym, co oznacza jeden wiersz wyniku.

## Zadanie 1

Pokaz wszystkich klientow.

Napisz pod zapytaniem komentarz:

```text
Grain wyniku: jeden wiersz = ...
```

## Zadanie 2

Pokaz wszystkie zamowienia.

Wynik powinien zawierac:

- `order_id`,
- `customer_id`,
- `order_date`,
- `status`,
- `total_amount`.

Napisz, jaki jest grain wyniku.

## Zadanie 3

Polacz `course.customers` z `course.orders`.

Wynik powinien zawierac:

- `customer_id`,
- `customer_name`,
- `order_id`,
- `total_amount`.

Uzyj `LEFT JOIN`.

Napisz, jaki jest grain wyniku po joinie.

## Zadanie 4

Policz liczbe wierszy po polaczeniu `customers` z `orders`.

Porownaj wynik z liczba klientow w tabeli `customers`.

Napisz jednym zdaniem, dlaczego te liczby moga byc rozne.

## Zadanie 5

Policz liczbe wszystkich klientow po joinie `customers` z `orders`.

Wynik powinien zawierac jedna kolumne:

- `customers_count`

Uzyj takiego podejscia, zeby klient z wieloma zamowieniami byl policzony tylko raz.

## Zadanie 6

Polacz `course.orders` z `course.order_items`.

Wynik powinien zawierac:

- `order_id`,
- `total_amount`,
- `order_item_id`,
- `quantity`,
- `unit_price`.

Napisz, jaki jest grain wyniku.

## Zadanie 7

Policz laczna wartosc pozycji zamowien.

Wynik powinien zawierac:

- `items_total_revenue`

Wartosc licz jako:

```text
quantity * unit_price
```

## Zadanie 8

Przygotuj wynik na grainie zamowienia.

Wynik powinien zawierac:

- `order_id`,
- `items_total_amount`

`items_total_amount` to suma:

```text
quantity * unit_price
```

## Zadanie 9

Przygotuj wynik na grainie klienta.

Wynik powinien zawierac:

- `customer_id`,
- `orders_count`,
- `total_revenue`

Uzyj tabeli `course.orders`.

## Zadanie 10

Napisz query z CTE:

1. pierwszy CTE ma policzyc wartosc pozycji na grainie zamowienia,
2. drugi CTE ma policzyc laczna sprzedaz na grainie klienta,
3. finalny SELECT ma pokazac:
   - `customer_id`,
   - `orders_count`,
   - `total_revenue`.

## Zadanie 11

Przygotuj raport per kraj klienta.

Wynik powinien zawierac:

- `country`,
- `customers_count`,
- `orders_count`,
- `total_revenue`

Zasady:

- `customers_count` ma liczyc unikalnych klientow,
- `orders_count` ma liczyc zamowienia,
- pokaz tez kraje klientow bez zamowien.

## Zadanie 12

Napisz jedno zdanie:

```text
Dlaczego SUM(o.total_amount) po joinie orders z order_items moze dac zly wynik?
```
