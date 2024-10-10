CREATE TABLE annastavi.name
(
    name VARCHAR(500) NOT NULL
);

INSERT INTO annastavi.name (name)
SELECT CONCAT('anna', g)
FROM generate_series(1, 1000000) as g;

select count(*)
from annastavi.name; --1 000 000;

select table_name,
       pg_size_pretty(pg_total_relation_size(quote_ident(table_name)))
from information_schema.tables
where table_name = 'name'
  and table_schema = 'annastavi';
--42 MB

UPDATE annastavi.name n
SET name = CONCAT(n.name, random())
FROM generate_series(1, 5);

select *
from annastavi.name;

select *
from pg_stat_user_tables t
where t.relname = 'name'
  and t.schemaname = 'annastavi';

UPDATE annastavi.name n
SET name = CONCAT(n.name, random())
FROM generate_series(1, 5);

select *
from pg_stat_user_tables t
where t.relname = 'name'
  and t.schemaname = 'annastavi';

select table_name,
       pg_size_pretty(pg_total_relation_size(quote_ident(table_name)))
from information_schema.tables
where table_name = 'name'
  and table_schema = 'annastavi';
--269 MB

ALTER TABLE annastavi.name SET (AUTOVACUUM_ENABLED = OFF);

UPDATE annastavi.name n
SET name = n.name
FROM generate_series(1, 5);

UPDATE annastavi.name n
SET name = CONCAT(n.name, random())
FROM generate_series(1, 5);

select table_name,
       pg_size_pretty(pg_total_relation_size(quote_ident(table_name)))
from information_schema.tables
where table_name = 'name'
  and table_schema = 'annastavi';
--751 MB

select *
from pg_stat_user_tables t
where t.relname = 'name'
  and t.schemaname = 'annastavi';

ALTER TABLE annastavi.name SET (AUTOVACUUM_ENABLED = ON);

--Количество символов в таблице изменилась и так как мы обновляли таблицу, появился блот. 