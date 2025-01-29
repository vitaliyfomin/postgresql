## Создаем базу данных:
```
CREATE DATABASE skillbox_db;
```
## Подключаемся к базе данных:
```
\c skillbox_db
```
## Создаём таблицы skillbox_test с полями: «Идентификатор», «Логин», «Размер заработной платы», «Контактный номер телефона»:
```
CREATE TABLE skillbox_test (
    id SERIAL PRIMARY KEY,
    login VARCHAR(255) NOT NULL,
    salary INTEGER NOT NULL,
    phone_number VARCHAR(15) NOT NULL
);
```
## Создаем пользователей для сотрудников отдела сбыта:
```
CREATE USER semionov WITH PASSWORD 'password_1';
CREATE USER danilov WITH PASSWORD 'password_2';
CREATE USER romanova WITH PASSWORD 'password_3';
```
## Добавляем данные в тестовую таблицу:
```
INSERT INTO skillbox_test (login, salary, phone_number) VALUES
('SEMIONOV', 50000, '791101234567'),
('DANILOV', 90000, '795301234567'),
('ROMANOVA', 70000, '790401234567');
```
## Предоставляем права на таблицу для всех пользователей:
```
GRANT SELECT ON skillbox_test TO semionov, danilov, romanova;
```
## Включаем RLS (Row Level Security) и создаём политику:
```
CREATE POLICY sales_policy
 ALTER TABLE skillbox_test ENABLE ROW LEVEL SECURITY;
```
## Политика для SEMIONOV (видит только свои записи):
```
CREATE POLICY sales_policy_semionov
    ON skillbox_test
    FOR SELECT
    TO semionov
    USING (login = 'SEMIONOV');
```
## Политика для DANILOV (видит только свои записи):
```
CREATE POLICY sales_policy_danilov
    ON skillbox_test
    FOR SELECT
    TO danilov
    USING (login = 'DANILOV');
```
## Политика для ROMANOVA (видит все записи):
```
CREATE POLICY accounting_policy
    ON skillbox_test
    FOR SELECT
    TO romanova
    USING (true);
```
## Создаём представления с маскировкой данных:
```
CREATE VIEW skillbox_test_masked AS
SELECT
    id,
    login,
    salary,
    CASE
        WHEN current_user IN ('semionov', 'danilov') THEN
            CONCAT(SUBSTRING(phone_number, 1, 4), '****')  -- Маскируем номер телефона
        ELSE
            phone_number  -- Для бухгалтерии показываем настоящий номер телефона
    END AS phone_number
FROM skillbox_test;
```
## Предоставляем права на представление:
```
GRANT SELECT ON skillbox_test_masked TO semionov, danilov, romanova;
```
## Проверяем работу маскировки:
### Проверка для SEMIONOV:
```
SET ROLE semionov;
SELECT * FROM skillbox_test_masked;
```
### Проверка для DANILOV:
```
SET ROLE danilov;
SELECT * FROM skillbox_test_masked;
```
### Проверка для ROMANOVA:
```
SET ROLE romanova;
SELECT * FROM skillbox_test_masked;
```
# ИЛИ

## Удаляем представление skillbox_test_masked:
```
DROP VIEW IF EXISTS skillbox_test_masked;
```
## Создаем новый запрос с учетом RLS и маскировки:
### Запрос для SEMIONOV:
```
SET ROLE semionov;
SELECT
    id,
    login,
    salary,
    CASE
        WHEN current_user IN ('semionov', 'danilov') THEN
            CONCAT(SUBSTRING(phone_number, 1, 4), '****')  -- Маскируем номер телефона
        ELSE
            phone_number  -- Для бухгалтерии показываем настоящий номер телефона
    END AS phone_number
FROM skillbox_test
WHERE login = 'SEMIONOV';  -- Применяем политику RLS
```
### Запрос для DANILOV:
```
SET ROLE danilov;
SELECT
    id,
    login,
    salary,
    CASE
        WHEN current_user IN ('semionov', 'danilov') THEN
            CONCAT(SUBSTRING(phone_number, 1, 4), '****')  -- Маскируем номер телефона
        ELSE
            phone_number  -- Для бухгалтерии показываем настоящий номер телефона
    END AS phone_number
FROM skillbox_test
WHERE login = 'DANILOV';  -- Применяем политику RLS
```
### Запрос для ROMANOVA:
```
SET ROLE romanova;
SELECT
    id,
    login,
    salary,
    CASE
        WHEN current_user IN ('semionov', 'danilov') THEN
            CONCAT(SUBSTRING(phone_number, 1, 4), '****')  -- Маскируем номер телефона
        ELSE
            phone_number  -- Для бухгалтерии показываем настоящий номер телефона
    END AS phone_number
FROM skillbox_test;  -- ROMANOVA видит все записи
```
