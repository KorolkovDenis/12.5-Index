### Домашнее задание к занятию «Индексы» - Корольков Денис

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Ответ:

Наш запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц

```
SELECT TABLE_SCHEMA AS db_name, CONCAT(ROUND(SUM(index_length)*100 / (SUM(data_length+ index_length)), 2), ‘%’) AS ‘% index’
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = ‘sakila’;
```
![screen1](https://github.com/KorolkovDenis/12.5-Index/blob/main/screenshots/screen1.jpg)


### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Ответ:

Узкое место – излишняя обработка оконной функции over (partition by..  – таких таблиц rental, inventory, film. Т.к. нужно посчитать сумму платежей покупателей за конкретную дату, обработка и присоединение таблиц rental, inventory, film излишня. Всю необходимую информацию можно получить из таблиц payment и customer, соответственно, остальные таблицы удаляем.

Итог моего запроса:
```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id
```

![screen2](https://github.com/KorolkovDenis/12.5-Index/blob/main/screenshots/screen2.jpg)

Этими действиями мы уменьшили во много раз время обработки запроса actual time =4.5..4.52. А был первоначально actual time = 5900..5900.

По Вашим замечаниям немного подправил скрипт. Убрал over (partition by c.customer_id) - добавил группировку по имени group by name, оставил rental r и inventory i
```
select  concat(c.last_name, ' ', c.first_name) as name, sum(p.amount)
from payment p, rental r, customer c, inventory i
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
group by name
```
![screen4](https://github.com/KorolkovDenis/12.5-Index/blob/main/screenshots/screen4.jpg)

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

Ответ:

![screen3](https://github.com/KorolkovDenis/12.5-Index/blob/main/screenshots/screen3.jpg)


[Cсылка на google docs по «Индексы»](https://docs.google.com/document/d/1sR6GNwFNSvLkm7zELSIrXcB-ID9cX-5B/edit?usp=drive_link&ouid=104113173630640462528&rtpof=true&sd=true)