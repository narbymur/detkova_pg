### Средняя зарплата сотрудников

```sql
SELECT g.value                                   AS grade,
       AVG(s.amount) OVER (PARTITION BY g.value) AS average_salary
FROM annastavi.salary s
         INNER JOIN annastavi.grade g ON s.fk_grade = g.id
GROUP BY g.value, s.amount;
```

### Ответ

| grade  | average\_salary |
|:-------|:----------------|
| junior | 100000          |
| middle | 200000          |
| senoir | 300000          |

### Максимальная и минимальная зарплата 

```sql
SELECT DISTINCT ON (e.first_name) e.first_name,
       e.last_name,
       MAX(s.amount) OVER (PARTITION BY s.fk_employee) AS max_salary,
       MIN(s.amount) OVER (PARTITION BY s.fk_employee) AS min_salary
FROM annastavi.employee e
         INNER JOIN annastavi.salary s
                    ON e.id = s.fk_employee;
```

### Ответ

| first\_name | last\_name | max\_salary | min\_salary |
|:------------|:-----------|:------------|:------------|
| Eugene      | Aristov    | 300000      | 100000      |
| Ivan        | Ivanov     | 200000      | 200000      |
| Petr        | Petrov     | 200000      | 200000      |


### Текущая зарплата сотрудников вместе с грейдом

```sql
WITH cte AS (SELECT e.first_name,
                    e.last_name,
                    s.amount,
                    g.value                                                         AS grade,
                    ROW_NUMBER() OVER (PARTITION BY e.id ORDER BY s.from_date DESC) AS rn
             FROM annastavi.employee e
                      INNER JOIN annastavi.salary s
                                 ON e.id = s.fk_employee
                      INNER JOIN annastavi.grade g
                                 ON s.fk_grade = g.id
                      INNER JOIN annastavi.title tl
                                 ON tl.id = s.fk_employee
                      INNER JOIN annastavi.title_name tln
                                 ON tln.id = tl.fk_titlename)
SELECT *
FROM cte c
WHERE c.rn = 1;
```

### Ответ

| first\_name | last\_name | amount | grade  | rn  |
|:------------|:-----------|:-------|:-------|:----|
| Eugene      | Aristov    | 300000 | senoir | 1   |
| Ivan        | Ivanov     | 200000 | middle | 1   |
| Petr        | Petrov     | 200000 | middle | 1   |

### Разница зарплат сотрудников

```sql
SELECT e.first_name,
       e.last_name,
       s.amount,
       g.value    AS grade,
       COALESCE(s.amount - LAG(s.amount) OVER (ORDER BY s.amount ASC), 0) delta_salary
FROM annastavi.employee e
         INNER JOIN annastavi.salary s
                    ON e.id = s.fk_employee
         INNER JOIN annastavi.grade g
                    ON s.fk_grade = g.id
         INNER JOIN annastavi.title tl
                    ON tl.id = s.fk_employee
         INNER JOIN annastavi.title_name tln
                    ON tln.id = tl.fk_titlename;
```

### Ответ 

| first\_name | last\_name | amount | grade  | delta\_salary |
|:------------|:-----------|:-------|:-------|:--------------|
| Eugene      | Aristov    | 100000 | junior | 0             |
| Eugene      | Aristov    | 200000 | middle | 100000        |
| Ivan        | Ivanov     | 200000 | middle | 0             |
| Petr        | Petrov     | 200000 | middle | 0             |
| Eugene      | Aristov    | 300000 | senoir | 100000        |
