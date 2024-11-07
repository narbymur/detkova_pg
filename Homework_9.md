### Создаем таблицу 

```sql
CREATE TABLE IF NOT EXISTS annastavi.json
(
    id               SERIAL PRIMARY KEY,
    json_data        JSONB,
    name             VARCHAR(255),
    date             TIMESTAMP,
    ch_employee_id   INTEGER,
    name_employee    VARCHAR(255),
    surname_employee VARCHAR(255),
    age              SMALLINT,
    title            VARCHAR(255),
    name_title       TEXT
);
```

### Генерируем туда данные - 1000000 строк

```sql
INSERT INTO annastavi.json (json_data, name, date, ch_employee_id, name_employee, surname_employee, age, title, name_title)
SELECT r.js,
       '858588458w85rtw4598w4587w495798w4578975439857439857938457974398574935798579745398735487458',
       NOW(),
       171718,
       '858588458w85rtw4598w4587w495798w4578975439857439857938457974398574935798579745398735487458',
       '858588458w85rtw4598w4587w495798w4578975439857439857938457974398574935798579745398735487458',
       23,
       1,
       repeat('Тост Живи. Тост Живи.Тост Живи.Тост Живи.Тост Живи.Тост Живи.Тост Живи.', 10000)
FROM generate_series(1, 50000)
         LEFT JOIN LATERAL (SELECT JSON_AGG(JSONB_BUILD_OBJECT('name', 1, 'age', 1, 'gender', 1, 'col1', 543534535,
                                                               'col2', 453545, 'col3', 43535345, 'col4', 54354354,
                                                               'col5', 453535, 'col6', 453534, 'col7', 32323, 'col8',
                                                               12122
                                            )) js
                            FROM generate_series(1, 100)) r
                   ON TRUE;
```

### Создаем индекс

```sql
CREATE INDEX IF NOT EXISTS ix_name ON annastavi.json USING GIN ((json_data));
```

### Смотрим размер таблицы и размер toast

```sql
select schemaname
     , relname
     , pg_size_pretty(pg_total_relation_size(psut.relid)) As "Total Size"
     , pg_size_pretty(pg_table_size(psut.relid))          As "Table Size"
     , pg_size_pretty(pg_indexes_size(psut.relid))        as "Index Size"
     , pg_size_pretty(pg_total_relation_size(psut.relid) - pg_relation_size(psut.relid) -
                      pg_indexes_size(psut.relid))        as "TOAST Size"
     , psut.last_vacuum
     , psut.last_autovacuum
     , psut.last_analyze
     , psut.last_autoanalyze
from pg_stat_user_tables psut
where 1 = 1
  and relname in ('json')
ORDER BY pg_total_relation_size(psut.relid) desc;
```

### Ответ

| schemaname | relname | Total Size | Table Size | Index Size | TOAST Size | last\_vacuum | last\_autovacuum                  | last\_analyze | last\_autoanalyze                 |
|:-----------|:--------|:-----------|:-----------|:-----------|:-----------|:-------------|:----------------------------------|:--------------|:----------------------------------|
| annastavi  | json    | 2130 MB    | 2128 MB    | 2200 kB    | 2041 MB    | null         | 2024-11-07 10:16:11.543504 +00:00 | null          | 2024-11-07 10:16:12.331133 +00:00 |


Размер TOAST вырос пропарционально таблице - он разрастается так же как и таблица

Чтобы избежать этого можно использовать команду VACUUM, чтобы очистить неиспользуемое пространство

```sql
VACUUM annastavi.json
```

| schemaname | relname | Total Size | Table Size | Index Size | TOAST Size | last\_vacuum                      | last\_autovacuum                  | last\_analyze | last\_autoanalyze                 |
|:-----------|:--------|:-----------|:-----------|:-----------|:-----------|:----------------------------------|:----------------------------------|:--------------|:----------------------------------|
| annastavi  | json    | 2130 MB    | 2128 MB    | 2200 kB    | 2041 MB    | 2024-11-07 10:25:28.591278 +00:00 | 2024-11-07 10:16:11.543504 +00:00 | null          | 2024-11-07 10:16:12.331133 +00:00 |

Сделала VACUUM - но ничего не изменилось.

Делаем VACUUM FULL

```sql
VACUUM FULL annastavi.json
```

| schemaname   | relname | Total Size | Table Size | Index Size | TOAST Size | last\_vacuum                      | last\_autovacuum                  | last\_analyze | last\_autoanalyze                 |
|:-------------|:--------|:-----------|:-----------|:-----------|:-----------|:----------------------------------|:----------------------------------|:--------------|:----------------------------------|
| annastavi    | json    | 1032 MB    | 1031 MB    | 1112 kB    | 987 MB     | 2024-11-07 10:25:28.591278 +00:00 | 2024-11-07 10:16:11.543504 +00:00 | null          | 2024-11-07 10:16:12.331133 +00:00 |

Размер таблицы и размер TOAST уменьшался в два раза.

Также можно сделать REINDEX, CLUSTER для перестройки индекса