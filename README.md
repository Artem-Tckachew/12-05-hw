# «Индексы» - Ткачев Артём

---

## Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Ответ: 

``` 

select sum(data_length) as 'Общий размер таблиц', sum(index_length) as 'Общий размер индексов', sum(index_length)*100.0/sum(data_length) as 'Процентное отношение'
from information_schema.tables
where table_schema='sakila' and data_length is not null;

 ```

![alt text](https://github.com/Artem-Tckachew/12-05-hw/blob/main/1.png)

---

## Задание 2

Выполните explain analyze следующего запроса:

``` 

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id

 ```

* перечислите узкие места;
* оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Ответ: 

* Запрос слишком сложный и в нём используются избыточные таблицы Из запроса убираем фрагмент over (partition by c.customer_id, f.title) и оператор distinct. Так же убираем избыточные таблицы film и inventory :

``` 

select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
group by c.last_name, c.first_name;

 ```

* Можно выполнить объединение таблиц в запросе, использую оператор join :

``` 

select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from customer c
join rental r on r.customer_id = c.customer_id
join payment p on p.payment_date = r.rental_date and date(p.payment_date) = '2005-07-30'
group by c.last_name, c.first_name;

 ```

* Для таблицы payment на столбец payment_date можно создать индекс : Смотрим существующие для таблицы индексы и создаём новый индекс

``` 

select *
from INFORMATION_SCHEMA.STATISTICS
where TABLE_NAME='payment';

create index idx_payment_date on payment(payment_date);

select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where p.payment_date >= '2005-07-30 00:00:00' and p.payment_date < date_add('2005-07-30 00:00:00', interval 1 day) and p.payment_date = r.rental_date and r.customer_id = c.customer_id
group by c.last_name, c.first_name;

drop index idx_payment_date on payment;

 ```

## анализ запроса без индекса

![alt text](https://github.com/Artem-Tckachew/12-05-hw/blob/main/2.png)

## анализ запроса с индексом

![alt text](https://github.com/Artem-Tckachew/12-05-hw/blob/main/3.png)


---