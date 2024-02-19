---
title: My First Article
date: 2024-02-12
name: Vladimir Shchepin
url: https://shchepin85.com
---

## Использование данных postgres в стороннем ПО

### Введение

На текущий момент ПО `%software_name` реализовано с использованием СУБД Postgres

В инструкции описаны методы миграции данных из Postgres в стороннее ПО:

* СУБД Oracle Database
* Microsoft Office (Excel и Access)

## Сырой текст

```{note}
_Исходные данные взяты из файла с тестовым заданием, перенесены в MD и слегка отформатированы._
```

### Использование данных postgres в стороннем ПО (oracle, excel...)

Штатным средством для объединения разнородных источников данных является ODBC, который есть в Windows со времен win95 и с некоторых времен даже в unix-системах.

Нужно найти и установить драйвер `psqlodbc` (например с сайта [postgresql.org](http://www.postgresql.org/ftp/odbc)), при этом крайне важно выбрать правильную разрядность. Если приложение, которому нужен доступ, 32-битное, а драйвер 64-битный, будем получать `ERROR [IM014] [Microsoft][ODBC Driver Manager] The specified DSN contains an architecture mismatch`.

Под Windows драйвер лучше всего установить из msi-пакета. Если Windows 64-битная, а драйвер 32-битный, то панель управления нужно вызвать вручную:

```text
c:\windows\system32\odbcad32.exe
```

Во всех остальных случаях интерфейс настройки вызывается штатно через Control Panel -> Administrative Tools -> Data Sources (ODBC).

Там на вкладке System DSN нужно создать новый источник данных, выбрав драйвер Postgres (unicode) и прописав учетные данные.

Под unix необходимо знать особенности ОС + поможет поиск в интернет, общая идея - прописать в файл .odbc.ini настройки, примерно так:

```text Text
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

В общем, будем считать, что драйвер корректно установлен и работает. Если не работает, значит что-то сделано не так: нужно проверить версии, разрядность, настройки, нажать reset и потрясти бубном.

### 1. Как поднять database link с ora на postgres

Теперь рассмотрим случай с Oracle. Опять же, разрядность драйвера должна быть такая же, какая у базы оракла. В примере база данных (не клиент!) установлена под Windows в каталог c:\oracle\product\11.2.0\database

Идем в c:\oracle\product\11.2.0\database\hs\admin\ и в файле initProduct.ora (Product - имя созданного ранее источника, может быть любым, здесь и далее используется Product), который нужно создать вручную, пишем:

```text
HS_FDS_CONNECT_INFO = Product
HS_FDS_TRACE_LEVEL = 0
HS_NLS_NCHAR = UCS2
HS_LANGUAGE = american_america.we8mswin1252
```

Последняя строка - не опечатка. Можно еще попробовать так:

```text
HS_LANGUAGE = american_america.al32utf8
```

Но у меня не заработало. Под unix скорее всего нужно будет добавить еще всяких разных строк:

```text
HS_FDS_CONNECT_INFO = MoodlePostgres
HS_FDS_SHAREABLE_NAME = /<path_to_postrges>/psqlodbc.so
HS_FDS_SUPPORT_STATISTICS = FALSE
HS_KEEP_REMOTE_COLUMN_SIZE = ALL
```

Далее идем в `c:\oracle\product\11.2.0\database\NETWORK\ADMIN\` и правим файл `listener.ora`, а именно нужно добавить запись в секцию `SID_LIST_LISTENER/SID_LIST`, должно получиться примерно так (первая запись SID_DESC здесь уже была, добавлялись строки для `SID_NAME = Product`):

```text
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

И наконец нужно добавить описание в tnsnames.ora в этом же каталоге (сервер `Product` нужно подставить свой, а порт - стандартный порт Oracle, так и должно быть, это важно):

```text
Product =
 (DESCRIPTION = 
 (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
 (CONNECT_DATA = 
 (SID = Product))
 (HS = OK)
 )
```

Теперь нужно перезапустить Oracle Database Listener, и можно выполнить стандартное создание линка (`Product_scr` - пользователь владельца схемы базы `Product`, `password` - его пароль):

```sql
create database link Product connect to "Product_scr" identified by "password" using 'Product';
```

Получившийся линк можно использовать почти стандартным образом, примерно так:

```sql
select * from "Product_rate_plans"@Product;
```

Разница в том, что таблицу нужно брать в кавычки. В остальном разницы нет. Кроме того, что вопрос трансакционности остается открытым, т.к. распределенные трансакции воообще штука непростая, а в случае с нестандартными источниками данных совсем ничего не понятно. Однако на чтение все должно нормально работать (забавно, что Oracle открывает трансакцию для SELECT точно так же, как он это делает и для своих родных линков). Наверное можно даже попрограммировать процедуры с автономными трансакциями, создать материализованное представление и т.п.

### 2. Как быстро получить данные из Product в Excel или Access

ODBC можно использовать с любыми совместимыми приложениями. Т.к. технология раньше активно продвигалась MS, то в продуктах MS это удобно в особенности. Можно сделать импорт в Access, а можно и в Excel. Для этого нужно создать файл с расширением .dqy, после чего написать в нем (сервер, пользователя и пароль указать нужного):

```text
XLODBC
1
DRIVER={PostgreSQL Unicode};DATABASE=Product;SERVER=127.0.0.1;PORT=5432;UID=<user>;PASSWORD=<passwordd>;SSLmode=disable;ReadOnly=0;Protocol=7.4;FakeOidIndex=0;ShowOidColumn=0;RowVersioning=0;ShowSystemTables=0;ConnSettings=;Fetch=100;Socket=4096;UnknownSizes=0;MaxVarcharSize=255;MaxLongVarcharSize=8190;Debug=0;CommLog=0;Optimizer=0;Ksqo=1;UseDeclareFetch=0;TextAsLongVarchar=1;UnknownsAsLongVarchar=0;BoolsAsChar=1;Parse=0;CancelAsFreeStmt=0;ExtraSysTablePrefixes=dd_;LFConversion=1;UpdatableCursors=1;DisallowPremature=0;TrueIsMinus1=0;BI=0;ByteaAsLongVarBinary=0;UseServerSidePrepare=0;LowerCaseIdentifier=0;GssAuthUseGSS=0;XaOpt=1
select * from Product_rate_plans
```

Всего должно получиться 4 строки, запрос - в последней. Запрос обязательно должен занимать 1 строку, иначе работать не будет. Теперь при открытии это файла запускается Excel (появится запрос операции, нужно согласиться с включением источника данных) с загруженными данными соответственно запроса из файла.
В Access аналогично можно сделать импорт, выбрав в контекстном меню "Источники данных ODBC", причем можно загрузить сразу все таблицы. После этого для анализа данных можно использовать всю мощь и удобство движка Access.

## Список использованной литературы

* [stackoverflow.com](http://stackoverflow.com/questions/6796252/setting-up-postgresql-odbc-on-windows)
* [dbaspot.wordpress.com](https://dbaspot.wordpress.com/2013/05/29/how-to-access-postgresql-from-oracle-database/)
* [community.oracle.com](https://community.oracle.com/thread/2549585)
* [www-01.ibm.com](http://www-01.ibm.com/support/docview.wss?uid=swg21651061)
