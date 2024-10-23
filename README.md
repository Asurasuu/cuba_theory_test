**Задание 1.** В БД mysql  есть одна единственная таблица «abc» с полями: id (int), name (varchar), cnt (int). В таблице содержится порядка 10 млн записей. Что нужно сделать, что быстро работали следующие запросы (3 разных случая)? Все остальные факторы кроме скорости чтения не критичны.

```sql
SELECT * FROM abc WHERE name = 'xxx' AND cnt = yyy
SELECT * FROM abc WHERE cnt = xxx AND name LIKE 'yyy%'
SELECT * FROM abc ORDER BY cnt ASC
```

**Ответы:**

1 Запрос:

Создать комбинированный индекс из name и cnt 

```sql
CREATE INDEX idx_name_cnt ON abc(name, cnt);
```

2 Запрос:

Создать индекс на каждый из параметров

```sql
CREATE INDEX idx_cnt ON abc(cnt);
CREATE INDEX idx_name ON abc(name);
```

3 Запрос:

Создать индекс из cnt

```sql
CREATE INDEX idx_cnt ON abc(cnt);
```

---

**Задание 2.**  У вас есть задача обработки большого объема данных в PHP (например, парсинг CSV-файлов объемом в несколько гигабайт). Как вы будете обрабатывать этот файл, чтобы избежать превышения лимита памяти, и какие функции PHP и подходы для этого будете использовать?

**Ответ:**

Я бы использова fgetcsv(), дабы не загружать весь файл в память.

```php
$handle = fopen('file.csv', 'r');
if ($handle !== false) {
    while (($data = fgetcsv($handle, 1000, ',')) !== false) {
        // Обработка строки $data
    }
    fclose($handle);
}
```

Так же можно использовать конфигурацию php.ini, для того чтобы увеличить лимит память и отключить ограничение время выполнения скрипта, но не стоит этим злоупотреблять.

```php
ini_set('memory_limit', '512M');
set_time_limit(0);
```

---

**Задание 3.** Есть код и запрос к БД, чтобы вы в нем изменили? Почему?
```php
$DB->query("SELECT * FROM abc WHERE id=" . $_GET['id']);
```

**Ответ:**

Привел бы код из задания в такой вид: 

(Предполагается, что мы используем PDO. Все пояснения в комментариях php)

```php

$id = $_GET['id'];

// Необходимо провалидировать входящие данные, дабы обезопаситься от SQL-иньекции
if( !is_numeric($id) ) {
    die("Неверный id");
}

// Сам запрос следует занести в try/catch, чтобы пользователь не видел ошибок в случае возникновения таковых при выполнении запроса
try {
    $res = $DB->fetchAll("SELECT * FROM abc WHERE id=:id", [':id' => $id]);

    // Можно обработать отсутсвие результата, но не прям чтобы обязательно
    if( !$res ) {
        echo "Нет записей";
    }
    
} catch(PDOException $ex) {
    // Обработка ошибки
}

```

---

**Задание 4.**

В БД есть таблица заказов (orders) с полями:

- date - дата оформления заказа
- customer_name - имя клиента
- order_price - сумма заказа
 
Напиши sql запросы для выборки:
1. Запрос, который покажет сколько денег принес каждый отдельно взятый покупатель с группировкой по месяцам.
2. Запрос, который выведет  имена клиентов, у которых суммарные покупки за весь период превысили 10 тыс. руб. и одновременно никогда не было заказов менее 500 руб.

**Ответы:**

_Запрос 1:_

```sql
SELECT customer_name, DATE_FORMAT(date, '%Y-%m') AS order_month, SUM(order_price) AS total_amount
FROM orders
GROUP BY customer_name, order_month
ORDER BY order_month, customer_name;
```

_Запрос 2:_

```sql
SELECT customer_name
FROM orders
GROUP BY customer_name
HAVING SUM(order_price) > 10000 AND MIN(order_price) >= 500;
```
