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
