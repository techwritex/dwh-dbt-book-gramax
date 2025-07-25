---
order: 7
title: "Определение источников "
---

Любой проект по созданию хранилища данных можно свести к двум основным задачам:

1. Поиск подходящих источников данных.

2. Преобразование данных для получения определенных результатов (удоволетворяющих бизнес).

Для реализации учебного проекта искать источники не потребуется. Как ранее было отмечено, предполагается, что транзакционные данные каршеринга уже загружены в хранилище (база PostgreSQL).

Но небольшую подготовку провести все-таки надо.

## **Загрузка данных в PostgreSQL**

<note title="Важно">

Содержимое базы данных проекта является вымыслом и не отражает реальные факты. Данные сформированы с помощью генераторов случайной информации. Любое сходство с реальными людьми и их персональными данными является случайным.

</note>

Скачайте по [бэкап](https://storage.yandexcloud.net/carsharing-dbt/backup/carsharing_db.sql) стартовой базы данных проекта (или же исходные данные для каждой таблицы в виде [CSV-файлов](https://storage.yandexcloud.net/carsharing-dbt/csv/)).

Откройте pgAdmin и загрузите скачанный файл в PostgreSQL.

В результате выполненных действий у вас появится база данных `carsharing_dbt`, которая состоит из 12 таблиц в схеме `public`.

Физическая схема транзакционных данных выглядит следующим образом:

![](./opredelenie-istochnikov-dannykh.png "Рисунок 15. ER-диаграмма транзакционных данных учебного проекта Carsharing"){width=2006px height=953px}

<note type="lab" title="Примечание">

Интерактивный вариант диаграммы можно посмотреть с помощью сервиса dbdiagram.io. <https://dbdiagram.io/d/carsharing-66475d4ef84ecd1d22768139>

</note>

## **Объявление источников данных**

Источники в dbt-проекте объявляются с помощью YAML-файла, который расположен в корне папки `staging`. При этом название для этого настроечного файла можно выбрать абсолютно любое.

<note type="lab" title="Примечание">

В продуктивном проекте может быть (и это вполне логично) несколько источников данных. Все эти источники можно указать как в одном настроечном файле на уровне папки `staging`, так и в виде отдельных файлов для каждой системы-источника, для которых создаются вложенные папки в `staging`. Второй вариант считается лучшей практикой.

</note>

Несмотря на то, что в текущем учебном проекте всего одна система-источник, создайте в `models` папку `staging` и вложенную в нее папку `pg`, в которой, в свою очередь, создайте новый файл `src_pg__carsharing.yml`.

![](./opredelenie-istochnikov-dannykh-2.png "Рисунок 16. Создание настроечного файла системы-источника"){width=1206px height=682px}

Добавьте следующие настройки в  `src_pg__carsharing.yml` для источников:

1. `name` – произвольное имя источника (например, pg)

2. `database` – имя базы, которое указано в настройке профилей `profiles.yml`

3. `schema` – имя схемы, которое указано в настройке профилей `profiles.yml`

4. `tables/name` – перечислите все таблицы, данные из которых импортируются в проект.

```bash
version: 2

sources:
  - name: pg
    database: carsharing_db
    schema: public
    tables:
      - name: customer
      - name: booking
      - name: payment
      - name: ride
      - name: location
      - name: car
      - name: category
      - name: service
      - name: service_type
      - name: breakdown
      - name: breakdown_type
      - name: accident
```

Источники определены, пришло время создать первые модели проекта. Но прежде сохраните текущее состояние проекта в репозитории GitHub.

## **Сохранение проекта в GitHub**

Загрузите текущее состояние dbt-проекта в репозиторий, который вы создали ранее (в главе «[Развертывание dbt-проекта](./razvertyvanie-dbt-proekta)»).

Для этого откройте в VS Code панель с терминалом или же запустите командную строку отдельно (предварительно переместитесь в папку dbt-проекта).

```bash
cd carsharing
```

Инициализируйте git-репозиторий в папке проекта:

```bash
git init
```

Добавьте все файлы проекта:

```bash
git add .
```

Добавьте сообщение для коммита:

```bash
git commit -m "first commit"
```

Переключитесь на основную ветку git:

```bash
git branch -M main
```

Вернитесь в GitHub и скопируйте ссылку на созданный ранее репозиторий - нажмите **Code**, а затем ссылку на вкладке **HTTPS**:

![](./razvertyvanie-dbt-proekta-10.png "Рисунок 17. Ссылка на репозиторий"){width=755px height=566px}

Вернитесь в терминал VS Code (или командную строку) и укажите скопированный репозиторий (замените на свой):

```bash
git remote add origin https://github.com/YOUR-ACCOUNT/carsharing-dbt.git
```

Отправьте локальный проект в репозиторий GitHub:

```bash
git push -u origin main
```

Перейдите в репозиторий на GitHub и убедитесь, что коммит успешно выполнен.

![](./opredelenie-istochnikov-dannykh-3.png "Рисунок 18. Выполнение первого коммита"){width=756px height=666px}

Отлично! Идем создавать модели.