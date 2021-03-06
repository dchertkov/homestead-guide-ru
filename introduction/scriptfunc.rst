################################################################################
Встроенные функции языка написания контрактов
################################################################################

Функции языка написания контрактов выполняют операции с данными полученными в секции data контракта: чтение значений из базы данных и запись значений в базу данных, преобразование типов значений и установление связи между контрактами. 

Функции не возвращают ошибок, так как все проверки на ошибки происходят автоматически.
При генерации ошибки в любой из функции, контракт прекращает свою работу и выводит описание ошибки в специальном окне.

********************************************************************************
Предопределенные значения
********************************************************************************

При выполнении контракта доступны следующие переменные.

* **$key_id** - числовой идентификатор (int64) кошелька, который подписал транзакцию.
* **$ecosystem_id** - идентификатор экосистемы, которому принадлежит гражданин, и в котором была создана данная транзакция. 
* **$type** - идентификатор вызываемого контракта. Если, например, контракт вызвал другой контракт, то здесь будет хранится идентификатор оригинального контракта.
* **$time** - время указанное в транзакции в формате Unix.
* **$block** - номер блока, в котором запечаталась данная транзакция. 
* **$block_time** - время указанное в блоке. 
* **$block_key_id** - числовой идентифкатор (int64) ноды, которая подписала блок. 

Следует иметь в виду, ято данные переменные доступны не только в функциях контракта, но и в прочих функциях и выражениях, например, в условиях, которые указываются для контрактов, страниц и прочих объектах. В этом случае, относящиеся к блоку переменные *$time, $block* и прочие, равны 0.

Если вы хотите возвратить из контракта значение, то присвойте его прредопределенной переменной **$result**.

********************************************************************************
Получение значений из базы данных
********************************************************************************

DBFind(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Limit(limit int)] [.Offset(offset int)] [.Ecosystem(ecosystemid int)] array
==========================
Функция получает данные из таблицы базы данных в соответствии с указанным запросом. Возвращается массив *array* состоящий из ассоциативных массивов *map*.

* *table* - имя таблицы.
* *сolumns* - список возвращаемых колонок. Если не указано, то возвратятся все колонки. 
* *Where* - условие поиска. Например, *.Where("name = 'John'")* или  *.Where("name = ?", "John")*
* *id* - поиск по идентификатору. Достаточно указать значение идентификатора.  Например, *.WhereId(1)*
* *order* - поле, по которому нужно отсортировать. Пол умолчанию, сортируется по *id*.
* *limit* - количество возвращаемых записей. По умолчанию, 25. Максимально возможно количество - 250.
* *offset* - смещение возвращаемых записей.
* *ecosystemid* - идентификатор экосистемы. По умолчанию, берутся данные из таблицы в текущей экосистеме.

.. code:: js

   var i int
   ret = DBFind("contracts").Columns("id,value").Where("id> ? and id < ?", 3, 8).Order("id")
   while i < Len(ret) {
       var vals map
       vals = ret[0]
       Println(vals["value"])
       i = i + 1
   }

DBAmount(tblname string, column string, id int) money
==============================
Функция возвращает значение колонки **amount** с типом *money* c поиском записи по значению указанной колонки таблицы. (Для получения данных типа money нельзя использовать  функции **DBInt()** и **DBIntExt()**, возвращающие  значения типа *int*).

* *tblname* - имя таблицы в базе данных;
* *column* - имя колонки, по которой будет идти поиск записи;
* *id* - значение для поиска записи, выборка *column=id*.

.. code:: js

    mymoney = DBAmount("dlt_wallets"), "wallet_id", $wallet)
	
EcosysParam(name string) string
==============================
Функция возвращает значение указанного параметра из настроек экосистемы (таблица *parameters*).

* *name* - имя получаемого параметра.

.. code:: js

    Println( EcosysParam("gov_account"))


DBInt(tblname string, name string, id int) int
==============================
Функция возвращает числовое значение из таблицы базы данных по указанному **id** записи.

* *tblname* - имя таблицы в базе данных.
* *name* - имя колонки, значение которой будет возвращено.
* *id* - идентификатор поля **id** записи, из которой будет взято значение.

.. code:: js

    var val int
    val = DBInt("mytable", "counter", 1)

DBIntExt(tblname string, name string, val (int|string), column string) int
==============================
Функция возвращает числовое значение из таблицы базы данных с поиском записи по указанному полю и значению.

* *tblname* - имя таблицы в базе данных.
* *name* - имя колонки, значение которой будет возвращено.
* *val* - значение, по которому будет искаться запись.
* *column* - имя колонки, по которой будет искаться запись; таблица должна иметь индекс по данной колонке.

.. code:: js

    var val int
    val = DBIntExt("mytable", "balance", $wallet, "wallet_id")

DBIntWhere(tblname string, name string, where string, params ...) int
==============================
Функция возвращает числовое значение из колонки таблицы базы данных с поиском записи по условиям указанным в **where**.

* *tblname* - имя таблицы в базе данных.
* *name* - имя колонки, значение которой будет возвращено.
* *where* - условия запроса для выборки записей; имена полей располагаются слева от знаков сравнения; для подстановки параметров используются символы **?** или **$**.
* *params* - параметры, подставляемые в условия запроса в заданной последовательности.

.. code:: js

    var val int
    val = DBIntWhere("mytable", "counter",  "idgroup = ? and statue=?", mygroup, 1 )

DBRowExt(tblname string, columns string, val (int|string), column string) map
==============================
Функция возвращает массив (map) значениий из таблицы базы данных с поиском записи по указанному полю и значению.

* *tblname* - имя таблицы в базе данных;
* *columns* - имя колонок, значение которых необходимо получить;
* *val* - значение, по которому будет искаться запись;
* *column* - имя колонки, по которой будет искаться запись. Таблица должна иметь индекс по данной колонке.

.. code:: js

    var vals map
    vals = DBRowExt("mytable", "address,postindex,name", $Company, "company" )

DBString(tblname string, name string, id int) string
==============================
Функция возвращает строковое значение из колонки таблицы базы данных по **id** записи.

* *tblname* - имя таблицы в базе данных.
* *name* - имя колонки, значение которой будет возвращено.
* *id* - идентификатор поля **id** записи, из которой будет взято значение.

.. code:: js

    var val string
    val = DBString("mytable", "name", $citizen)

DBStringExt(tblname string, name string, val (int|string), column string) string
==============================
Функция возвращает строковое значение из таблицы базы данных с поиском записи по указанному полю и значению.

* *tblname* - имя таблицы в базе данных;
* *name* - имя колонки, значение которой будет возвращено;
* *val* - значение, по которому будет искаться запись;
* *column* - имя колонки, по которой будет искаться запись. Таблица должна иметь индекс по данной колонке.

.. code:: js

    var val string
    val = DBStringExt("mytable", "address", $Company, "company" )
    
DBStringWhere(tblname string, name string, where string, params ...) string
==============================
Функция возвращает строковое значение из колонки таблицы базы данных с поиском записи по условиям указанным в *where*.

* *tblname* - имя таблицы в базе данных.
* *name* - имя колонки, значение которой будет возвращено.
* *where* - условия запроса для выборки записей; имена полей располагаются слева от знаков сравнения; для подстановки параметров используются символы **?** или **$**.
* *params* - параметры, подставляемые в условия запроса в заданной последовательности.

.. code:: js

    var val string
    val = DBStringWhere("mytable", "address",  "idgroup = ? and company=?",
           mygroup, "My company" )

	
LangRes(idres string, lang string) string
==============================
Функция возвращает языковой ресурс с именем idres для языка lang. Язык указывает в виде двухсимвольного кода, например, *en,fr,ru*. Поиск идет в соответствующей экосистеме. Если для такого языка нет ресурса, то возвращается на английском языке.

* *idres* - имя языкового ресурса;
* *lang* - двухсимвольный код языка;

.. code:: js

    warning LangRes("confirm", $Lang)
    error LangRes("problems", "de")
	
********************************************************************************
Изменение значений в таблицах 
********************************************************************************

DBInsert(tblname string, params string, val ...) int
==============================
Функция добавляет запись в указанную таблицу и возвращает **id** вставленной записи.

* *tblname* - имя таблицы в базе данных.
* *params* - список через запятую имен колонок, в которые будут записаны перечисленные в **val** значения. 
* *val* - список через запятую значений для перечисленных в **params** столбцов; значения могут иметь строковый или числовой тип.

.. code:: js

    DBInsert("mytable", "name,amount", "John Dow", 100)

DBInsertReport(tblname string, params string, val ...) int
==============================
Функция добавляет запись в указанную таблицу с отчетами и возвращает **id** вставленной записи. Данная функция практически идентична функции DBInsert, но запись возможна только в таблицу отчетов своего государства.

* *tblname* - имя таблицы в базе данных. Таблица для отчетов в базе данных должна иметь имя в формате **[state_id]_reports_[tblname]**.
* *params* - список через запятую имен колонок, в которые будут записаны перечисленные в **val** значения. 
* *val* - список через запятую значений для перечисленных в **params** столбцов; значения могут иметь строковый или числовой тип.

.. code:: js

    DBInsertReport("mytable", "name,amount", "John Dow", 100)

DBUpdate(tblname string, id int, params string, val...)
==============================
Функция изменяет значения столбцов в таблице в записи с указанным **id**.

* *tblname* - имя таблицы в базе данных.
* *id* - идентификатор **id** изменяемой записи.
* *params* - список имен изменяемых колонок; перечисляются через запятую.
* *val* - список значений для указанных столбцов перечисленных в **params**; могут иметь строковый или числовой тип.

.. code:: js

    DBUpdate("mytable", myid, "name,amount", "John Dow", 100)

DBUpdateExt(tblname string, column string, value (int|string), params string, val ...)
==============================
Функция обновляет столбцы в записи, у которой колонка имеет заданное значение. Таблица должна иметь индекс по указанной колонке.

* *tblname* - имя таблицы в базе данных.
* *column* - имя колонки, по которой будет идти поиск записи.
* *value* - значение для поиска записи в колонке.
* *params* - список имен колонок, в которые будут записаны значения указанные в **val**; перечисляются через запятую.
* *val* - список значений для записи в колонки перечисленные в  **params**; могут иметь строковый или числовой тип.

.. code:: js

    DBUpdateExt("mytable", "address", addr, "name,amount", "John Dow", 100)

********************************************************************************
Работа с контрактами и языком
********************************************************************************

CallContract(name string, params map)
==============================
Функция вызывает контракт по его имени. В передаваемом массиве должны быть перечислены все параметры, указанные в section data контракта. Функция возвращает значение, которое было присвоено переменной **$result** в контракте.

* *name* - имя вызываемого контракта.
* *params* - ассоциативный массив с входными данными для контракта.

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)

ContractAccess(name string, [name string]) bool
==============================
Функция проверяет, совпадает ли имя выполняемого контракта с одним из имен, перечисленных в параметрах. Как правило используется для контроля доступа контрактов к таблицам. Функция прописывается в полях *Permissions* при редактировании колонок таблицы или в полях  *Insert* и *New Column* в разделе *Table permission*.

* *name* - имя контракта.

.. code:: js

    ContractAccess("MyContract")  
    ContractAccess("MyContract","SimpleContract") 
    
ContractConditions(name string, [name string]) bool
==============================
Функция вызывает секцию **conditions** из контрактов с указанными именами. У таких контрактов блок *data* должен быть пустой. Если секция *conditions* выполнилась без ошибок, то возвращается *истина*. Если в процессе выполнения сгенерировалась ошибка, то родительский контракт также завершится с данной ошибкой. Эта функция, как правило, используется для контроля доступа контрактов к таблицам и может вызываться в полях *Permissions* при редактировании системных таблиц.

* *name* - имя контракта.

.. code:: js

    ContractConditions("MainCondition")  

EvalCondition(tablename string, name string, condfield string) 
==============================
Функция берет из таблицы *tablename* значение поля *condfield* из записи с полем *'name'*, которое равно параметру *name*, и проверяет выполнено ли условие полученное из поля *condfield* или нет. Если условие не выполнено, то генерируется ошибка, с которой и завершается вызывающий контракт.

* *tablename* - имя таблица.
* *name* - значение для поиска по полю 'name'.
* *condfield* - имя поля где хранится условие, которое необходимо будет проверить.

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)  

ValidateCondition(condition string, state int) 
==============================
Функция пытается скомпилировать условие, указанное в параметре *condition*. Если в процессе компиляции условия возникнет ошибка, то будет сгенерирована ошибка и вызывающий контракт закончит свою работу. Данная функция предназначена для проверки правильности условий при их изменении.

* *condition* - проверяемое условие.
* *state* - идентифкатор государства. Укажите ноль, если проверка для глобальных условий.

.. code:: js

    ValidateCondition(`ContractAccess("@0MyContract")`, 0)  

********************************************************************************
Операции со значениями переменных
********************************************************************************
    
AddressToId(address string) int
==============================
Функция возвращает идентификационный номер гражданина по строковому значению адреса его кошелька. Если указан неверный адрес, то возвращается 0.

* *address* - адрес кошелька в формате XXXX-...-XXXX или в виде числа.

.. code:: js

    wallet = AddressToId($Recipient)
    
Contains(s string, substr string) bool
==============================
Функция возвращает true, если строка *s* содержит подстроку *substr*.

* *s* - проверяема строка.
* *substr* - подстрока, которая ищется в указанной строке.

.. code:: js

    if Contains($Name, `my`) {
    ...
    }    

Float(val int|string) float
==============================
Функция преобразует целое число *int* или *string* в число с плавающей точкой.

* *val* - целое число или строка.

.. code:: js

    val = Float("567.989") + Float(232)

HasPrefix(s string, prefix string) bool
==============================
Функция возвращает true, если строка начинается с указанной подстроки *prefix*.

* *s* - проверяема строка.
* *prefix* - проверяемый префикс у данной строки.

.. code:: js

    if HasPrefix($Name, `my`) {
    ...
    }

HexToBytes(hexdata string) bytes
==============================
Функция преобразует строку с шестнадцатеричной кодировкой в значение  типа *bytes* (последовательность байт).

* *hexdata* - строка, содержащая шестнадцатеричную запись.

.. code:: js

    var val bytes
    val = HexToBytes("34fe4501a4d80094")

Join(in array, sep string) string
==============================
Функция объединяет элементы массива *in* в строку с указанным разделителем *sep*.

* *in* - массив типа *array*, элементы которого необходимо объеденить.
* *sep* - строка-разделитель.

.. code:: js

    var val string, myarr array
    myarr[0] = "first"
    myarr[1] = 10
    val = Join(myarr, ",")

Split(in string, sep string) array
==============================
Функция разбивает строку *in* на элементы массива в соответствии с указанным разделителем *sep*.

* *in* - строка, которую необходимо разбить.
* *sep* - строка-разделитель.

.. code:: js

    var myarr array
    myarr = Split("first,second,third", ",")

Int(val string) int
==============================
Функция преобразует строковое значение в целое число.

* *val* - строка содержащая число.

.. code:: js

    mystr = "-37763499007332"
    val = Int(mystr)

Len(val array) int
==============================
Функция возвращает количество элементов в указанном массиве.

* *val* - массив типа *array*.

.. code:: js

    if Len(mylist) == 0 {
      ...
    }

PubToID(hexkey string) int
==============================
Функция возвращает адрес кошелька по публичному ключу в шестнадцатеричной кодировке.

* *hexkey* - публичный ключ в шестнадцатеричном виде.

.. code:: js

    var wallet int
    wallet = PubToID("fa5e78.....34abd6")

Replace(s string, old string, new string) string
==============================
Функция заменять в строку *s* все вхождения строки *old* на строку *new* и возвращает полученный результат.

* *s* - исходная строка.
* *old* - заменяемая строка.
* *new* - новая строка.

.. code:: js

    s = Replace($Name, `me`, `you`)

Size(val string) int
==============================
Функция возвращает размер указанной строки.

* *val* - строка, для которой нужно вычислить размер.

.. code:: js

    var len int
    len = Size($Name)

Sha256(val string) string
==============================
Функция возвращает хэш **SHA256** от указанной строки.

* *val* - входящая строка, для которой нужно вычислить хэш **Sha256**.

.. code:: js

    var sha string
    sha = Sha256("Test message")

Sprintf(pattern string, val ...) string
==============================
Функция формирует строку на основе указанного шаблона и параметров, можно использовать *%d (число), %s (строка), %f (float), %v* (для любых типов).

* *pattern* - шаблон для формирования строки.

.. code:: js

    out = Sprintf("%s=%d", mypar, 6448)



Str(val int|float) string
==============================
Функция преобразует числовое значение типа *int* или *float* в строку.

* *val* - целое или число с плавающей точкой.

.. code:: js

    myfloat = 5.678
    val = Str(myfloat)

Substr(s string, offset int, length int) string
==============================
Функция возвращает подстроку от указанной строки начиная со смещения *offset* (cчитается с 0) и длиной *length*. В случае некорректных смещений или длины возвращается пустая строка. Если сумма смещения и *length* больше размера строки, то возвратится подстрока от смещения до конца строки.

* *val* - строка.
* *offset* - смещение подстроки.
* *length* - размер подстроки.

.. code:: js

    var s string
    s = Substr($Name, 1, 10)

UpdateLang(name string, trans string)
==============================
Функция обновляет языковой ресурс в памяти. Используется в транзакциях, которые меняют языковые ресурсы.

* *name* - имя языкового ресурса.
* *trans* - ресурс с переводами.

.. code:: js

    UpdateLang($Name, $Trans)


********************************************************************************
Работа с системными таблицами
********************************************************************************

SysParamString(name string) string
==============================
Функция возвращает значение указанного системного параметра.

* *name* - имя параметра;

.. code:: js

    url = SysParamString(`blockchain_url`)

SysParamInt(name string) int
==============================
Функция возвращает значение указанного системного параметра в виде числа.

* *name* - имя параметра;

.. code:: js

    maxcol = SysParam(`max_columns`)

UpdateSysParam(name, value, conditions string)
==============================
Функция обновляет значение и условие системного параметра. Если значение или условие менять не нужно, то следует в соответствующем параметре указать пустую строку.

* *name* - имя параметра;
* *value* - новое значение параметра;
* *conditions* - новое условие изменения параметра;

.. code:: js

    UpdateSysParam(`fuel_rate`, `400000000000`, ``)

********************************************************************************
Работа с PostgreSQL
********************************************************************************

Функции не дают возможности напрямую отправлять запросы с select, update и т.д., но они позволяют использовать возможности и функции PostgrеSQL при получении значений и описания условий where в выборках. Это относится в том числе и к функциям по работе с датами и временем. Например, необходимо сравнить колонку *date_column* и текущее время. Если *date_column* имеет тип timestamp, то выражение будет следующим *date_column > now()*, а если *date_column* хранит время в Unix формате в виде числа, то тогда выражение будет *to_timestamp(date_column) > now()*. 

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'

Рассмотрим ситуацию, когда у нас есть значение в формате Unix и необходимо записать его в поле имеющее тип *timestamp*. В этом случае, при перечислении полей, перед именем данной колонки необходимо указать **timestamp**. 

.. code:: js

   DBInsert("mytable", "name,timestamp mytime", "John Dow", 146724678424 )

Если же вы имеете строковое значение времени и вам нужно записать его в поле с типом *timestamp*. В этом случае,  **timestamp** необходимо указать перед самим значением. 

.. code:: js

   DBInsert("mytable", "name,mytime", "John Dow", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + date )
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + $txtime )

