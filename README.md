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

**Задание 2.**  У вас есть задача обработки большого объема данных в PHP (например, парсинг CSV-файлов объемом в несколько гигабайт). Как вы будете обрабатывать этот файл, чтобы избежать превышения лимита памяти, и какие функции PHP и подходы для этого ,eltnt использовать?

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
