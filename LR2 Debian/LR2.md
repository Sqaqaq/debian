# Отчет по лаборатноной работе №2 Лахмостовой Ю. А. группа ИС-21

1\. Утилиты резервного копирования. Изучить и сравнить pg_dump и
pg_basebackup. Описать сценарии применения каждой утилиты.

PostgreSQL предоставляет два основных способа резервного копирования --
логический (pg_dump) и физический (pg_basebackup).

**Логический бэкап (pg_dump):**

-   Копирует структуру (таблицы, схемы) и данные отдельно.

-   Позволяет гибко восстанавливать (можно выбрать, какие таблицы или
    схемы восстановить).

-   Подходит для миграции между серверами (даже если у них разные версии
    PostgreSQL).

**Физический бэкап (pg_basebackup)**

-   Копирует все файлы базы данных PostgreSQL.

-   Используется для репликации и восстановления сервера в том же виде.

-   Не позволяет выбрать отдельные базы или таблицы -- всё копируется
    целиком.

-   Подходит для аварийного восстановления и создания реплики БД.

2\. Выполнить полное резервное копирование (dump) базы данных из лаб.
№1. Использовать параметры (например, -Fc, -Ft, обязательно посмотреть
другие параметры).

![](LR2/media/image1.png) 
![](LR2/media/image2.png)

-   psql -U postgres - Подключаемся к PostgreSQL

-   \\l - Выводим список БД, с ними можно работать

-   sudo -u postgres pg_dump -U postgres -d dblya -Fc -f
    dblya_backup.dump - Создаём резервную копию

    -   -Fc -- формат custom создает резервную копию в специальном
        бинарном формате, который поддерживает сжатие и позволяет
        восстанавливать отдельные объекты

    -   -Fp -- создает копию в виде SQL скрипта

    -   -Fd -- сохраняет в виде набора файлов в каталоге

    -   -Ft -- tar-архив

3\. Частичное (выборочное) резервное копирование. Сделать дамп только
определённой схемы (например, test_schema, созданной в ЛР №1). Сделать
дамп только определённых таблиц из схемы public. Объяснить, в чём
отличие от резервного копирования всей базы.

![](LR2/media/image3.png) 

![](LR2/media/image4.png) 

![](LR2/media/image5.png)
-   pg_dump -U postgres -d dblya -Fc -f dblya_test_schema.backup.dump -n
    test_schema

    -   -U postgres: используем пользователя PostgreSQL postgres.

    ```{=html}
    <!-- -->
    ```
    -   -d dblya: делаем бэкап базы данных dblya.

    -   -Fc: выбираем формат Custom (сжатый бинарный формат).

    -   -f dblya_test_schema.backup.dump: указываем имя файла резервной
        копии.

    -   -n test_schema: резервируем только схему test_schema, игнорируя
        остальные схемы в базе.

-   pg_dump -U postgres -d dblya -Fc -f dblya_public_table_backup.dump
    **-t public.public_lya_table**

    -   -t public.public_lya_table: вместо -n (схема) используется флаг
        -t, указывающий конкретную таблицу public_lya_table в схеме
        public.

**Разница между полным и частичным копированием.**

Полная копия включает все схемы и таблицы базы dblya.

Частичная копия (-n или -t) позволяет скопировать только выбранные
объекты (определённую схему или таблицу). Полное копирование удобно в
случае перноса всей базы данных. Частичное копирование удобно для
переноса или выгрузки конкретного элемента.

4\. Восстановление из резервной копии. Восстановить базу из резервного
файла с помощью pg_restore или утилиты psql. Продемонстрировать процесс
и результаты.

Восстановление с помощью утилиты psql подходит для формата SQL или же
-Fp. В других случая можно восстановить с помощью pg_restore.

-   createdb -U postgres db1r2 -- создаем новую БД

![](LR2/media/image6.png) 


-   ls -- смотрим содержание бэкапов

![](LR2/media/image7.png) 


-   pg_restore -U postgres -d db1r2 -v dblya_backup.dump

    -   pg_restore: утилита для восстановления бэкапов pg_dump (в
        формате tar, custom, directory).

    -   -U postgres: указываем пользователя PostgreSQL postgres.

    -   -d db1r2: восстанавливаем в новую базу db1r2.

    -   -v: verbose --- показывает детальный вывод (какие таблицы, схемы
        и т.д. создаются).

    -   dblya_backup.dump: наш файл дампа, полная копия базы dblya.

![](LR2/media/image8.png) 


-   psql -U postgres -d db1r2 -- подключаемся к db1r2

-   \\dn -- просмотр схем

-   \\dt -- просмотр таблиц

-   SELECT \* FROM public_lya_table; - проверим наполненность таблицы

![](LR2/media/image9.png)

-   pg_restore -l dblya_backup.dump

    -   -l (list) показывает список объектов (TOC --- Table Of Contents)
        внутри дампа, без фактического восстановления. В выводе можно
        увидеть идентификаторы и названия схем, таблиц, индексов,
        constraints и т.д. Полезно для предпросмотра.

![](LR2/media/image10.png)

5\. Автоматизация бэкапов с помощью cron. Настроить планировщик cron на
Debian, чтобы ежедневно создавать резервные копии. Указать, куда
складываются дампы, и как выполняется ротация.

-   sudo mkdir -p /var/backups/pg -- создаем директорию

![](LR2/media/image11.png) 


-   sudo chown postgres:postgres /var/backups/pg -- выдать права на
    автоматическое создание логов

![](LR2/media/image12.png) 


-   crontab -e -- открываем файл crontab

![](LR2/media/image13.png) 


-   \* / 1 \* \* \* pg_dump -U postgres -Fc -f
    /var/backups/pg/dblya-\$(date +\\%F\_\\%H-\\%M).dump dblya \#
    Ежедневный бэкап базы dblya в формат custom, с датой в названии
    каждую минуту

-   \* / 2 \* \* \* find /var/backups/pg -name \"dblya-\*.dump\" -mtime
    +7 -delete \# Ротация: удаление бэкапов старше 2-х минут

![](LR2/media/image14.png)

-   systemctl start cron - запускаем

-   systemctl status cron -- смотрим статус

-   ls -l /var/backups/pg -- посмотреть содержимое

> В логах (в выводе systemctl status cron) видно, что cron перечитал
> настройки (RELOAD (crontabs)), а затем запускал команды pg_dump и
> find.

![](LR2/media/image15.png) 


Проверка на наличие дампов по
пути.![](LR2/media/image16.png) 


Ротация -- это процесс автоматического удаления резервных копий, которые
старше определённого периода (например, несколько дней или часов). Это
нужно, чтобы:

-   Не переполнялся диск. Со временем бэкапы накапливаются, и, если их
    не удалять, может закончиться место.

-   Поддерживать актуальный набор копий. Хранятся только последние
    (например, за неделю), что позволяет быстро восстановиться в случае
    сбоя без хранения ненужной истории.

6\. Мониторинг состояния системы. Использовать стандартные инструменты
Debian (например, top, htop, iotop) для мониторинга потребления ресурсов
PostgreSQL (CPU, RAM, IO). Уметь объяснить все значения выводимых
показателей.

При вводе команды top, iotop, htop -- отображается таблица с текущими
процессами в системе.

-   12:35:41 up 2 min, 1 user, load average: 2.46, 1.68, 0.69

    -   12:35:41 --- текущее время.

    -   up 2 min --- время, прошедшее с момента загрузки системы.

    -   1 user --- количество залогиненных пользователей.

    -   load average: 2.46, 1.68, 0.69 --- средняя нагрузка на систему
        за последние 1, 5 и 15 минут.

-   Tasks: 171 total, 1 running, 170 sleeping, 0 stopped, 0 zombie

    -   total --- всего процессов (или потоков).

    -   running --- активно выполняющихся процессов.

    -   sleeping --- процессов в режиме ожидания.

    -   zombie --- «зомби»-процессы (уже завершились, но ещё не «убраны»
        родительским процессом).

  ----------------------------------------------------------------------------------------
  PID   USER       PR   NI   VIRT    RES    SHR    S    %CPU   %MEM   TIME+     COMMAND
  ----- ---------- ---- ---- ------- ------ ------ ---- ------ ------ --------- ----------
  626   postgres   20   0    63292   9284   4096   S    0.3    0.5    0:01.25   postgres

  ----------------------------------------------------------------------------------------

-   PID --- идентификатор процесса (Process ID).

```{=html}
<!-- -->
```
-   USER --- под каким пользователем (ОС) запущен процесс.

-   PR и NI --- приоритет (Priority) и nice-значение. Чем выше nice, тем
    «добродушнее» процесс (меньше приоритет).

-   VIRT --- виртуальная память (включая swap, библиотеки, mmap и т.д.).

-   RES --- реальное использование оперативной памяти (resident set
    size).

-   SHR --- общая (shared) память, которую процесс может разделять с
    другими (библиотеки).

-   S --- состояние процесса (S = sleeping, R = running и т.д.).

-   %CPU --- доля CPU, используемая процессом (в процентах).

-   %MEM --- доля оперативной памяти, которую процесс занимает
    (относительно всей RAM).

-   TIME+ --- общее процессорное время, которое процесс уже отработал.

-   COMMAND --- имя процесса (или путь к исполняемому файлу).

![](LR2/media/image17.png)![](LR2/media/image18.png)

htop --- это улучшенная версия top с цветным интерфейсом и более удобным
управлением.

![](LR2/media/image19.png)

iotop --- утилита для мониторинга ввода-вывода (IO), то есть
чтения/записи на диск в реальном времени.

-   TID --- идентификатор потока (thread ID) или процесса.

-   PRIO --- приоритет потока.

-   USER --- пользователь, под которым работает поток/процесс.

-   DISK READ / DISK WRITE --- скорость чтения/записи на диск (KB/s,
    MB/s).

-   SWAPIN --- процент времени, когда процесс «подкачивается» (swap).

-   IO\> --- процент времени, когда процесс занят IO (аналог CPU% в
    top).

-   COMMAND --- имя команды/процесса.

![](LR2/media/image20.png)

Так же можно посмтореть процессы мониторинга в postgres:

![](LR2/media/image21.png) 


7\. Мониторинг PostgreSQL Изучить встроенные представления статистики в
PostgreSQL (например, pg_stat_activity, pg_stat_database). Показать, как
смотреть активные процессы, долгие запросы и т.д. Показать, как можно
принудительно завершить процесс зависший или слишком тяжелый запрос.

-   psql -U postgres -- подключились

-   SELECT \* FROM pg_stat_activity; - это встроенное представление в
    PostgreSQL, которое показывает все активные сессии и их состояние.

![](LR2/media/image22.png) 


Вывели все активные сессии:

-   datid, datname: ID и имя базы данных, к которой подключена сессия.

-   pid: идентификатор процесса PostgreSQL (каждое соединение -- это
    отдельный процесс).

-   usename: пользователь базы данных, под которым идёт сессия.

-   application_name: имя приложения (например, psql, DBeaver, PgAdmin и
    т.д.).

-   client_addr, client_port: IP-адрес и порт клиента, если сессия идёт
    по сети.

-   query: текущий запрос, который выполняется (или последний, если
    сессия idle).

-   query_start: когда этот запрос стартовал.

-   state: статус сессии (active, idle, idle in transaction и т.д.).

![](LR2/media/image23.png) 


SELECT \* FROM pg_stat_database; - просмотр работы БД, всей их
статистики, запросов, ошибок, объемы чтения и записи и т.д.

![](LR2/media/image24.png) 


-   datname -- имя базы данных (например, postgres, template1, dblya).

-   numbackends -- число активных бэкендов (подключений) к этой базе.

-   xact_commit / xact_rollback -- общее количество транзакций, которые
    были зафиксированы (commit) или отменены (rollback).

-   blks_read / blks_hit -- сколько блоков было прочитано с диска и
    сколько --- взято из кеша (hit). Чем больше hit, тем эффективнее
    кэширование.

-   tup_returned / tup_fetched / tup_inserted / tup_updated /
    tup_deleted -- статистика по количеству строк, которые возвращены,
    извлечены, вставлены, обновлены или удалены.

-   conflicts, temp_files, temp_bytes, deadlocks -- дополнительные
    показатели о конфликтующих транзакциях, временных файлах,
    взаимоблокировках и т.д.

-   stats_reset -- дата и время последнего сброса статистики.

![](LR2/media/image25.png) 


-   SELECT pid, age (clock_timestamp(), query_start), query FROM
    pg_stat_activity WHERE state = \'active\' ORDER BY age DESC; - поиск
    долгих запросов

![](LR2/media/image26.png) 


-   pg_sleep() -- пример долгого запроса, с помощью которого можно
    протестировать мониторинг.

![](LR2/media/image27.png) 
![](LR2/media/image28.png) 


-   SELECT pid, usename, application_name, state, query, query_start
    FROM pg_stat_activity WHERE state = \'active\' AND now() -
    query_start \> interval \'2 minutes\'; - вывести запросы, которые
    длятся больше 2-х минут.

-   SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state =
    \'active\' AND now() - query_start \> interval \'2 minutes\'; -
    отключение всех запосов, которые активеые и больше 2-х минут

![](LR2/media/image29.png) 


8\. Логирование и анализ логов. Найти логи PostgreSQL и системные логи
Debian (директория /var/log/, файлы syslog, daemon.log). Определить,
какие события логгирует СУБД, а какие -- ОС.

Логи СУБД.

![](LR2/media/image30.png) 
![](LR2/media/image31.png) 


Логи ОС.

![](LR2/media/image32.png)![](LR2/media/image33.png)


| **Логирование**                   | **PostgreSQL (СУБД)**                                                                                                                                                     | **ОС Debian**                                                                                                                                                                                     |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1. **Запуск и остановка**         | Когда сервер базы данных запускается или останавливается, PostgreSQL записывает сообщения о событии в информационный лог.                                                  | ОС фиксирует, когда запускаются или останавливаются системные сервисы (например, systemd, cron, rsyslog).                                                                                         |
| 2. **Аутентификация и подключения** | При входе пользователей в базу (например, при попытке подключения через psql или приложение) PostgreSQL фиксирует, успешно ли прошла аутентификация, или возникли ошибки (пароль неверный и т.п.). | ОС регистрирует события аутентификации на уровне системы (вход пользователей в систему, использование sudo), а также ошибки при входе (см. /var/log/auth.log).                                    |
| 3. **SQL-запросы и транзакции**   | При включённом логировании запросов (log_statement), СУБД записывает выполняемые SQL-команды. Также фиксируются события транзакций: commit, rollback и т.д.                                                      | ОС Debian фиксирует работу процессов и служб, записи об этом можно найти в `/var/log/syslog` или через `journalctl`. Сами SQL-запросы ОС не видит, только работу процесса `postgres`.             |
| 4. **Ошибки и предупреждения**     | В логи СУБД попадают ошибки запросов (синтаксические ошибки, нарушения ограничений), проблемы с блокировками, а также критические сообщения FATAL или PANIC при сбоях.     | Системные логи отражают ошибки и предупреждения, связанные с оборудованием, сетевыми интерфейсами, драйверами, а также сервисами (например, ошибки при запуске/остановке демонов).                |
| 5. **Репликация и конфигурация**   | События, связанные с настройкой репликации (подключение реплики, сломанная реплика и т.д.), а также изменения в конфигурации PostgreSQL (перезапуск, переинициализация).   | ОС Debian логирует изменения в конфигурации сервисов (например, запись в syslog о перезапуске `postgresql.service`), обновления пакетов (dpkg/apt), а также любые системные конфигурационные правки. |


**PostgreSQL следит за своей работой:**

-- Когда база запускается или останавливается.

-- Какие запросы выполняются, сколько времени они работают, и есть ли
ошибки в них.

-- Когда пользователь пытается подключиться к базе и с каким
результатом.

**ОС Debian следит за работой всей системы:**

-- Когда запускаются или останавливаются сервисы (например, cron, ssh).

-- Как работает оборудование (диски, сеть, процессор).

-- Кто входит в систему и какие действия выполняются на уровне ОС.
