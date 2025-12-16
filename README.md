**Лабораторные работы по БД**

Перечень [лабораторных работ](https://edu.irnok.net/doku.php?id=db:nn_tasks)

Telegram: [at]kilf3

# Постановка задачи (вариант 32)

**Учет контактов и взаимодействий (личная CRM)**

*Сущности:* Автомобили (гос. номер, марка, модель, тип двигателя, пробег), зоны парковки (адрес, количество мест), клиенты (ID, ФИО, водительский стаж).

*Процессы:*  Клиенты арендуют автомобили на определенные промежутки времени, поездки начинаются и заканчиваются в зонах парковки.

*Выходные документы:*
  - Выдать список самых популярных зон парковки для начала поездок за указанный период, отсортированный по количеству поездок.
  - Для заданного клиента выдать историю всех поездок с указанием стоимости и пробега, отсортированную по дате поездки.

# Лабораторная работа 1 (Проектирование логической и физической модели БД)

## Промпт к DeepSeek

```text
Лаба по проектированию информационной модели для реляционных баз данных.
Предполагаем Postgresql. 
Есть ошибки, замечания, неточности?

# Система каршеринга (аренды автомобилей по минутам)

## Постановка задачи

*Сущности:*
    Автомобили (гос_номер, марка, модель, тип_двигателя, пробег),
    Зоны_парковки (адрес, количество_мест),
    Клиенты (ID, ФИО, водительский_стаж).

*Процессы:*
    Клиенты арендуют автомобили на определенные промежутки времени,
    поездки начинаются и заканчиваются в зонах парковки.

*Выходные документы:*

  - Выдать список самых популярных зон парковки для начала поездок за указанный период, отсортированный по количеству поездок.

  - Для заданного клиента выдать историю всех поездок с указанием стоимости и пробега, отсортированную по дате поездки.

## ER-Модель

### Базовые сущности

    Автомобиль(гос_номер, марка, модель, тип_двигателя, пробег)
    ЗонаПарковки(адрес, количество_мест)
    Клиент(ФИО, водительский_стаж)

### Отношения

    [Клиент]-N,Optional------------------1,Required-[Поездка]-N,Required------------------1,Required-[Автомобиль]

    [ЗонаПарковки]-1,Optional------------------N,Required-[Поездка] (начало)
    [ЗонаПарковки]-1,Optional------------------N,Required-[Поездка] (окончание)

## Логическая модель

Получаем четыре таблицы:

  - ```Car(license_plate, brand, model, engine_type, mileage)```, primary key - license_plate
  - ```ParkingZone(address, spots_count)```, primary key - address
  - ```Client(client_id, full_name, driving_experience)```, primary key - client_id
  - ```Trip(trip_id, client_id, car_license_plate, start_zone_address, end_zone_address, start_time, end_time, cost, trip_mileage)```, primary key - trip_id

## Физическая модель

Зададим типы данных для атрибутов:

  - ```license_plate::varchar(10)```
  - ```brand::varchar(50)```
  - ```model::varchar(50)```
  - ```engine_type::varchar(20)```
  - ```mileage::integer```
  - ```address::varchar(200)```
  - ```spots_count::integer```
  - ```client_id::integer```
  - ```full_name::varchar(100)```
  - ```driving_experience::integer```
  - ```trip_id::integer```
  - ```start_zone_address::varchar(200)```
  - ```end_zone_address::varchar(200)```
  - ```start_time::timestamp```
  - ```end_time::timestamp```
  - ```cost::decimal(10,2)```
  - ```trip_mileage::integer```
```

## Ответ DeepSeek

Отличная основа! Модель логична и отражает основные бизнес-процессы, но есть несколько важных улучшений для создания более надежной базы данных.

### Критические замечания и рекомендации

#### 1. Выбор первичных ключей
**Проблема:** Использование естественных ключей (`license_plate`, `address`) как первичных — рискованно.
*   **Изменяемость:** Госномер автомобиля или адрес зоны парковки могут измениться.
*   **Сложность:** Длинные строковые ключи неэффективны для соединений и индексов.

**Исправление:** Используем суррогатные ключи:
```sql
Car(car_id, license_plate, brand, model, engine_type, mileage)
ParkingZone(zone_id, address, spots_count)
```

#### 2. Отслеживание пробега автомобиля
**Проблема:** Пробег в `Car` будет постоянно устаревать.
**Решение:** Хранить начальный пробег в `Car`, а общий пробег вычислять как сумму начального пробега и всех поездок.

#### 3. Валидация данных поездки
**Проблема:** Нет проверки, что время окончания позже времени начала.
**Решение:** Добавить CHECK-ограничение или валидацию на уровне приложения.

#### 4. Отсутствие индексов для частых запросов

### Исправленная и улучшенная модель

#### Логическая модель
*   **`Car`** (`car_id`, license_plate, brand, model, engine_type, initial_mileage)
*   **`ParkingZone`** (`zone_id`, address, spots_count)
*   **`Client`** (`client_id`, full_name, driving_experience)
*   **`Trip`** (`trip_id`, `client_id`, `car_id`, `start_zone_id`, `end_zone_id`, start_time, end_time, cost, trip_mileage)

#### Физическая модель (DDL для PostgreSQL)
```sql
CREATE TABLE Car (
    car_id SERIAL PRIMARY KEY,
    license_plate VARCHAR(10) NOT NULL UNIQUE,
    brand VARCHAR(50) NOT NULL,
    model VARCHAR(50) NOT NULL,
    engine_type VARCHAR(20) NOT NULL,
    initial_mileage INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE ParkingZone (
    zone_id SERIAL PRIMARY KEY,
    address VARCHAR(200) NOT NULL UNIQUE,
    spots_count INTEGER NOT NULL
);

CREATE TABLE Client (
    client_id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    driving_experience INTEGER NOT NULL CHECK (driving_experience >= 0)
);

CREATE TABLE Trip (
    trip_id SERIAL PRIMARY KEY,
    client_id INTEGER NOT NULL REFERENCES Client(client_id) ON DELETE RESTRICT,
    car_id INTEGER NOT NULL REFERENCES Car(car_id) ON DELETE RESTRICT,
    start_zone_id INTEGER NOT NULL REFERENCES ParkingZone(zone_id) ON DELETE RESTRICT,
    end_zone_id INTEGER NOT NULL REFERENCES ParkingZone(zone_id) ON DELETE RESTRICT,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    cost DECIMAL(10,2) NOT NULL CHECK (cost >= 0),
    trip_mileage INTEGER NOT NULL CHECK (trip_mileage >= 0),
    CHECK (end_time > start_time)
);

-- Индексы для ускорения часто используемых запросов
CREATE INDEX idx_trip_start_time ON Trip(start_time);
CREATE INDEX idx_trip_client_dates ON Trip(client_id, start_time);
CREATE INDEX idx_trip_start_zone_time ON Trip(start_zone_id, start_time);
```

#### Проверка нормальных форм
*   **1NF:** Соблюдена - все значения атомарные
*   **2NF:** Соблюдена - нет зависимостей неключевых атрибутов от части ключа
*   **3NF/BCNF:** Соблюдена - нет транзитивных зависимостей

---

### Примеры запросов для "Выходных документов"

**1. Самые популярные зоны парковки для начала поездок за период:**
```sql
SELECT 
    pz.address,
    COUNT(t.trip_id) as trip_count
FROM Trip t
JOIN ParkingZone pz ON t.start_zone_id = pz.zone_id
WHERE t.start_time BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY pz.zone_id, pz.address
ORDER BY trip_count DESC;
```

**2. История поездок для заданного клиента:**
```sql
SELECT 
    t.start_time,
    t.end_time,
    c.license_plate,
    c.brand,
    c.model,
    start_zone.address as start_address,
    end_zone.address as end_address,
    t.trip_mileage,
    t.cost
FROM Trip t
JOIN Car c ON t.car_id = c.car_id
JOIN ParkingZone start_zone ON t.start_zone_id = start_zone.zone_id
JOIN ParkingZone end_zone ON t.end_zone_id = end_zone.zone_id
WHERE t.client_id = 1  -- Подставьте нужный client_id
ORDER BY t.start_time DESC;
```

**Дополнительный полезный запрос - общий пробег автомобиля:**
```sql
SELECT 
    car_id,
    license_plate,
    initial_mileage + COALESCE(SUM(trip_mileage), 0) as total_mileage
FROM Car
LEFT JOIN Trip USING (car_id)
GROUP BY car_id, license_plate, initial_mileage;
```

## Полученные диаграммы:

### ER-диаграмма

```mermaid
erDiagram
    CLIENT ||--o{ TRIP : makes
    CAR ||--o{ TRIP : used_in
    PARKING_ZONE ||--o{ TRIP : starts_at
    PARKING_ZONE ||--o{ TRIP : ends_at
    
    CLIENT {
        integer client_id PK "SERIAL"
        varchar full_name
        integer driving_experience
    }
    
    CAR {
        integer car_id PK "SERIAL"
        varchar license_plate
        varchar brand
        varchar model
        varchar engine_type
        integer initial_mileage
    }
    
    PARKING_ZONE {
        integer zone_id PK "SERIAL"
        varchar address
        integer spots_count
    }
    
    TRIP {
        integer trip_id PK "SERIAL"
        integer client_id FK
        integer car_id FK
        integer start_zone_id FK
        integer end_zone_id FK
        timestamp start_time
        timestamp end_time
        decimal cost
        integer trip_mileage
    }
```

## Логическая модель в виде Диаграммы классов UML

```mermaid
classDiagram
    class Client {
        +client_id: Integer (PK)
        +full_name: String
        +driving_experience: Integer
        +getTrips() List~Trip~
    }
    
    class Car {
        +car_id: Integer (PK)
        +license_plate: String
        +brand: String
        +model: String
        +engine_type: String
        +initial_mileage: Integer
        +getTrips() List~Trip~
        +getTotalMileage() Integer
    }
    
    class ParkingZone {
        +zone_id: Integer (PK)
        +address: String
        +spots_count: Integer
        +getStartingTrips() List~Trip~
        +getEndingTrips() List~Trip~
    }
    
    class Trip {
        +trip_id: Integer (PK)
        +client_id: Integer (FK)
        +car_id: Integer (FK)
        +start_zone_id: Integer (FK)
        +end_zone_id: Integer (FK)
        +start_time: DateTime
        +end_time: DateTime
        +cost: Decimal
        +trip_mileage: Integer
        +getClient() Client
        +getCar() Car
        +getStartZone() ParkingZone
        +getEndZone() ParkingZone
    }
    
    Client "1" -- "*" Trip : совершает
    Car "1" -- "*" Trip : используется_в
    ParkingZone "1" -- "*" Trip : начинается_в
    ParkingZone "1" -- "*" Trip : заканчивается_в
```

## Физическая модель БД

```mermaid
erDiagram
    client {
        integer client_id PK "SERIAL"
        varchar full_name "NOT NULL"
        integer driving_experience "NOT NULL, CHECK (>=0)"
    }
    
    car {
        integer car_id PK "SERIAL"
        varchar license_plate "NOT NULL UNIQUE"
        varchar brand "NOT NULL"
        varchar model "NOT NULL"
        varchar engine_type "NOT NULL"
        integer initial_mileage "NOT NULL DEFAULT 0"
    }
    
    parking_zone {
        integer zone_id PK "SERIAL"
        varchar address "NOT NULL UNIQUE"
        integer spots_count "NOT NULL"
    }
    
    trip {
        integer trip_id PK "SERIAL"
        integer client_id FK "NOT NULL"
        integer car_id FK "NOT NULL"
        integer start_zone_id FK "NOT NULL"
        integer end_zone_id FK "NOT NULL"
        timestamp start_time "NOT NULL"
        timestamp end_time "NOT NULL"
        decimal cost "NOT NULL, CHECK (>=0)"
        integer trip_mileage "NOT NULL, CHECK (>=0)"
    }
    
    client ||--o{ trip : "FOREIGN KEY (client_id) REFERENCES client(client_id)"
    car ||--o{ trip : "FOREIGN KEY (car_id) REFERENCES car(car_id)"
    parking_zone ||--o{ trip : "FOREIGN KEY (start_zone_id) REFERENCES parking_zone(zone_id)"
    parking_zone ||--o{ trip : "FOREIGN KEY (end_zone_id) REFERENCES parking_zone(zone_id)"
```

## Заключение

В лабораторной работе произведено проектирование ER-, логической и физической модели базы данных для системы каршеринга (вариант 32). Основные улучшения, внесенные в ходе проектирования:

1. **Суррогатные ключи** - заменены естественные ключи (госномер, адрес) на суррогатные идентификаторы для стабильности структуры
2. **Управление пробегом** - разделен начальный пробег автомобиля и пробег поездок для точного отслеживания
3. **Валидация данных** - добавлены CHECK-ограничения для стажа вождения, стоимости, пробега и временных интервалов
4. **Оптимизация производительности** - созданы индексы для частых запросов по датам и клиентам
5. **Нормализация** - модель соответствует требованиям 3NF/BCNF

Модель эффективно поддерживает все требуемые бизнес-процессы и выходные документы, обеспечивая надежное хранение данных и высокую производительность запросов.


# Лабораторная работа 2
## Создание DDL-запросов для PostgreSQL
### Создана структура БД в pgAdmin

### Таблица car 
<img width="974" height="457" alt="image" src="https://github.com/user-attachments/assets/e101b0df-108a-4541-ba9a-3be7ffe9b4af" />

### Таблица client
<img width="974" height="330" alt="image" src="https://github.com/user-attachments/assets/3758dca3-2cbc-413f-9472-f85d3ae03f88" />

### Таблица parkingzone
<img width="974" height="334" alt="image" src="https://github.com/user-attachments/assets/83c9704d-ea88-4208-8a7f-52a59ccda07c" />

### Таблица trip
<img width="974" height="778" alt="image" src="https://github.com/user-attachments/assets/89b5daee-21b8-4e48-a188-71d7f2dd8e39" />
<img width="974" height="872" alt="image" src="https://github.com/user-attachments/assets/9d6fb9c4-e3f1-4795-8c23-d1d3317604f8" />

## Таблицы заполнены данными
### Таблица car 
<img width="974" height="230" alt="image" src="https://github.com/user-attachments/assets/5b417b73-58f8-49dc-bdd6-b59a8004d8e4" />
<img width="974" height="262" alt="image" src="https://github.com/user-attachments/assets/47e5118f-b0e5-4ff0-b965-eeb74099efb3" />

### Таблица client
<img width="974" height="233" alt="image" src="https://github.com/user-attachments/assets/116f7aca-97bf-4efa-a7a0-a3482736c6a2" />
<img width="827" height="433" alt="image" src="https://github.com/user-attachments/assets/8d8f1975-7193-4a3b-a074-6e1244f35f94" />

### Таблица parkingzone
<img width="913" height="245" alt="image" src="https://github.com/user-attachments/assets/e672bdbb-be7c-4973-9ad0-b54c6c53bf15" />
<img width="734" height="442" alt="image" src="https://github.com/user-attachments/assets/0f72eff2-ec2b-4bde-88a4-7c287d961dd5" />

### Таблица trip
<img width="974" height="161" alt="image" src="https://github.com/user-attachments/assets/bd269ab9-f470-49ea-85f2-918fbcd1dfdc" />
<img width="974" height="328" alt="image" src="https://github.com/user-attachments/assets/7f4edbd2-3de1-452c-b224-5aff6a55cf4f" />


## Выполнены SELECT-запросы c JOIN
### Запрос 1.
**Выдать список самых популярных зон парковки .**
<img width="507" height="464" alt="image" src="https://github.com/user-attachments/assets/807a58d4-3cce-4185-b2f7-0ab3777e9823" />

### Запрос 2. 
**История поездок клиента.**.
<img width="1219" height="535" alt="image" src="https://github.com/user-attachments/assets/173914f0-69ce-4dba-a2a4-7c2fe7821447" />



# Лабораторная работа 3
## Представления и процедуры
### Создано VIEW для запроса
**Выдать список самых популярных зон парковки .** 
<img width="600" height="563" alt="image" src="https://github.com/user-attachments/assets/fe0152d5-5191-43e0-a7b0-755d74ad6765" />
<img width="899" height="296" alt="image" src="https://github.com/user-attachments/assets/43a3976e-a2c7-46c7-a6e0-0d4af9955b0d" />
**История поездок клиента.**.
<img width="581" height="696" alt="image" src="https://github.com/user-attachments/assets/52bb0d12-2f7a-47d8-a123-0f2149d6385d" />
<img width="1400" height="237" alt="image" src="https://github.com/user-attachments/assets/f551b0ed-6fcb-4dde-a403-128ada694a68" />
**Статистика по автомобилям**
<img width="684" height="800" alt="image" src="https://github.com/user-attachments/assets/64054758-8acf-44ec-bdd0-27b122341a2b" />


### Создана процедур
**Процедура добавления нового автомобиля**
<img width="623" height="567" alt="image" src="https://github.com/user-attachments/assets/c41ff36a-0c39-409d-81a4-6ee584036e0c" />
<img width="712" height="230" alt="image" src="https://github.com/user-attachments/assets/995757b7-4eb7-4510-96de-5df92ebaf721" />

**Процедура добавления нового клиента**
<img width="533" height="421" alt="image" src="https://github.com/user-attachments/assets/69951d0b-62ee-4055-86eb-802b5a392b72" />
<img width="465" height="221" alt="image" src="https://github.com/user-attachments/assets/8c81198b-16e2-430a-8b2c-c18521e4d688" />

**Процедура добавления новой зоны парковки**
<img width="631" height="480" alt="image" src="https://github.com/user-attachments/assets/8ed2eb17-89d5-4c68-aea3-419a09491d21" />
<img width="446" height="229" alt="image" src="https://github.com/user-attachments/assets/c78caf90-0418-4bda-bdad-f64adfaeef69" />



# Лабораторная работа 4
## Анализ производительности
### Созданы генераторы данных, заполняющие таблицы 20000 записями
## Генератор для таблицы *car*
<img width="597" height="642" alt="image" src="https://github.com/user-attachments/assets/bcef6f25-53e7-4e4a-a0d4-8eccc545e11f" />
**Проверка работы представления**
<img width="421" height="134" alt="image" src="https://github.com/user-attachments/assets/1221c351-7125-40e5-879e-6881a5508d1a" />

## Генератор для таблицы *client*
<img width="694" height="573" alt="image" src="https://github.com/user-attachments/assets/8fc26e01-43c8-4fe6-96f3-3bdede22ada4" />
**Проверка работы представления**
<img width="411" height="130" alt="image" src="https://github.com/user-attachments/assets/97158cfc-3689-4c7c-9363-55008639ea57" />

## Генератор для таблицы *parking_zone*
<img width="822" height="531" alt="image" src="https://github.com/user-attachments/assets/25eb872a-f004-4448-9268-e92561f9d925" />
**Проверка работы представления**
<img width="486" height="122" alt="image" src="https://github.com/user-attachments/assets/7a1ce872-23f5-4a33-8cae-4f196d3d9f82" />

## Генератор для таблицы *trip*
<img width="639" height="686" alt="image" src="https://github.com/user-attachments/assets/f2fea794-c837-40a5-8da5-7ba0b59cdfa4" />
**Проверка работы представления**
<img width="406" height="106" alt="image" src="https://github.com/user-attachments/assets/ffecd143-4c2b-4690-add0-57f386162e83" />

## Проведем анализ плана выполнения основного запроса
**Самые популярные зоны парковки для начала поездок за 2024 год**
<img width="718" height="622" alt="image" src="https://github.com/user-attachments/assets/1a83d57c-fc36-4ad0-9b09-ba8eec9334a1" />
### Происходит полное сканирование таблицы trip (Seq Scan), что неэффективно при большом объеме данных.
## Оптимизация БД при помощи индексов
<img width="754" height="192" alt="image" src="https://github.com/user-attachments/assets/4d32f2db-a223-4d3b-8a78-bbaa0473d7d7" />
## Обновим статистику и посмотрим, что изменилось
<img width="716" height="660" alt="image" src="https://github.com/user-attachments/assets/7548ac73-ca3e-42dc-95fb-2fc6b31c330a" />
## Вывод
**Эффективность индексов: Создание составных индексов (start_zone_id, start_time и client_id, start_time) позволило ускорить выполнение основных запросов в 15-18 раз.**
**Выбор типа индекса: Для временных диапазонов лучше всего подходит B-дерево, которое эффективно для операторов BETWEEN, >, <.**
**Статистика: Регулярное обновление статистики (ANALYZE) помогает планировщику запросов выбирать оптимальные планы выполнения.**
**Специфика схемы piskunova: Все оптимизации применены с указанием схемы piskunova, что обеспечивает корректную работу в вашей структуре БД.**
**Индексы уже существуют: Согласно структуре таблицы trip, некоторые индексы (idx_trip_client_dates, idx_trip_start_zone_time, idx_trip_start_time) уже созданы. Процедуры генерации используют CREATE INDEX IF NOT EXISTS для предотвращения ошибок.**
**Производительность: Оптимизация базы данных системы каршеринга успешно выполнена, производительность ключевых запросов соответствует требованиям высокой нагрузки.**

# Лабораторная работа 5
## Реализация бизнес-логики на уровне БД и системы аудита для системы каршеринга
## Создадим триггер каскадного удаления для связей "один-ко-многим"
### При удалении автомобиля, будем удалять все связанные поездки
## Создадим триггерную функцию delete_car_trips
<img width="681" height="186" alt="image" src="https://github.com/user-attachments/assets/a27fce5c-1b27-4ac3-9718-98b0865d4864" />
## Создадим триггер на таблице car
## Проверка работы триггера
<img width="516" height="433" alt="image" src="https://github.com/user-attachments/assets/adc56ac8-2d61-4d6b-902e-4c172430be87" />
## Создадим аналогичный триггер для клиентов
<img width="671" height="280" alt="image" src="https://github.com/user-attachments/assets/bb6df04d-1369-4452-b85d-fda1b9509fd8" />

## Создадим таблицу-журнал для отслеживания изменений в таблице trip
<img width="625" height="377" alt="image" src="https://github.com/user-attachments/assets/5e6c1523-d090-4c4a-b246-dc9642ee5b34" />



