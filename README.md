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

---

**Задание 5.**

Есть две javascript-функции:
```js
function f(a,b) { return a+b }
```
и
```js
var f = function(a,b) { return a+b }
```
Есть ли между ними разница? Если есть то какая?

**Ответ:** Разница между двумя функциями есть.

- Если мы определяем функцию как в 1 варианте, то мы можем использовать функци до её фактического объявления в коде.
- Если же мы попробуем использовать функцию определенную по 2 варианту, то получим ошибку.
- Так же, функцию из 1 варианта мы можем увидеть в стеке вызовов, что в свою очередь полезно при отладке.
- А функцию из 2 варианта мы не увидим в стекек вызовов, т.к. она анонимная, там будет только имя переменной f 

---

**Задание 6.** Чем принципиально отличаются между собой условия LEFT JOIN и INNER JOIN в sql? Какой вариант JOIN может дать потенциально больше результатов (строк) и почему?

**Ответ:**

INNER JOIN: Возвращает только те строки, которые имеют совпадения в обеих таблицах. Это означает, что если в одной из таблиц нет соответствующей строки, она не будет включена в результирующий набор.

LEFT JOIN: Возвращает все строки из левой таблицы и соответствующие строки из правой таблицы. Если в правой таблице нет совпадений, в результирующем наборе будут NULL значения для столбцов правой таблицы.

LEFT JOIN потенциально может отдать больше строк, из-за того, что он включает все строки из левой таблицы, даже если не будет соответствующих записей из правой таблицы.

---

**Задание 7.** Вам нужно реализовать консольный php-скрипт на сервере под Unix, который бы выводил каждые 15 секунд фразу «Hello». После вывода «Hello» скрипт всегда завершает работу. Напишите этот скрипт и пошагово расскажите, что нужно сделать, чтобы выполнялись исходные условия. 

(P.S. Не особо понял формулировку задания)

1. Пишем файл для вывода. Назовем dae.php
```php
#!/usr/bin/php
<?php

// Устанавливаем обработчик сигналов для корректного завершения
pcntl_signal(SIGTERM, function() {
    exit;
});

// Бесконечный цикл
while (true) {
    // Выводим фразу "Hello"
    echo "Hello\n";

    // Задержка на 15 секунд
    sleep(15);
}
?>
```

2. Даем права на выполнение этому файлу
```shell
chmod +x ./dae.php
```

3. Запускаем
```shell
./dae.php &
```

---

**Задание  8.** Напишите на php функцию для распределения рублевой скидки по купону на все товары в корзине пропорционально стоимости товара. На входе в функцию передаются два параметра: размер скидки в рублях (!) и массив из цен товаров, на выходе тот же массив цен, но уже с учетом скидки: distribute_discount(int $discount, array $prices) → return array $prices;

```php
function distribute_discount(int $discount, array $prices): array {
    // Сумма всех цен товаров
    $totalPrice = array_sum($prices);
    
    // Если общая сумма равна нулю, возвращаем оригинальный массив
    if ($totalPrice === 0) {
        return $prices;
    }

    // Массив для хранения новых цен
    $discountedPrices = [];

    // Распределяем скидку по каждому товару
    foreach ($prices as $price) {
        // Вычисляем долю скидки для текущего товара
        $discountForItem = ($price / $totalPrice) * $discount;
        
        // Вычисляем новую цену с учетом скидки
        $newPrice = max(0, $price - $discountForItem); // Не допускаем отрицательных цен
        
        // Добавляем новую цену в массив
        $discountedPrices[] = round($newPrice, 2); // Округляем до двух знаков после запятой
    }

    return $discountedPrices;
}

// Пример использования функции
$prices = [100, 200, 300];
$discount = 100;

$newPrices = distribute_discount($discount, $prices);
print_r($newPrices);
?>

```
