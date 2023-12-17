# Relational-Databases-and-Database-Administration_HW12-05

Задние 1.

```select sum(index_length) / sum(data_length) * 100 from information_schema.tables;```

Задние 2.

Создаем индекс для оптимизации запроса: 

```create index payment_date on payment(payment_date);```

Сокращаем запросы 640000 проверяемых строк данных

```select concat(c.last_name, ' ', c.first_name) as customers, sum(p.amount) from customer c join rental r on c.customer_id = r.customer_id join payment p on r.rental_date = p.payment_date where date(p.payment_date) = '2005-07-30' group by c.customer_id;```

Получаем

```explain analyze select concat(c.last_name, ' ', c.first_name) as client, sum(p.amount) from customer c join rental r on c.customer_id = r.customer_id join payment p on r.rental_date = p.payment_date where date(p.payment_date) = '2005-07-30' group by c.customer_id;```

запрос до 

```| -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=7806..7806 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=7806..7806 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=3098..7525 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=3098..3219 rows=642000 loops=1)
                -> Stream results  (cost=21.3e+6 rows=16e+6) (actual time=0.452..2289 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=21.3e+6 rows=16e+6) (actual time=0.448..1827 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=19.7e+6 rows=16e+6) (actual time=0.445..1560 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=18.1e+6 rows=16e+6) (actual time=0.441..1276 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.54e+6 rows=15.4e+6) (actual time=0.432..69.9 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.61 rows=15400) (actual time=0.0385..10.3 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.61 rows=15400) (actual time=0.0247..6.14 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=0.0313..0.29 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1.04) (actual time=0.00121..0.00169 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=226e-6..253e-6 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=203e-6..230e-6 rows=1 loops=642000)
 |
```

запрос после 

```| -> Table scan on <temporary>  (actual time=12.1..12.1 rows=391 loops=1)
    -> Aggregate using temporary table  (actual time=12.1..12.1 rows=391 loops=1)
        -> Nested loop inner join  (cost=12610 rows=15990) (actual time=0.091..11.2 rows=642 loops=1)
            -> Nested loop inner join  (cost=7014 rows=15990) (actual time=0.0833..10.4 rows=642 loops=1)
                -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1564 rows=15400) (actual time=0.069..8.72 rows=634 loops=1)
                    -> Table scan on p  (cost=1564 rows=15400) (actual time=0.0509..5.62 rows=16044 loops=1)
                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.25 rows=1.04) (actual time=0.00177..0.00238 rows=1.01 loops=634)
            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.00104..0.00106 rows=1 loops=642)
 |
```
