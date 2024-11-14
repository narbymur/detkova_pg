### План запроса без индексов

| QUERY PLAN |
| :--- |
| Limit  \(cost=957504.90..957504.93 rows=10 width=56\) \(actual time=98027.610..98027.918 rows=10 loops=1\) |
|   -&gt;  Sort  \(cost=957504.90..957505.40 rows=200 width=56\) \(actual time=98027.608..98027.916 rows=10 loops=1\) |
|         Sort Key: r.startdate |
|         Sort Method: top-N heapsort  Memory: 25kB |
|         -&gt;  Group  \(cost=952189.95..957500.58 rows=200 width=56\) \(actual time=30007.887..97914.948 rows=1500000 loops=1\) |
|               Group Key: r.id, \(\(\(bs.city \|\| ', '::text\) \|\| bs.name\)\), \(count\(t.id\)\), \(count\(s\_1.id\)\) |
|               -&gt;  Incremental Sort  \(cost=952189.95..957497.58 rows=200 width=56\) \(actual time=30007.886..97750.717 rows=1500000 loops=1\) |
|                     Sort Key: r.id, \(\(\(bs.city \|\| ', '::text\) \|\| bs.name\)\), \(count\(t.id\)\), \(count\(s\_1.id\)\) |
|                     Presorted Key: r.id |
|                     Full-sort Groups: 46875  Sort Method: quicksort  Average Memory: 27kB  Peak Memory: 27kB |
|                     -&gt;  Nested Loop  \(cost=952163.32..957488.58 rows=200 width=56\) \(actual time=30007.710..97516.068 rows=1500000 loops=1\) |
|                           Join Filter: \(r.fkbus = s\_1.fkbus\) |
|                           Rows Removed by Join Filter: 2502123 |
|                           -&gt;  Nested Loop  \(cost=952158.32..957469.52 rows=200 width=84\) \(actual time=30007.064..97091.459 rows=1500000 loops=1\) |
|                                 Join Filter: \(bs.id = br.fkbusstationfrom\) |
|                                 Rows Removed by Join Filter: 6150000 |
|                                 -&gt;  Nested Loop  \(cost=952158.32..957440.85 rows=200 width=24\) \(actual time=30007.057..96522.356 rows=1500000 loops=1\) |
|                                       Join Filter: \(br.id = s.fkroute\) |
|                                       Rows Removed by Join Filter: 44250000 |
|                                       -&gt;  Nested Loop  \(cost=952158.32..957261.71 rows=200 width=24\) \(actual time=30007.028..93852.631 rows=1500000 loops=1\) |
|                                             Join Filter: \(s.id = r.fkschedule\) |
|                                             Rows Removed by Join Filter: 1124250000 |
|                                             -&gt;  Nested Loop  \(cost=952158.32..952730.96 rows=200 width=24\) \(actual time=30006.998..37016.361 rows=1500000 loops=1\) |
|                                                   -&gt;  Finalize GroupAggregate  \(cost=952157.89..952208.56 rows=200 width=12\) \(actual time=30000.750..30954.686 rows=1500000 loops=1\) |
|                                                         Group Key: t.fkride |
|                                                         -&gt;  Gather Merge  \(cost=952157.89..952204.56 rows=400 width=12\) \(actual time=30000.730..30647.623 rows=4500000 loops=1\) |
|                                                               Workers Planned: 2 |
|                                                               Workers Launched: 2 |
|                                                               -&gt;  Sort  \(cost=951157.87..951158.37 rows=200 width=12\) \(actual time=29962.698..30264.362 rows=1500000 loops=3\) |
|                                                                     Sort Key: t.fkride |
|                                                                     Sort Method: external merge  Disk: 38168kB |
|                                                                     Worker 0:  Sort Method: external merge  Disk: 38168kB |
|                                                                     Worker 1:  Sort Method: external merge  Disk: 38168kB |
|                                                                     -&gt;  Partial HashAggregate  \(cost=951148.22..951150.22 rows=200 width=12\) \(actual time=26265.392..29590.813 rows=1500000 loops=3\) |
|                                                                           Group Key: t.fkride |
|                                                                           Batches: 5  Memory Usage: 49217kB  Disk Usage: 420760kB |
|                                                                           Worker 0:  Batches: 5  Memory Usage: 50241kB  Disk Usage: 414664kB |
|                                                                           Worker 1:  Batches: 5  Memory Usage: 50241kB  Disk Usage: 415504kB |
|                                                                           -&gt;  Parallel Seq Scan on tickets t  \(cost=0.00..838653.48 rows=22498948 width=12\) \(actual time=0.450..19753.812 rows=17999158 loops=3\) |
|                                                   -&gt;  Index Scan using ride\_pkey on ride r  \(cost=0.43..2.61 rows=1 width=16\) \(actual time=0.004..0.004 rows=1 loops=1500000\) |
|                                                         Index Cond: \(id = t.fkride\) |
|                                             -&gt;  Materialize  \(cost=0.00..34.50 rows=1500 width=8\) \(actual time=0.000..0.017 rows=750 loops=1500000\) |
|                                                   -&gt;  Seq Scan on schedule s  \(cost=0.00..27.00 rows=1500 width=8\) \(actual time=0.020..0.289 rows=1500 loops=1\) |
|                                       -&gt;  Materialize  \(cost=0.00..1.90 rows=60 width=8\) \(actual time=0.000..0.001 rows=30 loops=1500000\) |
|                                             -&gt;  Seq Scan on busroute br  \(cost=0.00..1.60 rows=60 width=8\) \(actual time=0.024..0.027 rows=60 loops=1\) |
|                                 -&gt;  Materialize  \(cost=0.00..1.15 rows=10 width=68\) \(actual time=0.000..0.000 rows=5 loops=1500000\) |
|                                       -&gt;  Seq Scan on busstation bs  \(cost=0.00..1.10 rows=10 width=68\) \(actual time=0.005..0.005 rows=10 loops=1\) |
|                           -&gt;  Materialize  \(cost=5.00..5.08 rows=5 width=12\) \(actual time=0.000..0.000 rows=3 loops=1500000\) |
|                                 -&gt;  HashAggregate  \(cost=5.00..5.05 rows=5 width=12\) \(actual time=0.057..0.058 rows=5 loops=1\) |
|                                       Group Key: s\_1.fkbus |
|                                       Batches: 1  Memory Usage: 24kB |
|                                       -&gt;  Seq Scan on seat s\_1  \(cost=0.00..4.00 rows=200 width=8\) \(actual time=0.013..0.026 rows=200 loops=1\) |
| Planning Time: 7.033 ms |
| Execution Time: 98090.372 ms |

### Навесим индексы на внешние ключи

```sql
CREATE INDEX IF NOT EXISTS ix_ride_fkschedule ON book.ride (fkschedule, id, fkbus, startdate);
CREATE INDEX IF NOT EXISTS ix_seat_fkbus ON book.seat (id, fkbus);
CREATE INDEX IF NOT EXISTS ix_tickets_fkride ON book.tickets (id, fkride);
CREATE INDEX IF NOT EXISTS ix_schedule_fkroute ON book.schedule (id, fkroute);
CREATE INDEX IF NOT EXISTS ix_busroute_fkbusstationfrom ON book.busroute (id, fkbusstationfrom);
```

### Прогоняем ANALYZE 

```sql
ANALYZE book.ride;
ANALYZE book.schedule;
ANALYZE book.busroute;
ANALYZE book.tickets;
ANALYZE book.seat;
```
### План запроса с индексами

| QUERY PLAN |
| :--- |
| Limit  \(cost=2482038.46..2482038.48 rows=10 width=56\) \(actual time=15246.253..15246.325 rows=10 loops=1\) |
|   -&gt;  Sort  \(cost=2482038.46..2485800.19 rows=1504692 width=56\) \(actual time=15246.249..15246.320 rows=10 loops=1\) |
|         Sort Key: r.startdate |
|         Sort Method: top-N heapsort  Memory: 26kB |
|         -&gt;  HashAggregate  \(cost=2397563.71..2449522.60 rows=1504692 width=56\) \(actual time=14749.409..15140.091 rows=1500000 loops=1\) |
|               Group Key: r.id, \(\(bs.city \|\| ', '::text\) \|\| bs.name\), \(count\(t.id\)\), \(count\(s\_1.id\)\) |
|               Planned Partitions: 8  Batches: 9  Memory Usage: 57425kB  Disk Usage: 73056kB |
|               -&gt;  Hash Join  \(cost=2221054.54..2320095.58 rows=1504692 width=56\) \(actual time=12267.117..14249.623 rows=1500000 loops=1\) |
|                     Hash Cond: \(r.fkbus = s\_1.fkbus\) |
|                     -&gt;  Hash Join  \(cost=2221049.43..2305269.26 rows=1504692 width=36\) \(actual time=12266.464..14057.411 rows=1500000 loops=1\) |
|                           Hash Cond: \(br.fkbusstationfrom = bs.id\) |
|                           -&gt;  Hash Join  \(cost=2221048.20..2299644.25 rows=1504692 width=24\) \(actual time=12266.145..13936.833 rows=1500000 loops=1\) |
|                                 Hash Cond: \(s.fkroute = br.id\) |
|                                 -&gt;  Hash Join  \(cost=2221045.85..2295413.09 rows=1504692 width=24\) \(actual time=12265.826..13802.119 rows=1500000 loops=1\) |
|                                       Hash Cond: \(r.fkschedule = s.id\) |
|                                       -&gt;  Hash Join  \(cost=2221000.10..2291406.36 rows=1504692 width=24\) \(actual time=12263.873..13651.733 rows=1500000 loops=1\) |
|                                             Hash Cond: \(t.fkride = r.id\) |
|                                             -&gt;  Finalize HashAggregate  \(cost=2171816.10..2216251.54 rows=1504692 width=12\) \(actual time=11793.797..12507.740 rows=1500000 loops=1\) |
|                                                   Group Key: t.fkride |
|                                                   Planned Partitions: 4  Batches: 5  Memory Usage: 49201kB  Disk Usage: 120600kB |
|                                                   -&gt;  Gather  \(cost=1528548.13..2064254.13 rows=3009384 width=12\) \(actual time=7763.168..10996.449 rows=4499949 loops=1\) |
|                                                         Workers Planned: 2 |
|                                                         Workers Launched: 2 |
|                                                         -&gt;  Partial HashAggregate  \(cost=1527548.13..1762315.73 rows=1504692 width=12\) \(actual time=7750.486..10872.294 rows=1499983 loops=3\) |
|                                                               Group Key: t.fkride |
|                                                               Planned Partitions: 4  Batches: 5  Memory Usage: 50225kB  Disk Usage: 407400kB |
|                                                               Worker 0:  Batches: 5  Memory Usage: 50225kB  Disk Usage: 407512kB |
|                                                               Worker 1:  Batches: 5  Memory Usage: 50225kB  Disk Usage: 408424kB |
|                                                               -&gt;  Parallel Index Only Scan using idx\_tickets\_fkride on tickets t  \(cost=0.56..723370.43 rows=22499398 width=12\) \(actual time=0.854..4240.887 rows=17999158 loops=3\) |
|                                                                     Heap Fetches: 0 |
|                                             -&gt;  Hash  \(cost=23109.00..23109.00 rows=1500000 width=16\) \(actual time=468.652..468.652 rows=1500000 loops=1\) |
|                                                   Buckets: 1048576  Batches: 2  Memory Usage: 43378kB |
|                                                   -&gt;  Seq Scan on ride r  \(cost=0.00..23109.00 rows=1500000 width=16\) \(actual time=1.114..249.546 rows=1500000 loops=1\) |
|                                       -&gt;  Hash  \(cost=27.00..27.00 rows=1500 width=8\) \(actual time=1.916..1.917 rows=1500 loops=1\) |
|                                             Buckets: 2048  Batches: 1  Memory Usage: 75kB |
|                                             -&gt;  Seq Scan on schedule s  \(cost=0.00..27.00 rows=1500 width=8\) \(actual time=0.228..1.625 rows=1500 loops=1\) |
|                                 -&gt;  Hash  \(cost=1.60..1.60 rows=60 width=8\) \(actual time=0.261..0.262 rows=60 loops=1\) |
|                                       Buckets: 1024  Batches: 1  Memory Usage: 11kB |
|                                       -&gt;  Seq Scan on busroute br  \(cost=0.00..1.60 rows=60 width=8\) \(actual time=0.201..0.208 rows=60 loops=1\) |
|                           -&gt;  Hash  \(cost=1.10..1.10 rows=10 width=20\) \(actual time=0.074..0.074 rows=10 loops=1\) |
|                                 Buckets: 1024  Batches: 1  Memory Usage: 9kB |
|                                 -&gt;  Seq Scan on busstation bs  \(cost=0.00..1.10 rows=10 width=20\) \(actual time=0.051..0.053 rows=10 loops=1\) |
|                     -&gt;  Hash  \(cost=5.05..5.05 rows=5 width=12\) \(actual time=0.611..0.611 rows=5 loops=1\) |
|                           Buckets: 1024  Batches: 1  Memory Usage: 9kB |
|                           -&gt;  HashAggregate  \(cost=5.00..5.05 rows=5 width=12\) \(actual time=0.598..0.599 rows=5 loops=1\) |
|                                 Group Key: s\_1.fkbus |
|                                 Batches: 1  Memory Usage: 24kB |
|                                 -&gt;  Seq Scan on seat s\_1  \(cost=0.00..4.00 rows=200 width=8\) \(actual time=0.488..0.545 rows=200 loops=1\) |
| Planning Time: 8.313 ms |
| Execution Time: 15288.739 ms |

