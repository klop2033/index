# Домашнее задание «Индексы» - Фролов КС

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ 

```sql
SELECT
    SUM(index_length) AS razindex,
    SUM(data_length + index_length) AS razdan,
    SUM(index_length) / SUM(data_length + index_length) * 100 AS indexproc
FROM information_schema.TABLES
WHERE table_schema = 'sakila';)

или 

SELECT
    SUM(index_length) / SUM(data_length + index_length) * 100 AS indexproc
FROM information_schema.TABLES
WHERE table_schema = 'sakila';)

```

Ответ: 35,3511

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Ответ 

Запросом мы пытаемся получить список клиентов с суммами их платежей за 30 июля 2005 года. В запросе используется f.title в оконной функции, но нет соединения между inventory и film.Это приведет к некорректным результатам. Они должны быть связаны через film_id. Использование date(p.payment_date) препятствует использованию индексов.Функция date(p.payment_date) преобразует все значения в даты, а затем сравнивает с '2005-07-30'. Можно создать индекс для быстрого поиска в диапазоне и не преобразовывать значения.

```sql
--- Индекс для 
CREATE INDEX idx_payment_date ON payment(payment_date);

--- Запрос
EXPLAIN ANALYZE
SELECT 
    DISTINCT concat(c.last_name, ' ', c.first_name) AS customer_name,
    sum(p.amount) OVER (PARTITION BY c.customer_id) AS total_amount
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
WHERE p.payment_date >= '2005-07-30' 
  AND p.payment_date < '2005-07-31';

```

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

### Ответ 

GIN, GiST, SP-GiST, BRIN