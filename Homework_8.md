### Создаем таблицу продаж

```sql
CREATE TABLE IF NOT EXISTS annastavi.sales
(
    id        SERIAL         NOT NULL
        CONSTRAINT pk_sales PRIMARY KEY,
    sale_date DATE           NOT NULL,
    amount    NUMERIC(10, 2) NOT NULL
);
```

### Генерируем данные

```sql
INSERT INTO annastavi.sales (id, sale_date, amount)
SELECT g, now(), 100
FROM generate_series(1, 1000) AS g;
```

### Создаем функцию

1 версия

```sql
CREATE OR REPLACE FUNCTION annastavi.thirdoftheyear_get(_sale_date DATE) RETURNS JSON
    STABLE SECURITY DEFINER
    LANGUAGE plpgsql
AS
$$
BEGIN

    RETURN JSON_BUILD_OBJECT('data', JSON_AGG(_js))
        FROM (SELECT CASE
                         WHEN DATE_PART('MONTH', s.sale_date)::INT BETWEEN 1 AND 4
                             THEN 'Первая треть года'
                         WHEN DATE_PART('MONTH', s.sale_date)::INT BETWEEN 5 AND 8
                             THEN 'Вторая треть года'
                         WHEN DATE_PART('MONTH', s.sale_date)::INT BETWEEN 9 AND 12
                             THEN 'Третья треть года'
                         ELSE NULL
                         END period,
                     s.id,
                     s.amount
              FROM annastavi.sales s
              WHERE s.sale_date = COALESCE(_sale_date, CURRENT_DATE)) _js;
END;
$$;
```

2 версия

```sql
CREATE OR REPLACE FUNCTION annastavi.thirdoftheyear_get_v1(_sale_date DATE) RETURNS JSON
    STABLE SECURITY DEFINER
    LANGUAGE plpgsql
AS
$$
DECLARE
    _period NUMERIC(5, 2) := 12 / EXTRACT(MONTH FROM COALESCE(_sale_date, CURRENT_DATE));
BEGIN
    RETURN JSON_BUILD_OBJECT('data', JSON_AGG(_js))
        FROM (SELECT CASE
                         WHEN _period < 2 THEN 'Третья'
                         WHEN _period >= 3 THEN 'Первая'
                         ELSE 'Вторая'
                         END period,
                     s.id,
                     s.amount
              FROM annastavi.sales s
              WHERE s.sale_date = COALESCE(_sale_date, CURRENT_DATE)) _js;
END;
$$;
``

### Вызываем функцию

```sql
SELECT annastavi.thirdoftheyear_get(_sale_date := '2024-10-29');
```

### Вызываем функцию в SELECT 

```sql
SELECT annastavi.thirdoftheyear_get(s.sale_date),
       s.id,
       s.amount
FROM annastavi.sales s
WHERE s.sale_date = '2024-10-29';
```