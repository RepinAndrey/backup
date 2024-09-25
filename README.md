# Домашнее задание к занятию "`Работа с данными (DDL/DML)`" - `Репин Андрей`


### Задание 1. Резервное копирование

Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования.

Необходимо описать, какие варианты резервного копирования подходят в случаях:

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

> Зависит от того, какого объема база данных. Если небольшого объема, можно делать  полное резервное копирование каждый день.
> Если база большая, можно делать полное резервное копирование раз в неделю, на остальные дни использовать либо дифференциальный бэкап, либо инкрементальный. Требований к скорости восстановления нет.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

> В этом случае необходимо иметь возможность быстрого надежного восстановления. Лучшим способом будет использование дифференциального бэкапа.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

> Возможно использовать репликацию с задержкой, когда реплика создается с определенной задержкой. В этом случае при поломке основной базы, можно переключиться на работающую базу, в которой отсутствует поломка. Другой момент, если в базе закралась старая ошибка, которую сразу не обнаружили, то такой способ не поможет, т.к. ошибка будет и в реплике. 

### Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

> Выгрузка базы данных в специальном формате:
'''
$ pg_dump -Fc mydb > db.dump
'''

> -F format Указывает формат вывода копии. format может принимать следующие значения: c custom - это наиболее гибкий формат вывода, поскольку он позволяет вручную выбирать и переупорядочивать архивированные элементы во время восстановления; d directory -  создаст каталог с одним файлом для каждой таблицы и большого объекта, плюс так называемый файл оглавления, описывающий объекты в машиночитаемом формате, который может прочитать pg_restore; p plain - выводит обычный текстовый файл SQL скрипта (по умолчанию); t tar - выводит архив tar формата, подходящего для ввода в pg_restore.

Восстановление архива в ту же базу данных, из которой он был выгружен, с предварительным удалением текущего содержимого этой базы данных:
```
$ pg_restore -d mydb --clean --create db.dump
```

> -d имя_бд --dbname=имя_бд Указывает имя базы данных для подключения. -c --clean включить в выходной файл команды удаления (DROP) объектов базы данных перед командами создания (CREATE) этих объектов.   -C --create - сформировать в начале вывода команду для создания базы данных и затем подключения к ней. В этом случае не важно, какая база указана в параметрах подключения перед выполнением скрипта. Также, если указан ключ --clean, то скрипт сначала удалит, а затем пересоздаст базу данных перед подключением к ней. С ключом --create в выходной файл также включается комментарий к базе данных (если он задан) и все назначения переменных конфигурации, связанные с базой данных. Также выгружаются права доступа к самой базе данных, если не добавлен ключ --no-acl. Этот параметр игнорируется, когда данные выгружаются в архивных форматах (не в текстовом). Для таких форматов данный параметр можно указать при вызове pg_restore.

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

> Да, написав скрипт создания бэкапов и бодавив его в cron, запуская по расписанию.

### Задание 3*. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL.

> MySQL поддерживает инкрементные резервные копии: следует запустить сервер с опцией --log-bin , чтобы включить двоичное журналирование. 
> В MySQL можно реализовать создание инкрементальных резервных копий с помощью резервного копирования двоичных файлов журнала. Все транзакции, применяемые к серверу MySQL, последовательно записываются в двоичные файлы журнала. Следовательно, всегда можно восстановить исходную базу данных из этих файлов.

```
mysqldump --flush-logs --delete-master-logs --single-transaction --all-databases | gzip > /var/backups/mysql/$(date +%d-%m-%Y_%H-%M-%S)-inc.gz
```

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

> Обработка транзакций в реальном времени. Репликация позволяет иметь актуальные копии данных, которые можно использовать для непрерывной обработки транзакций, даже если основной сервер выйдет из строя.
> Распределение нагрузки. Репликация может использоваться для распределения нагрузки на чтение между несколькими серверами, что особенно полезно для обслуживания большого количества запросов на чтение. 
> Тестирование и разработка. Реплики могут использоваться для создания стабильных сред для тестирования и разработки, где можно безопасно экспериментировать с данными, не влияя на рабочую базу данных.
> Регулярное обновление и мгновенный доступ. Репликация синхронизирует данные между основным и реплицированными серверами почти в реальном времени, в то время как резервные копии создаются через определённые интервалы времени. Это означает, что реплики обеспечивают более актуальные данные для запросов и восстановления. 