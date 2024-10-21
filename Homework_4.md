### Создание таблицы

```sql
CREATE TABLE IF NOT EXISTS annastavi.accounts
(
    id     INTEGER NOT NULL,
    amount NUMERIC NOT NULL
);
```

### Вставка данных

```sql
INSERT INTO annastavi.accounts (id, amount)
VALUES (1, 100), (2, 200), (3, 300);
```

### Заходим в перкую консоль

```sql
BEGIN;
-- Блокируем первую строку
UPDATE annastavi.accounts a
SET amount = 330
WHERE a.id = 2;

--Блокируем вторую строку
UPDATE annastavi.accounts a
SET amount = 220
WHERE a.id = 1;
```

### Заходим во вторую консоль

```sql
BEGIN;
-- Блокируем первую строку
UPDATE annastavi.accounts a
SET amount = 220
WHERE a.id = 1;

--Блокируем вторую строку
UPDATE annastavi.accounts a
SET amount = 330
WHERE a.id = 2;
```

### Выдается ошибка 
/*
 [40P01] ERROR: deadlock detected Подробности:
 Process 38657 waits for ShareLock on transaction 1162; blocked by process 38512.
 Process 38512 waits for ShareLock on transaction 1164; blocked by process 38657.
 Подсказка: See server log for query details.
 Где: while updating tuple (0,1) in relation "accounts"
*/

### Смотрим какой файл используется
```sql
SELECT pg_current_logfile(); --log/postgresql-2024-10-19_000000.log
```

### Заходим в терминал и смотрим логи по файлу
--MacBook-Air-Ana:log root# cat -n /Library/PostgreSQL/16/data/log/postgresql-2024-10-19_000000.log 10 | tail -n 50

/*
    42	2024-10-21 13:03:40.700 MSK [38657] ERROR:  deadlock detected
    43	2024-10-21 13:03:40.700 MSK [38657] DETAIL:  Process 38657 waits for ShareLock on transaction 1162; blocked by process 38512.
    44		Process 38512 waits for ShareLock on transaction 1164; blocked by process 38657.
    45		Process 38657: UPDATE annastavi.accounts a
    46		SET amount = 220
    47		WHERE a.id = 1
    48		Process 38512: UPDATE annastavi.accounts a
    49		SET amount = 330
    50		WHERE a.id = 2
    51	2024-10-21 13:03:40.700 MSK [38657] HINT:  See server log for query details.
    52	2024-10-21 13:03:40.700 MSK [38657] CONTEXT:  while updating tuple (0,1) in relation "accounts"
 */
