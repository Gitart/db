Оптимизация SQLite сложна. Производительность вложений в C\-приложение может варьироваться от 85 вставок в секунду до более чем 96 000 вставок в секунду!

**Фон:** Мы используем SQLite как часть настольного приложения. У нас есть большое количество данных конфигурации, хранящихся в XML файлах, которые анализируются и загружаются в базу данных SQLite для дальнейшей обработки, когда приложение инициализируется. SQLite идеально подходит для этой ситуации, потому что он быстрый, он не требует специализированной конфигурации, а база данных хранится на диске как один файл.

**Обоснование:** Первоначально я был разочарован тем, что я видел. Оказывается, производительность SQLite может значительно различаться (как для объемных вставок, так и для их выбора) в зависимости от того, как настроена база данных и как вы используете API. Не было тривиального вопроса выяснить, какие все варианты и методы были, поэтому я счел разумным создать эту запись в вики сообщества, чтобы поделиться результатами с помощью Qaru reader, чтобы спасти других от проблем, связанных с теми же исследованиями.

**Эксперимент:** Вместо того, чтобы просто говорить о советах по производительности в общем смысле (т.е. "Использовать транзакцию!" ), я подумал, что лучше написать некоторый код С и на самом деле измерить влияние различных опции. Мы начнем с простых данных:

*   Текстовый файл с разделителями TAB объемом 28 МБ (приблизительно 865 000 записей) [полный тарифный план для города Торонто](http://www.toronto.ca/open/datasets/ttc-routes)
*   Моя тестовая машина \- это 3,6 ГГц P4, работающая под управлением Windows XP.
*   Код скомпилирован с [Visual С++](http://en.wikipedia.org/wiki/Visual_C%2B%2B#32-bit_versions) 2005 как "Release" с "Полная оптимизация" (/Ox) и пользуется Fast Code (/Ot).
*   Я использую SQLite "Amalgamation", скомпилированный непосредственно в тестовое приложение. Версия SQLite, с которой я столкнулась, немного старше (3.6.7), но я подозреваю, что эти результаты будут сопоставимы с последней версией (пожалуйста, оставьте комментарий, если вы думаете иначе).

Запишите код!

**Код:** Простая программа на C, которая читает текстовый файл по строкам, разбивает строку на значения и затем вставляет данные в базу данных SQLite. В этой "базовой" версии кода создается база данных, но мы фактически не будем вставлять данные:

```
/*************************************************************
    Baseline code to experiment with SQLite performance.

    Input data is a 28 MB TAB-delimited text file of the
    complete Toronto Transit System schedule/route info
    from http://www.toronto.ca/open/datasets/ttc-routes/

**************************************************************/
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>
#include "sqlite3.h"

#define INPUTDATA "C:\\TTC_schedule_scheduleitem_10-27-2009.txt"
#define DATABASE "c:\\TTC_schedule_scheduleitem_10-27-2009.sqlite"
#define TABLE "CREATE TABLE IF NOT EXISTS TTC (id INTEGER PRIMARY KEY, Route_ID TEXT, Branch_Code TEXT, Version INTEGER, Stop INTEGER, Vehicle_Index INTEGER, Day Integer, Time TEXT)"
#define BUFFER_SIZE 256

int main(int argc, char **argv) {

    sqlite3 * db;
    sqlite3_stmt * stmt;
    char * sErrMsg = 0;
    char * tail = 0;
    int nRetCode;
    int n = 0;

    clock_t cStartClock;

    FILE * pFile;
    char sInputBuf [BUFFER_SIZE] = "\0";

    char * sRT = 0;  /* Route */
    char * sBR = 0;  /* Branch */
    char * sVR = 0;  /* Version */
    char * sST = 0;  /* Stop Number */
    char * sVI = 0;  /* Vehicle */
    char * sDT = 0;  /* Date */
    char * sTM = 0;  /* Time */

    char sSQL [BUFFER_SIZE] = "\0";

    /*********************************************/
    /* Open the Database and create the Schema */
    sqlite3_open(DATABASE, &db);
    sqlite3_exec(db, TABLE, NULL, NULL, &sErrMsg);

    /*********************************************/
    /* Open input file and import into Database*/
    cStartClock = clock();

    pFile = fopen (INPUTDATA,"r");
    while (!feof(pFile)) {

        fgets (sInputBuf, BUFFER_SIZE, pFile);

        sRT = strtok (sInputBuf, "\t");     /* Get Route */
        sBR = strtok (NULL, "\t");            /* Get Branch */
        sVR = strtok (NULL, "\t");            /* Get Version */
        sST = strtok (NULL, "\t");            /* Get Stop Number */
        sVI = strtok (NULL, "\t");            /* Get Vehicle */
        sDT = strtok (NULL, "\t");            /* Get Date */
        sTM = strtok (NULL, "\t");            /* Get Time */

        /* ACTUAL INSERT WILL GO HERE */

        n++;
    }
    fclose (pFile);

    printf("Imported %d records in %4.2f seconds\n", n, (clock() - cStartClock) / (double)CLOCKS_PER_SEC);

    sqlite3_close(db);
    return 0;
}
```

---

## "Контроль"

Выполнение кода as\-is фактически не выполняет каких\-либо операций с базой данных, но это даст нам представление о том, насколько быстро выполняются операции ввода\-вывода с исходным кодом C и строки.

> Imported 864913 записей в 0.94 секунд

Отлично! Мы можем делать 920 000 вставок в секунду, если мы фактически не делаем никаких вставок: \-)

---

## "Сценарий наихудшего случая"

Мы собираемся сгенерировать строку SQL с использованием значений, считанных из файла, и вызвать эту операцию SQL с помощью sqlite3\_exec:

```
sprintf(sSQL, "INSERT INTO TTC VALUES (NULL, '%s', '%s', '%s', '%s', '%s', '%s', '%s')", sRT, sBR, sVR, sST, sVI, sDT, sTM);
sqlite3_exec(db, sSQL, NULL, NULL, &sErrMsg);
```

Это будет медленным, потому что SQL будет скомпилирован в код VDBE для каждой вставки, и каждая вставка произойдет в его собственной транзакции. Как медленно?

> Imported 864913 записей в 9933.61 секунд

Хлоп! 2 часов и 45 минут! Это только **85 вложений в секунду.**

## Использование транзакции

По умолчанию SQLite будет оценивать каждый оператор INSERT/UPDATE в уникальной транзакции. Если вы выполняете большое количество вставок, рекомендуется выполнить транзакцию в транзакции:

```
sqlite3_exec(db, "BEGIN TRANSACTION", NULL, NULL, &sErrMsg);

pFile = fopen (INPUTDATA,"r");
while (!feof(pFile)) {

    ...

}
fclose (pFile);

sqlite3_exec(db, "END TRANSACTION", NULL, NULL, &sErrMsg);
```

> Imported 864913 записей в 38.03 секунд

Это лучше. Простое перенос всех наших вставок в одну транзакцию улучшил нашу производительность до **23 000 вставок в секунду.**

## Использование подготовленного заявления

Использование транзакции было огромным улучшением, но повторная компиляция оператора SQL для каждой вставки не имеет смысла, если мы используем один и тот же SQL\-код over\-and\-over. Позвольте использовать `sqlite3_prepare_v2` для компиляции нашей инструкции SQL один раз, а затем привязать наши параметры к этому выражению, используя `sqlite3_bind_text`:

```
/* Open input file and import into the database */
cStartClock = clock();

sprintf(sSQL, "INSERT INTO TTC VALUES (NULL, @RT, @BR, @VR, @ST, @VI, @DT, @TM)");
sqlite3_prepare_v2(db,  sSQL, BUFFER_SIZE, &stmt, &tail);

sqlite3_exec(db, "BEGIN TRANSACTION", NULL, NULL, &sErrMsg);

pFile = fopen (INPUTDATA,"r");
while (!feof(pFile)) {

    fgets (sInputBuf, BUFFER_SIZE, pFile);

    sRT = strtok (sInputBuf, "\t");   /* Get Route */
    sBR = strtok (NULL, "\t");        /* Get Branch */
    sVR = strtok (NULL, "\t");        /* Get Version */
    sST = strtok (NULL, "\t");        /* Get Stop Number */
    sVI = strtok (NULL, "\t");        /* Get Vehicle */
    sDT = strtok (NULL, "\t");        /* Get Date */
    sTM = strtok (NULL, "\t");        /* Get Time */

    sqlite3_bind_text(stmt, 1, sRT, -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 2, sBR, -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 3, sVR, -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 4, sST, -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 5, sVI, -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 6, sDT, -1, SQLITE_TRANSIENT);
    sqlite3_bind_text(stmt, 7, sTM, -1, SQLITE_TRANSIENT);

    sqlite3_step(stmt);

    sqlite3_clear_bindings(stmt);
    sqlite3_reset(stmt);

    n++;
}
fclose (pFile);

sqlite3_exec(db, "END TRANSACTION", NULL, NULL, &sErrMsg);

printf("Imported %d records in %4.2f seconds\n", n, (clock() - cStartClock) / (double)CLOCKS_PER_SEC);

sqlite3_finalize(stmt);
sqlite3_close(db);

return 0;
```

> Imported 864913 записей в 16.27 секунд

Ницца! Там немного больше кода (не забудьте вызвать `sqlite3_clear_bindings` и `sqlite3_reset`), но мы увеличили нашу производительность до **53 000 вставок в секунду.**

## PRAGMA synchronous = OFF

По умолчанию SQLite приостанавливается после выдачи команды записи на уровне ОС. Это гарантирует, что данные записываются на диск. Установив `synchronous = OFF`, мы инструктируем SQLite просто передавать данные в ОС для записи, а затем продолжить. Там вероятность того, что файл базы данных может стать поврежденным, если компьютер страдает катастрофическим сбоем (или сбоем питания), прежде чем данные будут записаны в пластинку:

```
/* Open the database and create the schema */
sqlite3_open(DATABASE, &db);
sqlite3_exec(db, TABLE, NULL, NULL, &sErrMsg);
sqlite3_exec(db, "PRAGMA synchronous = OFF", NULL, NULL, &sErrMsg);
```

> Imported 864913 записей в 12.41 секунд

Усовершенствования теперь меньше, но мы до **69 600 вставок в секунду.**

## PRAGMA journal\_mode = MEMORY

Рассмотрите возможность хранения журнала отката в памяти путем оценки `PRAGMA journal_mode = MEMORY`. Ваша транзакция будет быстрее, но если вы потеряете электроэнергию или ваша программа выйдет из строя во время транзакции, вы можете оставить базу данных в поврежденном состоянии с частично выполненной транзакцией:

```
/* Open the database and create the schema */
sqlite3_open(DATABASE, &db);
sqlite3_exec(db, TABLE, NULL, NULL, &sErrMsg);
sqlite3_exec(db, "PRAGMA journal_mode = MEMORY", NULL, NULL, &sErrMsg);
```

> Imported 864913 записей в 13.50 секунд

Немного медленнее предыдущей оптимизации на **64 000 вставок в секунду.**

## PRAGMA synchronous = OFF и PRAGMA journal\_mode = MEMORY

Объедините предыдущие две оптимизации. Это немного рискованно (в случае сбоя), но мы просто импортируем данные (не запускаем банк):

```
/* Open the database and create the schema */
sqlite3_open(DATABASE, &db);
sqlite3_exec(db, TABLE, NULL, NULL, &sErrMsg);
sqlite3_exec(db, "PRAGMA synchronous = OFF", NULL, NULL, &sErrMsg);
sqlite3_exec(db, "PRAGMA journal_mode = MEMORY", NULL, NULL, &sErrMsg);
```

> Засчитано 864913 записей в 12.00 секунд

Fantastic! Мы можем сделать **72 000 вставок в секунду.**

## Использование базы данных с памятью

Просто для ударов, опирайтесь на все предыдущие оптимизации и переопределяем имя файла базы данных, поэтому мы полностью работаем в ОЗУ:

```
#define DATABASE ":memory:"
```

> Imported 864913 записей в 10.94 секунд

Это не супер\-практично хранить нашу базу данных в ОЗУ, но впечатляет, что мы можем выполнять **79 000 вставок в секунду.**

## Рефакторинг кода C

Хотя это не является особым улучшением SQLite, мне не нравятся дополнительные операции присваивания `char*` в цикле `while`. Позвольте быстро реорганизовать этот код, чтобы передать вывод `strtok()` непосредственно в `sqlite3_bind_text()`, и пусть компилятор попытается ускорить процесс для нас:

```
pFile = fopen (INPUTDATA,"r");
while (!feof(pFile)) {

    fgets (sInputBuf, BUFFER_SIZE, pFile);

    sqlite3_bind_text(stmt, 1, strtok (sInputBuf, "\t"), -1, SQLITE_TRANSIENT); /* Get Route */
    sqlite3_bind_text(stmt, 2, strtok (NULL, "\t"), -1, SQLITE_TRANSIENT);    /* Get Branch */
    sqlite3_bind_text(stmt, 3, strtok (NULL, "\t"), -1, SQLITE_TRANSIENT);    /* Get Version */
    sqlite3_bind_text(stmt, 4, strtok (NULL, "\t"), -1, SQLITE_TRANSIENT);    /* Get Stop Number */
    sqlite3_bind_text(stmt, 5, strtok (NULL, "\t"), -1, SQLITE_TRANSIENT);    /* Get Vehicle */
    sqlite3_bind_text(stmt, 6, strtok (NULL, "\t"), -1, SQLITE_TRANSIENT);    /* Get Date */
    sqlite3_bind_text(stmt, 7, strtok (NULL, "\t"), -1, SQLITE_TRANSIENT);    /* Get Time */

    sqlite3_step(stmt);        /* Execute the SQL Statement */
    sqlite3_clear_bindings(stmt);    /* Clear bindings */
    sqlite3_reset(stmt);        /* Reset VDBE */

    n++;
}
fclose (pFile);
```

**Примечание. Мы вернемся к использованию реального файла базы данных. Базы данных в памяти бывают быстрыми, но не обязательно практичными.**

> Imported 864913 записей в 8.94 секунд

Небольшой рефакторинг для кода обработки строк, используемого при связывании параметров, позволил нам выполнить **96 700 вставок в секунду.** Я считаю, что можно сказать, что это довольно быстро. Когда мы начнем изменять другие переменные (т.е. Размер страницы, создание индекса и т.д.), Это будет нашим эталоном.

---

## Резюме (до сих пор)

Надеюсь, ты все еще со мной! Причина, по которой мы пошли по этой дороге, заключается в том, что производительность вместительной вставки сильно отличается от SQLite, и не всегда очевидно, какие изменения необходимо внести для ускорения нашей работы. Используя тот же самый компилятор (и параметры компилятора), ту же версию SQLite и те же данные, мы оптимизировали наш код и наше использование SQLite для перехода **из наихудшего сценария из 85 вставок в секунду и более 96 000 вставок в секунду!**

---

## CREATE INDEX, затем INSERT vs. INSERT, затем CREATE INDEX

Прежде чем мы начнем измерять производительность `SELECT`, мы знаем, что будем создавать индексы. В одном из ответов ниже было предложено, что при выполнении массовых вставок быстрее создавать индекс после того, как данные были вставлены (вместо того, чтобы сначала создавать индекс, а затем вставлять данные). Попробуйте:

**Создать индекс, а затем вставить данные**

```
sqlite3_exec(db, "CREATE  INDEX 'TTC_Stop_Index' ON 'TTC' ('Stop')", NULL, NULL, &sErrMsg);
sqlite3_exec(db, "BEGIN TRANSACTION", NULL, NULL, &sErrMsg);
...
```

> Imported 864913 записей в 18.13 секунд

**Вставить данные, а затем Создать индекс**

```
...
sqlite3_exec(db, "END TRANSACTION", NULL, NULL, &sErrMsg);
sqlite3_exec(db, "CREATE  INDEX 'TTC_Stop_Index' ON 'TTC' ('Stop')", NULL, NULL, &sErrMsg);
```

> Imported 864913 записей в 13.66 секунд

Как и ожидалось, объемные вставки медленнее, если индексируется один столбец, но имеет значение, если индекс создается после того, как данные вставлены. Наш базовый уровень без индекса составляет 96 000 вставок в секунду. **Создание индекса сначала, а затем вставка данных дает нам 47 700 вставок в секунду, тогда как вставка данных сначала, а затем создание индекса дает нам 63 300 вставок в секунду.**

---

Я бы с радостью принял предложения по другим сценариям, чтобы попробовать... И скоро будет компилировать похожие данные для запросов SELECT.



## Несколько советов:

1.  Вставьте вставки/обновления в транзакцию.
2.  Для более старых версий SQLite \- рассмотрите режим менее параноидального журнала (`pragma journal_mode`). Существует `NORMAL`, а затем есть `OFF`, что может значительно увеличить скорость вставки, если вы не слишком беспокоитесь о том, что база данных может быть повреждена, если ОС сбой. Если ваше приложение выходит из строя, данные должны быть точными. Обратите внимание, что в более новых версиях настройки `OFF/MEMORY` небезопасны для сбоев на уровне приложений.
3.  Игра с размерами страниц также имеет значение (`PRAGMA page_size`). Имея большие размеры страниц, вы можете сделать чтение и запись немного быстрее, поскольку в памяти хранятся более крупные страницы. Обратите внимание, что для вашей базы данных будет использоваться больше памяти.
4.  Если у вас есть индексы, подумайте о вызове `CREATE INDEX` после выполнения всех ваших вставок. Это значительно быстрее, чем создание индекса, а затем выполнение ваших вставок.
5.  Вы должны быть достаточно осторожны, если у вас есть одновременный доступ к SQLite, поскольку вся база данных заблокирована при выполнении записи, и, хотя возможны несколько считывателей, записи будут заблокированы. Это несколько улучшилось с добавлением WAL в новых версиях SQLite.
6.  Воспользуйтесь преимуществами экономии места... более мелкие базы данных идут быстрее. Например, если у вас есть пары ключевых значений, попробуйте сделать ключ `INTEGER PRIMARY KEY` если это возможно, что заменит подразумеваемый уникальный столбец строк в таблице.
7.  Если вы используете несколько потоков, вы можете попробовать использовать [кеш разделяемой страницы](http://sqlite.org/c3ref/enable_shared_cache.html), который позволит обмениваться загружаемыми страницами между потоками, что позволяет избежать дорогостоящих вызовов ввода\-вывода.
8.  [Не используйте `!feof(file)` !](https://stackoverflow.com/questions/5431941/why-is-while-feof-file-always-wrong)
