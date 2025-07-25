---
order: 8
title: Staging-слой хранилища
---

Теперь, когда определены источники, можете наконец-то начать использовать функциональность dbt для преобразования данных.

Первым шагом в процессе преобразования будет создание моделей staging-слоя.

## Модели слоя

Как ранее было обозначено, одно из основных преимуществ dbt автоматизация повторного использования фрагментов кода с помощью шаблонизатора Jinja.

<note type="lab" title="Примечание">

Одна из наиболее важных функций dbt (реализованных с помощью Jinja) – `ref()`, которая позволяет связывать модели между собой. Есть еще одна функция, которая тоже выполняет подобную задачу, но используется для привязки источников данных к моделям – `source()`.

</note>

Staging-модели, как и модели других слоев хранилища, создаются с помощью обобщенных табличных выражений или common table expressions (CTE). В общем виде модель первичного слоя выглядит следующим образом:

```sql
with source as (
    
    select * from {{ source('имя источника','таблица источника') }}

),

staged as (

    select
    
        (перечень полей таблицы источника)
    
    from source

)

select * from staged
```

Создайте вашу первую модель для хранения данных о заказчиках.

В папке  `models/staging/pg` создайте файл `stg_pg__customers.sql`, добавьте следующий код и сохраните модель:

```PostgreSQL
with source as (
    
    select * from {{ source('pg','customer') }}

),

staged as (

    select
    
        customer_id,
        first_name,
        last_name,
        gender,
        driving_licence_number,
        driving_licence_valid_from,
        phone,
        email,
        last_update as updated_at   
    
    from source

)

select * from staged
```

Создайте по аналогии модели по всем таблицам системы-источника, указанным в настроечном файле `src_pg__carsharing.yml`.

![](./sozdanie-modeley-pervichnogo-sloya-staging.png "Рисунок 19. Модели staging-слоя"){width=1528px height=932px}

## Запуск проекта и обновление хранилища

В предыдущем разделе вы создали первые модели вашего dbt-проекта. Но пока это всего лишь набор файлов, организованных в папки. Если сейчас вы проверите базу данных PostgreSQL, то никаких изменений там не встретите.

Для того, чтобы изменения появились, необходимо запустить dbt-проект.

Вернитесь в VS Code и перейдите в панель с терминалом (или же запустите командную строку, предварительно переместившись в папку dbt-проекта, если вы оттуда вышли).

Выполните команду запуска проекта:

```bash
dbt run
```

После запуска команды вы сразу увидите лог со следующей информацией:

-  версия dbt;

-  используемый адаптер к системе-источнику;

-  предупреждение (warning) о том, что в настройках проекта есть два слоя хранилища, которые не содержат никаких моделей;

-  количество найденных объектов;

-  статусы создания моделей;

-  статус запуска (создания) проекта.

![](./sozdanie-modeley-pervichnogo-sloya-staging-2.png "Рисунок 20. Результат запуск проекта (создание staging-моделей)"){width=1234px height=941px}

Проверьте полученный результат в базе. В соответствие с настройками создана новая схема `staging` и 12 представлений (views):

<image src="./sozdanie-modeley-pervichnogo-sloya-staging-3.png" title="Рисунок 21. Создание схемы и представлений первичного слоя" crop="0,0,100,100" objects="square,6.3141,28.6219,46.5665,3.5336,,top-left&square,7.6295,68.9046,45.5142,31.0954,,top-left" width="685px" height="1020px"/>

Также обратите внимание, что после выполнения команды `dbt run` в структуре проекта (в среде ведения разработки) создается папка `target` с двумя подпапками – `compiled` и `run`, а также некоторыми другим файлами.

<image src="./sozdanie-modeley-pervichnogo-sloya-staging-4.png" title="Рисунок 22. Изменение структуры проекта после сборки (dbt run)" crop="0,0,100,100" objects="square,7.9452,41.4258,26.9178,39.21,,top-left" width="1460px" height="1038px"/>

Папка `target/compiled/` содержит скомпилированный код моделей, состоящих только из `select` выражений. Здесь в код подставляются имена базы, схемы и таблицы источника.

В папке `target/run/` содержатся файлы, код которых будет выполняться в базе (платформе данных) и создавать соответствующие объекты. В этих файлах содержатся `create` выражения.

Посмотрите разницу содержимого файлов в этих двух папках на примере модели `stg_pg__customers`.

`target/compiled/carsharing/models/staging/pg/stg_pg__customers.sql`

```sql
with source as (
    
    select * from "carsharing_db"."public"."customer"

),

staged as (

    select
    
        customer_id,
        first_name,
        last_name,
        gender,
        driving_licence_number,
        driving_licence_valid_from,
        phone,
        email,
        last_update as updated_at   
    
    from source

)

select * from staged
```

`target/run/carsharing/models/staging/pg/stg_pg__customers.sql`

```sql
create view "carsharing_db"."staging"."stg_pg__customers__dbt_tmp"
    
    
  as (
    with source as (
    
    select * from "carsharing_db"."public"."customer"

),

staged as (

    select
    
        customer_id,
        first_name,
        last_name,
        gender,
        driving_licence_number,
        driving_licence_valid_from,
        phone,
        email,
        last_update as updated_at   
    
    from source

)

select * from staged
  );
```

Также после запуска проекта обновится журнал `logs/dbt.log`.

## Сохранение проекта в GitHub

Итак, с помощью dbt вы создали объекты staging-слоя в вашей платформе данных. Но сам код до сих пор хранится только на вашей локальной машине. Загрузите текущее состояние dbt-проекта в GitHub-репозиторий.

Добавьте все файлы проекта:

```bash
git add .
```

Добавьте сообщение для коммита:

```bash
git commit -m "staging-models"
```

Отправьте локальный проект в репозиторий GitHub:

```bash
git push
```

Теперь актуальный код проекта хранится в GitHub-репозитории и может использоваться любым разработчиком, который имеет доступ к этому репозиторию.