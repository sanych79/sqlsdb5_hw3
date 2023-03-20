# ` Домашнее задание к занятию 12.5. «Индексы» - Борисов Александр`

## Задание 1 Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

`Если я правильно понял задание то так:`

`select SUM(index_length)/SUM(DATA_LENGTH)*100 
FROM information_schema.TABLES
WHERE table_schema= 'sakila'`


## Задание 2 Выполните explain analyze следующего запроса:
#
#### select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
#### from payment p, rental r, customer c, inventory i, film f
#### where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.
#### customer_id = c.customer_id and i.inventory_id = r.inventory_id
#

### 1) перечислите узкие места;
        1.1) Inner hash join (no condition)  (cost=1581474.80 rows=15813000) (actual time=0.336..54.244 rows=634000 loops=1)
            `Присоединение таблицы Film без условий. Увеличивает количество строк в 1000 раз. Что потом и дает замедление по остальным шагам запроса. 
            
            Судя по тому, что результат это итого потраченные деньги покупателями за '2005-07-30', таблица Filmи вовсе не нужна, как и таблица inventory`
        1.2) 
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title ) (actual time=2338.923..5921.984 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title (actual time=2338.888..2419.755 rows=642000 loops=1)

        Агрегатная функция с окошком в котором происходит сортировка по двум параметрам. Совсем не понимаю, зачем это тут. Можно же проще агрегировать с группировкой по одному параметру.

        1.3) 
        -> Temporary table with deduplication (cost=0.00..0.00 rows=0) (actual time=6138.331..6138.331 rows=391 loops=1)

        Потрачено время на убирание дублей функцией distinct. 

        ИТОГО потрачено actual time=6138.331
        
### 2) оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы

    2.1) Если добавить условие на присоединение таблица Films мы значительно увеличим производительность (and i.film_id  = f.film_id)
    ИТОГО потрачено actual time=11.506
    2.2) Если убрать таблицу с фильмами, inventory, окно и distinct, то добьемся actual time=8.597.
     мой код был бы такой:
     
    select concat(c.last_name, ' ', c.first_name), sum(p.amount)
	    from payment p
	    join rental r on p.payment_date = r.rental_date 
	    join customer c on r.customer_id=c.customer_id 
    where date(p.payment_date) = '2005-07-30'
    group by concat(c.last_name, ' ', c.first_name)

    2.3) Хотел добавить составной индекс  CREATE INDEX indx_payment_date ON payment(payment_id, payment_date), но он ничего не дает в итоге.


## Задание 3 Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

`В Постгресе используются типы индексов `

`1) B-Tree`
`2) Hash - как в MySQL`
 
`4) GiST`
`5) SP-GiST`
`6) GIN`
`7) BRIN - свои индексы`
