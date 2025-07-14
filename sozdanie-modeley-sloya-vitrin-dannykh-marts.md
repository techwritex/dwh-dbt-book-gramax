---
order: 10
title: Создание моделей слоя витрин данных (marts)
---

Структура папок слоя витрин данных повторяет структуру промежуточного слоя. Здесь создаются и хранятся как модели, ориентированные на конкретные бизнес-подразделения, так и модели, которые могут быть использованы несколькими подразделениями.

## Структура слоя

Создайте вложенные папки в  `marts`:

-  `core` – для хранения «общих» моделей,

-  `finance` – для хранения моделей финансового отдела.

## Модели слоя

### Общие модели

К этому моменту все необходимые преобразования  выполнены, поэтому остается лишь переложить итоговые модели промежуточного слоя в модели витрин.

Слой витрин данных для учебного проекта будет содержать четыре модели:

-  заказчики (customers),

-  автомобили (cars),

-  календарь (calendar)

-  оплата  (payments).

Начните с заказчиков.

Создайте в папке `models/marts/core/` файл `dim_customers.sql` и добавьте в него следующий код:

```sql
with customers as (

    select

        customer_id, 
        full_name, 
        gender, 
        driving_licence_number, 
        driving_licence_valid_from, 
        phone, 
        email, 
        updated_at

    from {{ ref('int_customers_name_concatenated') }}

)

select * from customers
order by customer_id
```

![](./sozdanie-modeley-sloya-vitrin-dannykh-marts.png "Рисунок №. Витрина Заказчики"){width=1656px height=1066px}

В той же папке создайте файл `dim_cars.sql` и добавьте в него следующий код:

```sql
with cars as (

    select

        car_id, 
        brand, 
        model, 
        category_text, 
        rate, 
        car_year, 
        vin, 
        licence_plate, 
        mileage

    from {{ ref('int_cars_joined_to_categories') }}

)

select * from cars
order by car_id
```

![](./sozdanie-modeley-sloya-vitrin-dannykh-marts-2.png "Рисунок №. Витрина Автомобили"){width=1684px height=1062px}

Создайте последнее измерение `dim_calendar.sql` и добавьте в него следующий код:

```sql
with calendar as (

	select

		date_id,
		full_date,
		year,
		month,
		day

	from  {{ ref('int_dates_package_generated') }}

)

select * from calendar
```

![](./sozdanie-modeley-sloya-vitrin-dannykh-marts-3.png "Рисунок №. Витрина Календарь"){width=1642px height=1054px}

### Модели финансового отдела

Осталось добавить последнюю модель с основной мерой (показателем) для финансового отдела.

Создайте в папке `models/marts/finance/` файл `fct_payments.sql` и добавьте в него следующий код:

```sql
with payments as (

    select 

        payment_id, 
        created_at, 
        customer_id,
        car_id, 
        amount

    from {{ ref('int_payments_joined_to_bookings') }}

)

select * from payments
```

![](./sozdanie-modeley-sloya-vitrin-dannykh-marts-4.png "Рисунок №. Витрина Оплата"){width=1694px height=1096px}

## Запуск проекта и обновление хранилища

Поздравляю! Вы построили хранилище данных в парадигме Кимбалла с помощью dbt!

Теперь данные можно анализировать, например, с помощью различных BI-инструментов (или при желании через sql-запросы). Но это отдельная тема, а сейчас время запустить движок dbt для сохранения новых объектов в хранилище.

Выполните команду запуска проекта:

```bash
dbt run
```

Перейдите в pgAdmin и проверьте результат:

<image src="./sozdanie-modeley-sloya-vitrin-dannykh-marts-5.png" title="Рисунок №. Создание схемы и объектов витрин" crop="0,0,100,100" objects="square,10.5262,17.7758,39.7854,4.1156,,top-left&square,10.7046,64.7986,38.3581,17.8634,,top-left" width="1390px" height="1416px"/>

Также для обновления линейного графа (и всей документации конечно же) выполните последовательно две команды:

```bash
dbt docs generate
dbt docs serve
```

После запуска локального сайта с документацией откройте линейный граф и убедитесь, что там добавлены новые модели витрин.

<image src="./sozdanie-modeley-sloya-vitrin-dannykh-marts-6.png" title="Рисунок №. Обновленный граф зависимости моделей проекта" crop="0,0,100,100" objects="square,88.4683,16.0511,10.3873,65.1989,,top-left" width="1136px" height="704px"/>

Остановите локальный сервер (`Ctrl+C`) и сохраните код текущего состояния проекта в git-репозитории.

## Сохранение проекта в Github

Загрузите текущее состояние вашего dbt-проекта в git-репозиторий.

Добавьте все файлы проекта:

```bash
git add .
```

Добавьте сообщение для коммита:

```bash
git commit -m "marts-models"
```

Отправьте локальный проект в репозиторий Github:

```bash
git push
```

Теперь актуальный код проекта хранится в Github-репозитории.