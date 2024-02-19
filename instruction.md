---
title: Использование данных из Postgres в стороннем ПО
date: 2024-02-12
name: Vladimir Shchepin
url: https://shchepin85.com
---

## Введение

На текущий момент ПО `%software_name` реализовано с использованием СУБД Postgres. В связи с запросами от клиентов, а так же в связи с внутренней необходимостью, может потребоваться использование данных из Postgres в стороннем ПО.

В инструкции описаны следующие сценарии:

* Доступ к данным из СУБД Oracle Database.
* Доступ к данным из Microsoft Office (Excel и Access).

## Установка и настройка ODBC-драйвера

### Что такое ODBC

ODBC - это API (см. [ODBC Reference](https://learn.microsoft.com/en-us/sql/odbc/reference/syntax/odbc-reference)), предназначенный для унифицированного доступа к различным источникам данных.

### Установка ODBC-драйвера

Для работы с Postgres рекомендуется установить драйвер psqlODBC. Существуют реализации драйвера для наиболее популярных ОС (операционных систем): Windows и Linux. С подробной документацией можно ознакомиться на сайте проекта [odbc.postgresql.org](https://odbc.postgresql.org/).

```{important}
Разрядность (битность) драйвера должна совпадать с разрядностью приложения, для которого необходимо настроить подключение к БД Postgres. К примеру, если сервер Oracle Database 64-битный, то и драйвер необходим 64-битный.
```

#### Установка psqlODBC в Windows

В соответствии с [FAQ](https://odbc.postgresql.org/faq.html#1.3) из официальной документации, необходимо воспользоваться MSI-установщиком. Скачать установщик можно с сайта [postgresql.org(https://www.postgresql.org/ftp/odbc/versions/msi/).

#### Установка psqlODBC в Linux

На официальном сайте проекта psqlODBC сборки драйвера для Linux не публикуются. Для установки драйвера необходимо воспользоваться штатными средствами ОС (пакетным менеджером). Примеры команд для наиболее популярных дистрибутивов:

* DEB-based (Debian / Ubuntu): `sudo apt install odbc-postgresql`
* RPM-based (Centos / Oracle / etc): `sudo yum install odbc-postgresql`

```{note}
Если в силу каких-то причин необходимо вручную собрать бинарный файл, исходные тексты можно скачать с сайта проекта по [ссылке](https://www.postgresql.org/ftp/odbc/versions/src/).
```

### Настройка ODBC-драйвера

#### Настройка psqlODBC в Windows

Настройка параметров конфигурации драйвера в Windows / Windows Server производится в ПУ (панели управления).

* Если разрядность psqlODBC-драйвера **совпадает** с разрядностью ОС, ПУ запускается стандартным для Windows способом: `Control Panel` -> `Administrative Tools` -> `Data Sources (ODBC)`.
* Если разрядность драйвера **не совпадает**  с разрядностью ОС, запуск ПУ необходимо произвести вручную. Исполняемый файл ПУ находится, как правило, здесь: `C:\Windows\system32\odbcad32.exe`.

В панели управления драйвера необходимо:

* Перейти во вкладку `System DSN`.
* Добавить новый источник данных для `Postgres (unicode)`.
* Указать параметры конфигурации, необходимые для подключения к БД.

#### Настройка psqlODBC в Linux

Настройка драйвера в Linux производится в файле `odbc.ini`. Как правило, файл конфигурации расположен в директории `/etc`. Для редактирования файла можно воспользоваться привычным привычным текстовым редактором, к примеру:

```{code}
sudo nano /etc/odbc.ini
```

Пример настроек в файле конфигурации:

```{code}
[ODBC Data Sources]
  Product = PostgreSQL
[Product]
  Debug = 1
  CommLog = 1
  ReadOnly = no
  Driver = /usr/pgsql-9.1/lib/psqlodbc.so
  Servername = <PostgreSQL_IP>
  FetchBufferSize = 99
  Username = <user>
  Password = <pass>
  Port = 5432
  Database = <db_name>
[Default]
  Driver = /usr/lib64/liboplodbcS.so.1
```

#### Проверка соединения с базой данных

Если возникли проблемы с работой драйвера, необходимо проверить:

* Версию и разрядность установленного psqlODBC-драйвера
* Настройки подключения к БД

Дополнительную информацию по настройке psqlODBC можно найти на официальном сайте проекта [odbc.postgresql.org](https://odbc.postgresql.org/) и в документации к выбранной операционной системе.

## Настройка в Oracle DB-линка к БД Postgres

Рекомендации в этом разделе предназначены для сотрудников, которые занимаются эксплуатацией системы, а так же для системных администраторов.

```{important}
Разрядность (битность) драйвера должна совпадать с разрядностью СУБД Oracle!
```

Для настройки DB-линка необходимо:

1. Скорректировать параметры в файлах конфигурации `init<Product>.ora`, `listener.ora` и `tnsnames.ora`.
1. Перезапустить процесс `Oracle Database Listener`.
1. Создать DB-линк.

Ниже приведены более подробные рекомендации по каждому из шагов.

### Настройка файлов конфигурации Oracle

#### `init<Product>.ora`

```{important}
**ВАЖНО! ЭТОТ РАЗДЕЛ НЕОБХОДИМО ДОРАБОТАТЬ!**

**Исходная информация, которая была перенесена в инструкцию, нуждается в проверке.**
```

Для редактирования файла необходимо: 

1. Зайти в директорию `C:\Oracle\product\<dbversion>\database\hs\admin\`, где `<dbversion>` - версия Oracle (к примеру, `11.2.0`).
2. Создать файл `init<Product>.ora`, где `<Product>` - имя добавленного в настройках `psqlODBC` источника данных.

Windows

```{code}
HS_FDS_CONNECT_INFO = Product
HS_FDS_TRACE_LEVEL = 0
HS_NLS_NCHAR = UCS2
HS_LANGUAGE = american_america.we8mswin1252
```

Последняя строка - не опечатка. Можно еще попробовать так:

```{code}
HS_LANGUAGE = american_america.al32utf8
```

Но у меня не заработало. Под unix скорее всего нужно будет добавить еще всяких разных строк:

```{code}
HS_FDS_CONNECT_INFO = MoodlePostgres
HS_FDS_SHAREABLE_NAME = /<path_to_postrges>/psqlodbc.so
HS_FDS_SUPPORT_STATISTICS = FALSE
HS_KEEP_REMOTE_COLUMN_SIZE = ALL
```

#### `listener.ora`

1. Зайти в директорию `C:\Oracle\product\<dbversion>\database\NETWORK\ADMIN\`, где `<dbversion>` - версия Oracle.
1. Открыть файл `listener.ora` для редактирования.
1. В секцию `SID_LIST_LISTENER/SID_LIST` добавить новую запись `SID_DESC/SID_NAME = <Product>`

Пример файла конфигурации:

```{code}
SID_LIST_LISTENER =
 (SID_LIST =
 (SID_DESC =
 (SID_NAME = CLRExtProc)
 (ORACLE_HOME = C:\oracle\product\11.2.0\database)
 (PROGRAM = extproc)
 (ENVS = "EXTPROC_DLLS=ONLY:C:\oracle\product\11.2.0\database\bin\oraclr11.dll")
 )
 (SID_DESC=
 (SID_NAME = Product)
 (ORACLE_HOME = C:\oracle\product\11.2.0\database)
 (PROGRAM = dg4odbc)
 )
 )
```

#### `tnsnames.ora`

1. Зайти в директорию `C:\Oracle\product\<dbversion>\database\NETWORK\ADMIN\`
1. Открыть файл `tnsnames.ora` для редактирования.
1. Добавить секцию с настройками подключения к Oracle DB. Указать имя источника данных вместо `Product` в примере ниже

Пример настроек в `tnsnames.ora`:

```{code}
Product =
 (DESCRIPTION = 
 (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
 (CONNECT_DATA = 
 (SID = Product))
 (HS = OK)
 )
```

### Перезапуск процесса `Oracle Database Listener`

`(TBD)`

### DB-линк к БД Postgres

#### Создание DB-линка

Ниже - пример SQL-скрипта, который создает DB-линк, где:

* `Product` - имя БД,
* `Product_scr` - пользователь БД,
* `password` - пароль к БД.

```{code} sql
create database link Product connect to "Product_scr" identified by "password" using 'Product';
```

#### Использование DB-линка

SQL-запросы к БД через DB-линк происходят почти стандартным образом. Однако, имя таблицы необходимо брать в кавычки:

```{code} sql
create database link Product connect to "Product_scr" identified by "password" using 'Product';
```

## Запрос данных из Postgres средствами Microsoft Office

Поскольку стандарт ODBC разрабатывался Microsoft, поддержка работы с источниками данных через ODBC-драйвер хорошо реализована во многих продуктах этой компании. К примеру, импортировать данные можно, используя MS Excel или MS Access.

### Импорт данных в MS Excel

#### Создание `.dqy`-файла

Для работы с сторонним источником данных  необходимо:

1. Создать пустой файл с расширением `.dqy` и произвольным названием.
1. Открыть созданный файл. Для редактирования файла можно использовать Блокнот, Notepad++, или любой редактор исходного кода.
1. В `.dqy`-файл необходимо добавить 4 строки, в соответствии с этим примером:

```{code-block} text
:linenos:

XLODBC
1
DRIVER={PostgreSQL Unicode};DATABASE=Product;SERVER=127.0.0.1;PORT=5432;UID=<user>;PASSWORD=<passwordd>;SSLmode=disable;ReadOnly=0;Protocol=7.4;FakeOidIndex=0;ShowOidColumn=0;RowVersioning=0;ShowSystemTables=0;ConnSettings=;Fetch=100;Socket=4096;UnknownSizes=0;MaxVarcharSize=255;MaxLongVarcharSize=8190;Debug=0;CommLog=0;Optimizer=0;Ksqo=1;UseDeclareFetch=0;TextAsLongVarchar=1;UnknownsAsLongVarchar=0;BoolsAsChar=1;Parse=0;CancelAsFreeStmt=0;ExtraSysTablePrefixes=dd_;LFConversion=1;UpdatableCursors=1;DisallowPremature=0;TrueIsMinus1=0;BI=0;ByteaAsLongVarBinary=0;UseServerSidePrepare=0;LowerCaseIdentifier=0;GssAuthUseGSS=0;XaOpt=1
select * from Product_rate_plans
```

Далее, требуется скорректировать параметры подключения к источнику данных:

* DATABASE=Product
* SERVER=127.0.0.1;
* UID=<user>
* PASSWORD=<password>

Последняя строка - это SQL запрос. Синтаксис запроса - стандартный.

```{important}
Важно!

* `.dqy`-файл должен состоять всего из 4 строк
* SQL-запрос должен занимать только одну строку
```

#### Импорт данных из Postgres

1. Созданный на предыдущем шаге `.dqy`-файл необходимо запустить (к примеру, двойным щелчком по файлу из Проводника Windows).
1. В интерфейсе Excel появится запрос на подключение к источнику данных - необходимо будет его одобрить.
1. После импорта с данными можно работать стандартными средствами Microsoft Excel.

### Импорт данных в MS Access

`[TBD]`

_В Access аналогично можно сделать импорт, выбрав в контекстном меню "Источники данных ODBC", причем можно загрузить сразу все таблицы. После этого для анализа данных можно использовать всю мощь и удобство движка Access._