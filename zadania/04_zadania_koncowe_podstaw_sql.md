# Zadania koncowe - podstawy SQL

## Zadanie 1

Przygotuj raport aktywnosci klientow.

Wynik powinien zawierac:

- `customer_id`
- `customer_name`
- `country`
- `acquisition_channel`
- `orders_count`
- `paid_orders_count`
- `cancelled_orders_count`
- `total_revenue`
- `average_order_value`
- `first_order_date`
- `last_order_date`
- `customer_status`

Zasady:

- pokaz wszystkich klientow,
- klienci bez zamowien maja miec `total_revenue = 0`,
- `customer_status`:
  - `no_orders`, jezeli klient nie ma zamowien,
  - `buyer`, jezeli klient ma przynajmniej jedno zamowienie,
- wynik posortuj po `total_revenue` malejaco.

## Zadanie 2

Przygotuj raport sprzedazy produktow.

Wynik powinien zawierac:

- `product_id`
- `product_name`
- `category`
- `base_price`
- `units_sold`
- `orders_count`
- `gross_revenue`
- `average_unit_price`
- `sale_status`

Zasady:

- pokaz wszystkie produkty,
- produkty bez sprzedazy maja miec `units_sold = 0` i `gross_revenue = 0`,
- `sale_status`:
  - `not_sold`, jezeli produkt nigdy sie nie sprzedal,
  - `sold`, jezeli produkt ma przynajmniej jedna pozycje zamowienia,
- wynik posortuj po `gross_revenue` malejaco.

## Zadanie 3

Przygotuj miesieczny raport sprzedazy per kraj klienta.

Wynik powinien zawierac:

- `sales_month`
- `country`
- `orders_count`
- `customers_count`
- `paid_orders_count`
- `cancelled_orders_count`
- `total_revenue`
- `average_order_value`

Zasady:

- miesiac wylicz z `orders.order_date`,
- `customers_count` ma liczyc unikalnych klientow,
- `paid_orders_count` ma liczyc tylko zamowienia ze statusem `paid`,
- `cancelled_orders_count` ma liczyc tylko zamowienia ze statusem `cancelled`,
- pokaz tylko miesiace i kraje, gdzie `total_revenue > 100`,
- wynik posortuj po `sales_month`, a potem po `total_revenue` malejaco.

## Zadanie 4

Pokaz klientow, ktorych laczna wartosc zamowien jest wieksza niz srednia laczna wartosc zamowien per klient.

Uzyj CTE.

Wynik powinien zawierac:

- `customer_id`
- `customer_name`
- `orders_count`
- `total_revenue`

Wynik posortuj po `total_revenue` malejaco.

## Zadanie 5

Przygotuj raport kontrolny problemow w danych.

Wynik powinien zawierac:

- `issue_type`
- `object_id`
- `object_name`
- `details`

Raport ma pokazac:

- klientow bez zamowien,
- produkty bez sprzedazy,
- zamowienia bez pozycji zamowienia,
- klientow bez emaila.

Zasady:
- wynik posortuj po `issue_type`, potem po `object_id`.

## Zadanie 6

Przygotuj raport zamowien z porownaniem kwoty z tabeli `orders` do sumy pozycji z `order_items`.

Wynik powinien zawierac:

- `order_id`
- `customer_name`
- `status`
- `order_total_amount`
- `items_total_amount`
- `difference_amount`
- `amount_check`

Zasady:

- `order_total_amount` to `orders.total_amount`,
- `items_total_amount` to suma `quantity * unit_price`,
- `difference_amount` to roznica miedzy `orders.total_amount` i suma pozycji,
- `amount_check`:
  - `match`, jezeli roznica wynosi `0`,
  - `different`, jezeli roznica jest inna niz `0`,
- pokaz wszystkie zamowienia, rowniez te bez pozycji,
- uzyj CTE albo subquery do policzenia sumy pozycji per zamowienie,
- wynik posortuj tak, aby najpierw byly rekordy z `amount_check = 'different'`.

## Zadanie 7

Przygotuj raport kanalow pozyskania klientow i ich sprzedazy.

Wynik powinien zawierac:

- `acquisition_channel`
- `customers_count`
- `customers_with_orders_count`
- `orders_count`
- `paid_orders_count`
- `total_revenue`
- `average_revenue_per_customer`

Zasady:

- uwzglednij wszystkich klientow,
- `customers_count` ma liczyc wszystkich klientow w danym kanale,
- `customers_with_orders_count` ma liczyc tylko unikalnych klientow z przynajmniej jednym zamowieniem,
- `orders_count` ma liczyc wszystkie zamowienia,
- `paid_orders_count` ma liczyc tylko zamowienia `paid`,
- `average_revenue_per_customer` to `total_revenue / customers_count`,
- wynik posortuj po `total_revenue` malejaco.

## Zadanie 8

Pokaz produkty sprzedane klientom z krajow `PL` i `DE`.

Wynik powinien zawierac:

- `product_id`
- `product_name`
- `category`
- `countries`
- `units_sold`
- `total_revenue`

Zasady:

- uzyj joinow przez tabele `order_items`, `orders` i `customers`,
- wez pod uwage tylko klientow z krajow `PL` i `DE`,
- `countries` ma pokazac kraje, w ktorych produkt zostal sprzedany,
- do `countries` uzyj `STRING_AGG`,
- `units_sold` to suma `quantity`,
- `total_revenue` to suma `quantity * unit_price`,
- pokaz tylko produkty, ktore sprzedaly sie lacznie w liczbie wiekszej niz `1`,
- wynik posortuj po `units_sold` malejaco, potem po `total_revenue` malejaco.

## Zadanie 9

Pokaz klientow, ktorzy kupili produkt z kategorii `course`, ale nigdy nie kupili produktu z kategorii `template`.

Wynik powinien zawierac:

- `customer_id`
- `customer_name`

Wynik posortuj po `customer_id`.

## Zadanie 10

Przygotuj raport zamowien od dnia `2026-05-01`.

Wynik powinien zawierac:

- `order_id`
- `order_date`
- `customer_name`
- `status`
- `items_count`
- `items_quantity`
- `items_total_amount`
- `order_total_amount`

Zasady:

- pokaz tylko zamowienia ze statusem `paid` albo `pending`,
- `items_count` ma liczyc liczbe pozycji zamowienia,
- `items_quantity` ma liczyc sume sztuk,
- `items_total_amount` ma liczyc sume `quantity * unit_price`,
- pokaz rowniez zamowienia bez pozycji,
- uzyj CTE do policzenia metryk pozycji per zamowienie,
- wynik posortuj po `order_date`, potem po `order_id`.

## Zadanie 11

Pokaz ostatnie zamowienie kazdego klienta.

Wynik powinien zawierac:

- `customer_id`
- `customer_name`
- `order_id`
- `order_date`
- `total_amount`

Zasady:

- jezeli klient nie ma zamowien, nie musi pojawiac sie w wyniku,
- ostatnie zamowienie oznacza najnowsza date `order_date`,
- przy remisie sortuj dodatkowo po `order_id` malejaco,
- wynik posortuj po `customer_id`.

## Zadanie 12

Pokaz wszystkie zamowienia i dodaj narastajaca sume sprzedazy klienta.

Wynik powinien zawierac:

- `customer_id`
- `order_id`
- `order_date`
- `total_amount`
- `customer_running_total`

Zasady:

- suma narastajaca ma byc liczona osobno dla kazdego klienta,
- sortowanie w oknie: `order_date`, potem `order_id`,
- wynik posortuj po `customer_id`, `order_date`, `order_id`.

## Zadanie 13

Pokaz zamowienia razem z poprzednia kwota zamowienia klienta.

Wynik powinien zawierac:

- `customer_id`
- `order_id`
- `order_date`
- `total_amount`
- `previous_order_amount`
- `difference_vs_previous_order`

Zasady:

- poprzednie zamowienie licz osobno dla kazdego klienta,
- roznica to `total_amount - previous_order_amount`,
- dla pierwszego zamowienia klienta roznica moze byc `NULL`.

## Zadanie 14

Przygotuj ranking produktow w ramach kazdej kategorii.

Wynik powinien zawierac:

- `product_id`
- `product_name`
- `category`
- `total_revenue`
- `rank_in_category`

Zasady:

- `total_revenue` licz jako suma `quantity * unit_price`,
- pokaz tylko produkty, ktore sie sprzedaly,
- ranking licz osobno dla kazdej kategorii,
- najwyzszy przychod ma miec ranking `1`

## Zadanie 15

Przygotuj deduplikacje klientow po emailu.

Wynik powinien zawierac:

- `customer_id`
- `customer_name`
- `email`
- `signup_date`

Zasady:

- traktuj email case-insensitive,
- zostaw najstarszy rekord per email,
- jezeli email jest `NULL`, pomin taki rekord,
- wynik posortuj po `email`.

## Zadanie 16

Utworz widok:

```text
course.v_customer_order_activity
```

Widok ma pokazywac aktywnosc klientow.

Kolumny widoku:

- `customer_id`
- `customer_name`
- `country`
- `orders_count`
- `total_revenue`
- `last_order_date`

Zasady:

- pokaz wszystkich klientow,
- klienci bez zamowien maja miec `total_revenue = 0`.

## Zadanie 17

Utworz materialized view:

```text
course.mv_monthly_sales_by_country
```

Widok zmaterializowany ma zawierac:

- `sales_month`
- `country`
- `orders_count`
- `total_revenue`

Zasady:

- miesiac wylicz z `orders.order_date`,
- uzyj `DATE_TRUNC`,
- po utworzeniu widoku napisz query, ktore go odswieza.

## Zadanie 18

Dodaj produkt przez upsert.

Dane produktu:

- `product_id = 250`
- `product_name = 'DML Practice Pack'`
- `category = 'course'`
- `base_price = 89.00`

Zasady:

- jezeli produkt nie istnieje, ma zostac dodany,
- jezeli produkt istnieje, ma zostac zaktualizowana cena i nazwa produktu.

## Zadanie 19

Przetestuj transakcje.

Napisz skrypt, ktory:

1. rozpoczyna transakcje,
2. dodaje testowego klienta,
3. dodaje testowe zamowienie dla tego klienta,
4. pokazuje dodane dane przez `SELECT`,
5. wycofuje transakcje przez `ROLLBACK`.

Zasady:

- uzyj takich ID, ktore nie konfliktuja z istniejacymi danymi,
- po `ROLLBACK` dane nie powinny zostac w tabelach.

## Zadanie 20

Pokaz produkty, ktorych cena jest wieksza niz srednia cena w ich kategorii.

Wynik powinien zawierac:

- `product_id`
- `product_name`
- `category`
- `base_price`
- `category_avg_price`

Wynik posortuj po `category`, a potem po `base_price` malejaco.
