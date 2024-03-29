# 20 вопросов и ответов на знание базы данных SQLite

Хорошая подготовка \- залог успешного прохождения интервью. CareerGuru99 собрали топ\-20 вопросов на знание базы данных SQLite, а мы перевели их для вас.

# 1) Что такое SQLite?

SQLite \- это система управления реляционными базами данных, совместимая с ACID, содержащаяся в относительно небольшой библиотеке языка C.

# 2) Перечислите список стандартных команд SQLite.

Стандартные команды SQLite, взаимодействующие с реляционными базами данных, аналогичны SQL.

*   SELECT
*   CREATE
*   INSERT
*   UPDATE
*   DROP
*   DELETE

# 3) Что такое транзакции SQLite?

Транзакция называется единица работы, которая выполняется в отношении базы данных. Это одно или несколько изменений в базе данных, свойства которых определяются ACID.

*   Атомарность: гарантирует, что все транзакции успешно завершены.
*   Согласованность: гарантирует, что база данных меняет состояние при успешной транзакции.
*   Изолированность: позволяет транзакциям работать прозрачно и независимо друг от друга.
*   Устойчивость: гарантирует, что результат или эффект совершенной транзакции сохранится в случае сбоя системы.

# 4) В каких областях используется SQLite?

SQLite работает с

*   Встроенными устройствами и Интернетом вещей
*   Форматами файла приложения
*   Анализом данных
*   Веб\-сайтами
*   Кэшем для корпоративных данных
*   Базами данных на стороне сервера
*   Файловыми архивами
*   Внутренними или временными базами данных
*   Заменой для файлов ad hoc
*   Экспериментальными расширения языка SQL
*   В режиме ожидания для базы данных предприятия во время демонстрации или тестирования

# 5) В чем разница между SQL и SQLite?

| SQL | SQLite |
| --- | --- |
| SQL \- это структурированный язык запросов | SQLite \- это мощная встроенная система управления реляционными базами данных, в основном используемая в мобильных устройствах для хранения данных |
| Поддерживаемые SQL\-процедуры | SQLite не поддерживает хранимые процедуры |
| SQL основан на сервере | SQLite основан на файлах |

# 6) Перечислите преимущества SQLite?

*   Для работы не требуется отдельная серверная процессорная система
*   Нет необходимости в настройке или администрировании. SQlite поставляется с нулевой конфигурацией
*   База данных SQLite может храниться в одном кросс\-платформенном диске
*   SQLite очень компактен \- менее 400 KiB
*   SQLite является автономным, что означает отсутствие внешних зависимостей
*   Он поддерживает практически все типы ОС
*   Он написан на ANSI\-C и предоставляет простой в использовании API

# 7) Укажите, какие классы хранения есть в SQLite?

Классы хранения SQLite включают

*   Null : имеет значение NULL
*   Integer: представляет собой целое число со знаком (1,2,3 и т. д.),
*   Real: IEEE 8\-байтовое число с плавающей запятой
*   Text: текстовая строка, хранящаяся с использованием кодировки базы данных (UTF\-8, UTF\-16BE)
*   BLOB (Binary Large Object) : блок данных, точно сохраненный при вводе

# 8) Как в SQLite хранятся булевы значения?

Булевы значения в SQLite хранятся в виде целых чисел 0 (false) и 1 (true). SQLite не имеет отдельного булева класса хранения.

# 9) Как работает команда group by в SQLITE?

group by используется вместе с оператором SELECT для организации идентичных данных в группы.

# 10) Назовите команду, которая используется для создания базы данных в SQLite?

Для создания базы данных в SQLite используется команда «sqlite3». Основной синтаксис создания базы данных \- $ sqlite3 DatabaseName.db.

# 11) Для чего используется команда .dump?

Команда .dump используется для создания дампа базы данных SQLite, но стоит помнить, что при после ее использования все данные будут сброшены навсегда и восстановить их будет невозможно.

# 12) Как удалить или добавить столбцы из/в существующей таблицы в SQLite?

Существует очень ограниченная поддержка для команды alter (добавить или удалить). Если вы хотите удалить или добавить столбцы из существующей таблицы в SQLite, вы должны сначала сохранить существующие данные во временную таблицу, сбросить старую таблицу или столбец, создать новую таблицу и затем скопировать данные с временной таблица.

# 13) Какой максимальный размер VARCHAR в SQLite?

SQLite не имеет определенной длины для VARCHAR. Например, вы можете объявить VARCHAR (10), и SQLite сохранит там 500 миллионов символов. Он сохранит в целости все 500 символов.

# 14) Назовите случаи, когда нужно использовать SQLite, а когда нет?

SQLite можно использовать в следующих условиях:

*   Встроенные приложения: не требуют расширения, например, мобильные приложения или игры
*   Disk assess replacement: приложение, которое требует прямой записи или чтения файлов на диск
*   Тестирование: при тестировании логики бизнес\-приложений

Когда не нужно использовать SQLite:

*   Многопользовательские приложения: в тех случаях, когда несколько клиентов должны иметь доступ и использовать одну и ту же базу данных
*   Приложения, требующие записей в больших объемах: он позволяет использовать только одну операцию записи в любой момент времени

# 15) Как восстановить удаленные данные из базы данных SQLite?

Для восстановления информации, вы можете использовать резервную копию файла базы данных, если ее нет, восстановление невозможно. SQLite использует SQLITE SECURE DELETE, который перезаписывает все удаленное содержимое нулями.

# 16) В каком случае можно получить ошибку SQLITE\_SCHEMA?

Ошибка SQLITE\_SCHEMA возникает, если подготовленный оператор SQL недействителен и не может быть выполнен. Такой тип ошибок возникает только при использовании интерфейсов sqlite3 prepare () и sqlite3 step () для запуска SQL.

# 17) Что такое EECN в SQLite?

Исходный код основного источника публичного домена SQLite не описывается никаким ECCN. Следовательно, ECCN следует указывать как EAR99. Но, если вы добавляете новый код или связываете SQLite с приложением, он может изменить номер EECN.

# 18) Объясните, что такое представление в SQLite?

В SQLite представление фактически представляет собой состав таблицы в виде предопределенного запроса SQLite. Представление может состоять из всех строк таблицы или выбранных строк из одной или нескольких таблиц.

# 19) Что такое индексы SQLite?

Индексы SQLite представляют собой специальные таблицы поиска, используемые поисковой системой базы данных для ускорения нахождения данных. Простыми словами, это указатель на данные в таблице.

# 20) Когда следует избегать индексов?

Следует избегать индексов, если

*   Таблицы небольшие
*   Таблицы часто меняются
*   Столбцы, которые часто используются или имеют большое количество значений NULL
