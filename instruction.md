---
title: Использование данных из Postgres в стороннем ПО
date: 2024-02-12
name: Vladimir Shchepin
url: https://shchepin85.com
---

## Введение

На текущий момент ПО `%software_name` реализовано с использованием СУБД Postgres. В связи с запросами от клиентов, а так же в связи с внутренней необходимостью, может потребоваться использование данных из Postgres в стороннем ПО.

В инструкции описаны следующие сценарии:

* Доступ к данным или их миграция в СУБД Oracle Database
* Microsoft Office (Excel и Access)

## Использование ODBC для объединения источников данных

ODBC - это API (см. [ODBC Reference](https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/odbc-reference)), предназначенный для унифицированного доступа к различным источникам данных.

### Установка драйвера ODBC

Существуют реализации ODBC-драйвера для наиболее популярных ОС: Windows,  Linux и MacOS.

```{important}
Разрядность (битность) драйвера **должна** совпадать с разрядностью сервера Postgres.
```

#### MSI-установщик для Windows

Для установки драйвера в Windows или Windows Server необходимо ознакомиться с инструкцией с сайта [learn.microsoft.com](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server). Для установки драйвера требуется воспользоваться MSI-установщиком, в соответствии с инструкцией.


#### Установка пакета psqlODBC в Linux

Для работы с Postgres в Linux необходимо установить драйвер psqlODBC. С подробной документацией о драйвере можно ознакомиться на сайте [odbc.postgresql.org](https://odbc.postgresql.org/).

Для разных 

```code
sudo apt install 
```
