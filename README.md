# SQL iteration 4 - grain, CTE i window functions

Ta iteracja rozwija SQL w kierunku pracy data engineera.

W poprzednich iteracjach byly:

- DQL: czytanie danych, joiny, agregacje, subquery,
- DML: zmiana danych, transakcje, upsert,
- DDL: struktura bazy, constraints, indeksy, widoki.

W tej iteracji skupiamy sie na pisaniu bardziej zlozonych transformacji SQL.

## Na czym pracujemy

Pracujemy na tych samych tabelach:

- `course.customers`,
- `course.orders`,
- `course.order_items`,
- `course.products`.

Przed rozpoczeciem upewnij sie, ze baza ma dane z poprzednich iteracji.

## Kolejnosc nauki

1. `teoria/01_grain.md`
2. `zadania/01_grain.md`
3. `teoria/02_cte.md`
4. `zadania/02_cte.md`
5. `teoria/03_window_functions.md`
6. `zadania/03_window_functions.md`
7. `zadania/04_zadania_koncowe_podstaw_sql.md`

## Najwazniejsza mysl

Grain pomaga pilnowac, co oznacza jeden wiersz wyniku i czy join albo agregacja nie zawyzaja liczb.

CTE pomaga pisac query krok po kroku.

Window functions pozwalaja liczyc wartosci "w kontekscie grupy", ale bez zwijania wielu wierszy do jednego wiersza jak przy klasycznym `GROUP BY`.
