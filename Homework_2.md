--1 сессия
CREATE SCHEMA IF NOT EXISTS annastavi;

CREATE TABLE IF NOT EXISTS annastavi.aboba
(
    id   INTEGER NOT NULL,
    name VARCHAR NOT NULL
);

insert into annastavi.aboba (id, name)
VALUES (1, 'aboba1'), (2, 'aboba2'), (3, 'aboba3');

select * from annastavi.aboba;

show transaction isolation level; --read committed

begin;

insert into annastavi.aboba (id, name)
VALUES (6, 'aboba3');

commit;
end;

BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

end;


--2 сессия
begin;

select * from annastavi.aboba;

--Новую запись не вижу, потому что при уровне изоляции read committed мы видим только закомиченные данные

select * from annastavi.aboba;

--Теперь видим новую запись, так как мы завершили транзакцию, тем самым закомители изменения

end;

BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

select * from annastavi.aboba;

--Нет, не видим, мы не видим не закомиченные данные

select * from annastavi.aboba;

--Не увидели новую запись, так как уровень изоляции REPEATABLE READ держит снимок данных от первого запроса после BEGIN и не прогоняет его