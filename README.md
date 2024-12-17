# Отчет по базе данных энергопотребления

## Схема базы данных

### Таблица ПРЕДПРИЯТИЕ
- Поля:
  - ИДЕНТИФИКАТОР (тип: VARCHAR)
  - НАЗВАНИЕ (тип: VARCHAR)
  - ОБЛАСТЬ (тип: VARCHAR)
  - СКИДКА,% (тип: DECIMAL)

### Таблица ПОДСТАНЦИЯ
- Поля:
  - ИДЕНТИФИКАТОР (тип: VARCHAR)
  - НАЗВАНИЕ (тип: VARCHAR)
  - ОБЛАСТЬ (тип: VARCHAR)
  - ПОТЕРИ,% (тип: DECIMAL)

### Таблица ЭЛЕКТРОЭНЕРГИЯ
- Поля:
  - ИДЕНТИФИКАТОР (тип: VARCHAR)
  - ВРЕМЕННОЙ ИНТЕРВАЛ (тип: VARCHAR)
  - СТОИМОСТЬ 1KW, РУб (тип: NUMERIC)
  - ДОНОР (тип: VARCHAR)
  - ОБЩИЙ ЛИМИТ (тип: NUMERIC)

### Таблица ПОТРЕБЛЕНИЕ
- Поля:
  - УЧЕТНЫЙ НОМЕР (тип: VARCHAR)
  - ДАТА (тип: VARCHAR)
  - ПРЕДПРИЯТИЕ (внешний ключ к таблице ПРЕДПРИЯТИЕ)
  - ПОДСТАНЦИЯ (внешний ключ к таблице ПОДСТАНЦИЯ)
  - ВРЕМЯ (тип: VARCHAR)
  - РАСХОД (тип: NUMERIC)
  - К ОПЛАТЕ, РУБ (тип: NUMERIC)


## Обоснование выбора типов данных

### Критерии выбора типов данных:
1. Идентификаторы: `VARCHAR(10)` - достаточная длина для кодов
2. Названия: `VARCHAR(100)` - запас для полных наименований
3. Области: `VARCHAR(50)` - покрывает длину названий регионов
4. Числовые поля:
   - Скидки, потери: `DECIMAL(5,2)` с ограничением 0-100%
   - Стоимость, лимиты: `NUMERIC(10,2)` для точных финансовых расчетов

### Средства поддержания целостности:
- `PRIMARY KEY` для уникальности записей
- `FOREIGN KEY` для связей между таблицами
- `CHECK` ограничения для:
  - Процента скидки (0-100%)
  - Процента потерь (0-100%)
  - Стоимости за кВт (> 0)
- `NOT NULL` для обязательных полей

## Уровень 1

### Задание 1: Создание структуры таблиц
```sql
> 
CREATE TABLE enterprise (
    id VARCHAR(10) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    region VARCHAR(50) NOT NULL,
    discount DECIMAL(5,2) CHECK (discount >= 0 AND discount <= 100)
);

CREATE TABLE substation (
    id VARCHAR(10) PRIMARY KEY, 
    name VARCHAR(100) NOT NULL,
    region VARCHAR(50) NOT NULL,
    losses DECIMAL(5,2) CHECK (losses >= 0 AND losses <= 100)
);

CREATE TABLE electricity (
    id VARCHAR(10) PRIMARY KEY,
    time_interval VARCHAR(10) NOT NULL,
    cost_per_kw NUMERIC(10,2) NOT NULL CHECK (cost_per_kw > 0),
    donor VARCHAR(50) NOT NULL,
    total_limit NUMERIC(10,2) NOT NULL
);

CREATE TABLE consumption (
    account_number VARCHAR(10) PRIMARY KEY,
    date VARCHAR(20) NOT NULL,
    enterprise_id VARCHAR(10) REFERENCES enterprise(id),
    substation_id VARCHAR(10) REFERENCES substation(id),
    time VARCHAR(10) NOT NULL,
    consumption NUMERIC(10,2) NOT NULL,
    payment NUMERIC(10,2) NOT NULL,
    losses DECIMAL(5,2)
);
```
> 
![image-31](https://github.com/user-attachments/assets/d32dda21-9251-408a-b919-fdb7e24897c8)


### Задание 2: Ввод данных
```sql
INSERT INTO enterprise (id, name, region, discount) VALUES
('001', 'Авиационный завод', 'Нижегородская', 5.0),
('002', 'Приборостроительный завод', 'Владимирская', 0.0),
('003', 'ГАЗ', 'Нижегородская', 5.0),
('004', 'Невский завод', 'Ленинградская', 2.0),
('005', 'ЗИЛ', 'Московская', 2.0);

INSERT INTO substation (id, name, region, losses) VALUES
('001', 'Главная-Южная', 'Нижегородская', 2.0),
('002', 'N12', 'Московская', 2.0),
('003', 'Западная', 'Ленинградская', 2.0),
('004', 'Центральная', 'Московская', 1.0),
('005', 'N23', 'Владимирская', 2.0),
('006', 'Горьковская', 'Нижегородская', 2.0);

INSERT INTO electricity (id, time_interval, cost_per_kw, donor, total_limit) VALUES
('001', '0-3', 7000.00, 'Ленинградская', 600000.00),
('002', '4-6', 8000.00, 'Нижегородская', 580000.00),
('003', '7-8', 9000.00, 'Московская', 580000.00),
('004', '9-11', 10000.00, 'Свердловская', 550000.00),
('005', '12-14', 14000.00, 'Свердловская', 500000.00),
('006', '15-18', 10000.00, 'Владимирская', 700000.00),
('007', '21-24', 7000.00, 'Нижегородская', 720000.00);

INSERT INTO consumption (account_number, date, enterprise_id, substation_id, time, consumption, payment, losses) VALUES
('37111', 'Понедельник', '002', '002', '001', 200.00, 1400000.00, 2.0),
('37112', 'Понедельник', '003', '005', '001', 200.00, 1400000.00, 2.0),
('37113', 'Вторник', '001', '002', '005', 10.00, 140000.00, 2.0),
('37114', 'Вторник', '002', '005', '004', 250.00, 2500000.00, 2.0),
('37115', 'Вторник', '002', '005', '003', 200.00, 1800000.00, 2.0),
('37116', 'Вторник', '003', '005', '002', 150.00, 1200000.00, 2.0),
('37117', 'Вторник', '004', '003', '001', 200.00, 1400000.00, 2.0),
('37118', 'Вторник', '004', '004', '002', 100.00, 80000.00, 1.0),
('37119', 'Среда', '002', '003', '005', 300.00, 4200000.00, 2.0),
('37120', 'Среда', '002', '005', '007', 120.00, 84000.00, 2.0),
('37121', 'Среда', '003', '005', '001', 200.00, 140000.00, 2.0),
('37122', 'Среда', '005', '002', '006', 100.00, 1000000.00, 2.0),
('37123', 'Четверг', '005', '005', '004', 110.00, 1100000.00, 2.0),
('37124', 'Четверг', '005', '004', '003', 90.00, 810000.00, 2.0),
('37125', 'Пятница', '005', '004', '005', 300.00, 4200000.00, 1.0),
('37126', 'Пятница', '005', '001', '004', 100.00, 1000000.00, 2.0);
```
> 
![image-32](https://github.com/user-attachments/assets/7bd7d670-3b21-4142-90a2-f49498eb9ea3)


### Задание 3: Проверка данных
```sql
SELECT * FROM enterprise;
SELECT * FROM substation;
SELECT * FROM electricity;
SELECT * FROM consumption;
```
> 
![image-3](https://github.com/user-attachments/assets/a557b0de-a95b-4290-9267-7a3a0df184b5)


### Задание 4:
a) Различные дни потребления
```sql
SELECT DISTINCT date FROM consumption;
```
> 
![image-4](https://github.com/user-attachments/assets/b5f20802-327b-41f5-b050-1aebea37d461)


b) Названия предприятий
```sql
SELECT name FROM enterprise;
```
> 
![image-5](https://github.com/user-attachments/assets/434c0a59-e987-4c8c-94c8-4c88a811ea99)


c) Области подстанций
```sql
SELECT DISTINCT region FROM substation;
```
> 
![image](https://github.com/user-attachments/assets/b6db2d76-b3c3-40b3-8e9f-7fbd26175b11)


### Задание 5:
a) Подстанции Нижегородской/Московской области
```sql
SELECT * FROM substation 
WHERE region IN ('Нижегородская', 'Московская');
```
> 
![image-6](https://github.com/user-attachments/assets/ac787792-144b-4dd2-94c1-909995d9d522)


b) Временные интервалы с донором из Свердловской области
```sql
SELECT * FROM electricity 
WHERE donor = 'Свердловская' AND cost_per_kw > 10000;
```
> 
![image-7](https://github.com/user-attachments/assets/21607bdf-c085-4337-8748-c3a77f3cbf87)


c) Предприятия с "завод" в названии и скидкой > 1%
```sql
SELECT * FROM enterprise 
WHERE name LIKE '%завод%' AND discount > 1
ORDER BY name, region;
```
> 
![image-8](https://github.com/user-attachments/assets/a1d0a933-a8b4-4e46-8e48-376a51d11273)


## Задание 6:

### a) Учетный номер, название предприятия, сумма к оплате
```sql
SELECT 
    c.account_number, 
    e.name, 
    c.payment
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
ORDER BY c.payment;
```
> 
![image-9](https://github.com/user-attachments/assets/401d3bb5-d02d-4132-a95a-e2882eceaaf7)

> 

### b) Дата, название подстанции, расход электроэнергии
```sql
SELECT 
    c.date, 
    s.name, 
    c.consumption
FROM consumption c
JOIN substation s ON c.substation_id = s.id;
```
> 
![image-10](https://github.com/user-attachments/assets/daf7c4c6-c12c-4ec1-9513-124e2d7b823b)

> 

## Задание 7:

### a) Подстанции Московской области
```sql
SELECT 
    c.account_number, 
    e.name, 
    c.date
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
JOIN substation s ON c.substation_id = s.id
WHERE s.region = 'Московская';
```
> 
![image-11](https://github.com/user-attachments/assets/a6c6cc54-3a71-4738-986c-374d3949211e)

> 

### b) Подстанции с энергией для предприятий со скидкой ≥ 5%
```sql
SELECT 
    s.name, 
    s.region, 
    c.date
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
JOIN substation s ON c.substation_id = s.id
WHERE e.discount >= 5 AND c.date NOT IN ('Пятница')
ORDER BY c.date, s.name;
```
> 
![image-12](https://github.com/user-attachments/assets/ac6959e5-98f4-4287-8c57-65df60bb6c8f)

> 

### c) Доноры для промежутков, когда предприятиям Московской/Владимирской областей предоставляли эл.энергию подстанции с потерями > 1.5%
```sql
SELECT DISTINCT el.donor
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
JOIN substation s ON c.substation_id = s.id
JOIN electricity el ON el.id = (SELECT id FROM electricity LIMIT 1)
WHERE (e.region IN ('Московская', 'Владимирская')) 
AND s.losses > 1.5;
```
> 
![image-13](https://github.com/user-attachments/assets/986e8033-be42-4e9e-9dc9-22d49e7c1ffd)

> 

### d) Временные интервалы для потребителя "ГАЗ" с расходом > 100
```sql

SELECT 
    el.time_interval, 
    el.cost_per_kw, 
    el.donor
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
JOIN electricity el ON el.id = (SELECT id FROM electricity LIMIT 1)
WHERE e.name = 'ГАЗ' AND c.consumption > 100;
```
> 
![image-14](https://github.com/user-attachments/assets/890ab4b5-9593-41e9-9661-375c7f6e7bd5)

> 

## Задание 8: Модификация суммы оплаты

### Обновление сумм с учетом скидки
```sql
UPDATE consumption c
SET payment = c.payment * (1 - (e.discount / 100.0))
FROM enterprise e
WHERE c.enterprise_id = e.id;

SELECT 
    account_number, 
    payment AS original_payment, 
    payment AS discounted_payment
FROM consumption;
```
> 
![image-15](https://github.com/user-attachments/assets/65b19c80-0616-42c3-9bb1-a690c3ed1127)

> 

## Задание 9: Добавление столбца потерь(уже добавлен в начальной структуре)

## Уровень 2

### Задание 10: Операции IN (NOT IN)

#### a) Подстанции без энергии для Н.Новгорода
```sql
SELECT s.* 
FROM substation s
WHERE s.id NOT IN (
    SELECT DISTINCT c.substation_id 
    FROM consumption c
    JOIN enterprise e ON c.enterprise_id = e.id
    WHERE e.region = 'Нижегородская'
);
```
> 
![image-16](https://github.com/user-attachments/assets/d996308b-66e0-462d-8646-c034465cea21)

> 

#### b) Предприятия с потреблением через подстанции с низкими потерями
```SQL
SELECT DISTINCT e.* 
FROM enterprise e
JOIN consumption c ON e.id = c.enterprise_id
JOIN substation s ON c.substation_id = s.id
WHERE s.id IN (
    SELECT id 
    FROM substation 
    WHERE losses < 2
);
```
> 
![image-17](https://github.com/user-attachments/assets/c7daa41a-dbae-4b22-891c-425c11df938a)

> 

### Задание 11: Операции ALL-ANY

#### a) Временной интервал с максимальной стоимостью
```sql
SELECT time_interval 
FROM electricity 
WHERE cost_per_kw >= ALL (
    SELECT cost_per_kw 
    FROM electricity
);
```
> 
![image-18](https://github.com/user-attachments/assets/cbdf8b50-2a54-4aa3-8b8e-815dbc083d2c)

> 

#### b) Предприятие с минимальным потреблением
```sql
SELECT e.* 
FROM enterprise e
WHERE e.discount < (SELECT MAX(discount) FROM enterprise)
AND e.id IN (
    SELECT enterprise_id 
    FROM consumption 
    WHERE consumption = (
        SELECT MIN(consumption) 
        FROM consumption
    )
);
```

> 
#### c)
```sql
SELECT DISTINCT s.* 
FROM substation s
JOIN consumption c ON s.id = c.substation_id
WHERE c.date = 'Вторник'
AND EXISTS (
    SELECT 1 
    FROM electricity e 
    WHERE e.cost_per_kw >= ALL (
        SELECT cost_per_kw 
        FROM electricity
    )
);
```
![image-19](https://github.com/user-attachments/assets/0b7ec1bf-d3ef-4710-8ec0-7ae93dd04307)

> 

### Задание 12: UNION области
```sql
SELECT DISTINCT region FROM enterprise
UNION
SELECT DISTINCT region FROM substation
ORDER BY region;
```
![image-20](https://github.com/user-attachments/assets/7caf2940-e390-47e2-be26-9f5ff1d2d09e)

> 

> 

### Задание 13: Операции EXISTS (NOT EXISTS)

#### a) Предприятия и подстанции области
```sql
SELECT e.* 
FROM enterprise e
WHERE EXISTS (
    SELECT 1 
    FROM substation s
    WHERE s.region = e.region
    AND NOT EXISTS (
        SELECT 1 
        FROM consumption c
        WHERE c.enterprise_id = e.id 
        AND c.substation_id = s.id
    )
);
```
![image-21](https://github.com/user-attachments/assets/4229f4b9-7902-4699-9f47-4981b9981f10)


#### b) Интервалы без подстанций с потерями > 1%
```sql
SELECT DISTINCT c.time
FROM consumption c
WHERE NOT EXISTS (
    SELECT 1 
    FROM substation s
    WHERE s.losses > 1
    AND s.id = c.substation_id
);
```
![image-22](https://github.com/user-attachments/assets/bc530cec-fce1-4752-a369-37cd354f2083)

> 
#### c) Предприятия, не использующие подстанции своей области с потерями > 1%
```sql
SELECT e.* 
FROM enterprise e
WHERE NOT EXISTS (
    SELECT 1 
    FROM consumption c
    JOIN substation s ON c.substation_id = s.id
    WHERE c.enterprise_id = e.id 
    AND s.region = e.region 
    AND s.losses > 1
);


```
![image-23](https://github.com/user-attachments/assets/f09350b7-a0de-4f05-a004-5a794a5ce5dd)
> 

### Задание 14: Агрегатные функции

#### a) Количество предприятий, получавших энергию через подстанции Нижегородской области в 7-14 часов
```sql
SELECT COUNT(DISTINCT c.enterprise_id)
FROM consumption c
JOIN substation s ON c.substation_id = s.id
WHERE s.region = 'Нижегородская' 
AND c.time BETWEEN '7' AND '14';
```
> 
![image-24](https://github.com/user-attachments/assets/07946f9c-a00d-49f1-a771-51190b65f3e3)


#### b) Сумма к оплате для ГАЗа
```sql
SELECT SUM(payment) 
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
WHERE e.name = 'ГАЗ';
```
![image-25](https://github.com/user-attachments/assets/37b14cae-5aad-4ca9-bf34-df4d52cb1f20)


#### c) Подстанции с потерями больше средних
```sql
SELECT s.* 
FROM substation s
WHERE s.losses > (SELECT AVG(losses) FROM substation);
```

![image-26](https://github.com/user-attachments/assets/6b925e53-7c9e-4b76-baf3-52736e8fb110)


#### d) Количество подстанций для предприятий Московской области
```sql
SELECT COUNT(DISTINCT c.substation_id)
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
WHERE e.region = 'Московская';
```
![image-27](https://github.com/user-attachments/assets/d8a0399d-e21c-4e36-85b0-a6f320534388)

> 

### Задание 15:

#### a) Количество предприятий по временным интервалам
```sql
SELECT time, COUNT(DISTINCT enterprise_id) as enterprise_count
FROM consumption
GROUP BY time;
```
![image-28](https://github.com/user-attachments/assets/eb9e5de7-b7a6-433f-beeb-3c683c231d60)

> 

> 

#### b) Предприятия, заплатившие более 10000000
```sql
SELECT 
    e.name, 
    SUM(c.payment) as total_payment
FROM enterprise e
JOIN consumption c ON e.id = c.enterprise_id
GROUP BY e.name
HAVING SUM(c.payment) > 10000000;
```
#### Суммарное потребление для предприятий со скидкой > 2% по временным интервалам
```sql
SELECT 
    c.time, 
    e.name, 
    SUM(c.consumption) as total_consumption
FROM consumption c
JOIN enterprise e ON c.enterprise_id = e.id
WHERE e.discount > 2
GROUP BY c.time, e.name
ORDER BY total_consumption DESC;
```
![image-29](https://github.com/user-attachments/assets/c1a77502-d63d-429c-9bff-0c36c8eed50a)


#### c) Выдача электроэнергии для каждой подстанции по временным интервалам
```sql
SELECT 
    s.name as substation_name, 
    c.time, 
    SUM(c.consumption) as total_consumption
FROM consumption c
JOIN substation s ON c.substation_id = s.id
GROUP BY s.name, c.time
ORDER BY s.name, c.time;
```
![image-30](https://github.com/user-attachments/assets/51192360-b420-40cb-bc7d-372b18fbe265)

> 

> 

