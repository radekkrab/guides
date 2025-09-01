# 🚀 Гайд по оптимизации MySQL запросов

## 📊 Содержание
- [Анализ запросов](#-анализ-запросов)
- [Оптимизация индексов](#-оптимизация-индексов)
- [Оптимизация JOIN](#-оптимизация-join)
- [Оптимизация WHERE](#-оптимизация-where)
- [Оптимизация GROUP BY и ORDER BY](#-оптимизация-group-by-и-order-by)
- [Работа с подзапросами](#-работа-с-подзапросами)
- [Временные таблицы](#-временные-таблицы)
- [Мониторинг](#-мониторинг)
- [Практические паттерны](#-практические-паттерны)

## 🔍 Анализ запросов

### EXPLAIN - основной инструмент
```sql
-- Базовый анализ
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Детальный анализ (MySQL 8.0+)
EXPLAIN ANALYZE 
SELECT * FROM orders WHERE user_id = 123;

-- JSON формат для детальной информации
EXPLAIN FORMAT=JSON
SELECT u.name, COUNT(o.id) 
FROM users u 
JOIN orders o ON u.id = o.user_id 
GROUP BY u.id;
```

### Ключевые поля в EXPLAIN:
- **type**: const, eq_ref, ref, range, index, ALL
- **key**: Используемый индекс
- **rows**: Оценочное количество строк
- **Extra**: Дополнительная информация (Using filesort, Using temporary)

### Включение профайлинга
```sql
-- Включение профайлинга для сессии
SET profiling = 1;

-- Выполнение запроса
SELECT * FROM large_table WHERE condition;

-- Просмотр статистики
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;
```

## 🗂️ Оптимизация индексов

### Создание эффективных индексов
```sql
-- Базовые индексы
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_unique_username ON users(username);

-- Составные индексы (порядок важен!)
CREATE INDEX idx_orders_user_status_date ON orders(user_id, status, created_at);

-- Индексы для сортировки
CREATE INDEX idx_products_price_desc ON products(price DESC);

-- Частичные индексы
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;
```

### Анализ использования индексов
```sql
-- Поиск неиспользуемых индексов
SELECT * FROM sys.schema_unused_indexes;

-- Статистика по индексам
SHOW INDEX FROM users;
ANALYZE TABLE users; -- Обновление статистики

-- Принудительное использование индекса
SELECT * FROM users FORCE INDEX (idx_users_email) 
WHERE email LIKE 'test%';
```

### Оптимизация больших таблиц
```sql
-- Партиционирование
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);
```

## 🔗 Оптимизация JOIN

### Эффективные JOIN стратегии
```sql
-- Неоптимально: фильтрация после JOIN
SELECT * 
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.country = 'USA' AND o.amount > 1000;

-- Оптимально: фильтрация до JOIN
SELECT * 
FROM orders o
JOIN (
    SELECT id FROM users WHERE country = 'USA'
) u ON o.user_id = u.id
WHERE o.amount > 1000;
```

### Оптимизация порядка JOIN
```sql
-- Правильный порядок: от маленькой к большой таблице
EXPLAIN 
SELECT * 
FROM small_table s
JOIN large_table l ON s.id = l.small_id;

-- Использование STRAIGHT_JOIN для принудительного порядка
SELECT STRAIGHT_JOIN *
FROM small_table s
JOIN large_table l ON s.id = l.small_id;
```

### Batch JOIN для больших данных
```sql
-- Пакетная обработка больших JOIN
CREATE TEMPORARY TABLE temp_user_ids AS
SELECT id FROM users WHERE last_login > '2024-01-01';

SELECT o.* 
FROM orders o
JOIN temp_user_ids t ON o.user_id = t.id
WHERE o.amount > 100;
```

## 📋 Оптимизация WHERE

### Эффективные условия
```sql
-- Не использует индекс
SELECT * FROM users WHERE YEAR(created_at) = 2024;
SELECT * FROM products WHERE price * 1.1 > 100;

-- Использует индекс
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
SELECT * FROM products WHERE price > 100 / 1.1;

-- LIKE оптимизация
SELECT * FROM users WHERE name LIKE 'john%'; -- Использует индекс
SELECT * FROM users WHERE name LIKE '%john%'; -- Не использует индекс
```

### Оптимизация IN и OR
```sql
-- Медленно для больших списков
SELECT * FROM users WHERE id IN (1, 2, 3, ..., 10000);

-- Быстрее: временная таблица или JOIN
CREATE TEMPORARY TABLE temp_ids (id INT PRIMARY KEY);
INSERT INTO temp_ids VALUES (1), (2), (3), ...;

SELECT u.* 
FROM users u
JOIN temp_ids t ON u.id = t.id;
```

## 📊 Оптимизация GROUP BY и ORDER BY

### Правильные индексы для агрегации
```sql
-- Индекс для группировки и сортировки
CREATE INDEX idx_orders_user_date_amount ON orders(user_id, order_date, amount);

-- Покрывающий индекс
EXPLAIN 
SELECT user_id, SUM(amount) 
FROM orders 
WHERE order_date >= '2024-01-01'
GROUP BY user_id;
```

### Оптимизация HAVING
```sql
-- Неоптимально: фильтрация после группировки
SELECT user_id, SUM(amount) as total
FROM orders
GROUP BY user_id
HAVING total > 1000;

-- Оптимально: фильтрация до группировки
SELECT user_id, SUM(amount) as total
FROM orders
WHERE amount > 0 -- Ранняя фильтрация
GROUP BY user_id
HAVING total > 1000;
```

## 🧩 Работа с подзапросами

### Замена подзапросов на JOIN
```sql
-- Медленно: коррелированный подзапрос
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.id AND o.amount > 1000
);

-- Быстро: JOIN
SELECT DISTINCT u.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.amount > 1000;
```

### Оптимизация с CTE
```sql
-- WITH вместо повторяющихся подзапросов
WITH user_stats AS (
    SELECT user_id, COUNT(*) as order_count
    FROM orders
    GROUP BY user_id
)
SELECT 
    u.*,
    COALESCE(us.order_count, 0) as order_count
FROM users u
LEFT JOIN user_stats us ON u.id = us.user_id;
```

## 📝 Временные таблицы

### Для сложных вычислений
```sql
-- Многоэтапная обработка
CREATE TEMPORARY TABLE temp_analysis AS
SELECT 
    user_id,
    COUNT(*) as order_count,
    SUM(amount) as total_amount
FROM orders
WHERE created_at >= '2024-01-01'
GROUP BY user_id;

-- Дальнейшая обработка
SELECT 
    u.name,
    ta.order_count,
    ta.total_amount
FROM users u
JOIN temp_analysis ta ON u.id = ta.user_id
WHERE ta.total_amount > 1000;
```

### Батч обработка
```sql
-- Пакетное обновление
CREATE TEMPORARY TABLE batch_update AS
SELECT id FROM large_table 
WHERE condition LIMIT 10000;

UPDATE large_table l
JOIN batch_update b ON l.id = b.id
SET l.status = 'processed';

DROP TEMPORARY TABLE batch_update;
```

## 📈 Мониторинг

### Настройка мониторинга
```sql
-- Включение slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- Performance Schema
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY sum_timer_wait DESC LIMIT 10;

-- Статистика по таблицам
SELECT * FROM sys.schema_table_statistics;
```

### Анализ медленных запросов
```sql
-- Просмотр slow log
SELECT * FROM mysql.slow_log 
ORDER BY query_time DESC 
LIMIT 10;

-- Поиск проблемных шаблонов
SELECT 
    digest_text,
    count_star,
    avg_timer_wait,
    max_timer_wait
FROM performance_schema.events_statements_summary_by_digest
ORDER BY max_timer_wait DESC
LIMIT 10;
```

## 🎯 Практические паттерны

### Пагинация с ключами
```sql
-- Традиционная пагиация (медленно на больших offset)
SELECT * FROM orders ORDER BY id LIMIT 10 OFFSET 10000;

-- Быстрая пагинация (по ключу)
SELECT * FROM orders 
WHERE id > 10000 
ORDER BY id 
LIMIT 10;
```

### Оптимизация COUNT
```sql
-- Медленно: COUNT(*)
SELECT COUNT(*) FROM large_table WHERE condition;

-- Быстрее: приблизительный подсчет
SELECT TABLE_ROWS 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_NAME = 'large_table';

-- Кеширование счетчика
CREATE TABLE counters (
    table_name VARCHAR(100) PRIMARY KEY,
    count_value BIGINT
);
```

### Оптимизация больших DELETE
```sql
-- Медленно и блокирует
DELETE FROM old_logs WHERE created_at < '2023-01-01';

-- Пакетное удаление
WHILE EXISTS (SELECT 1 FROM old_logs WHERE created_at < '2023-01-01') DO
    DELETE FROM old_logs 
    WHERE created_at < '2023-01-01' 
    LIMIT 1000;
    COMMIT;
    DO SLEEP(1); -- Пауза для уменьшения нагрузки
END WHILE;
```

### Оптимизация текстовых полей
```sql
-- Вынос больших текстов в отдельную таблицу
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    description TEXT -- Большое поле
);

CREATE TABLE product_details (
    product_id INT PRIMARY KEY,
    long_description LONGTEXT,
    specifications JSON,
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

## ⚡ Быстрые советы

1. **Всегда используйте EXPLAIN** перед оптимизацией
2. **Индексы решают 80% проблем** с производительностью
3. **Фильтруйте данные как можно раньше**
4. **Избегайте SELECT *** - выбирайте только нужные поля
5. **Используйте LIMIT** для exploratory queries
6. **Мониторьте slow query log** регулярно
7. **Тестируйте на реалистичных данных**

## 🔧 Полезные команды

```sql
-- Обновление статистики
ANALYZE TABLE table_name;
OPTIMIZE TABLE table_name;

-- Просмотр статуса
SHOW STATUS LIKE 'Handler%';
SHOW ENGINE INNODB STATUS;

-- Настройки буферов
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SET GLOBAL innodb_buffer_pool_size = 1024*1024*1024; -- 1GB
```

---

**Remember**: Оптимизация - это итеративный процесс. Всегда измеряйте результаты и тестируйте на production-подобных данных! 🚀
