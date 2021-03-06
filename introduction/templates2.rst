################################################################################
Шаблонизатор страниц v2
################################################################################
********************************************************************************
Отличия от первой версии
********************************************************************************
В связи с переходом общения интерфейсной части только посредством API и выделения интерфейса на новую платформу (**React**) потребовалось изменить результат работы шаблонизатора. Также в новой версии нужно было учесть пожелания и замечания, который возникли при работе с первой версией шаблонизатора. 
В отличии от первой версии, вторая версия шаблонизатора выдает не готовую HTML страницу, а промежуточное дерево, которое представляет собой HTML тэги расположенные в виде дерева в соответствии с тем, как они описаны в шаблонизаторе. Для получения этого дерева для страниц и меню необходимо использовать команду content из API v2. Кроме этого, для тестирования, можно отправлять POST запрос в **api/v2/content** c параметром *template*, который содержит шаблон для обработки.
Следует заметить, что в новой версии мы отказались от двух типров функций () и {} и оставили только вызовы (), но кроме этого для всех вызовов сделали возможность указывать параметры по их именам. Об этом подробнее будет написано ниже. Сам список функций тоже будет переработан.

********************************************************************************
Общее описание шаблонизатора
********************************************************************************
Типы функций
==============================
Шаблоны страниц приложений создаются с помощью набора функций, который можно рассматривать как специализированный язык для создания интерфейсов приложений APLA. Функции можно разделить на несколько групп по типу выполняемых операций:

* получение значений из базы данных;
* оперирование с форматами и значениями переменных;
* представление данных в виде таблиц и диаграмм;
* построение форм с необходимым набором полей для ввода данных контрактов;
* вывод элементов навигации и вызова контрактов;
* создание элементов HTML разметки страницы – различных контейнеров с возможностью указания css классов;
* реализация условного вывода фрагментов шаблонов страниц; 
* создание многоуровневого меню.

Общее описание языка шаблонизатора
==============================
Язык построения шаблонов страниц по сути является функциональным языком, где вы вызывает функции в виде FuncName(parameters) и причем функции могут вкладываться друг в друга. Параметры можно не заключать в кавычки. Если параметр не нужен, то его можно никак не обозначать.

.. code:: js

      Text MyFunc(parameter number 1, parameter number 2) another text.
      MyFunc(parameter 1,,,parameter 4)

Если параметр содержит запятую, то тогда его нужно заключить в обратные или двойные кавычки. При этом, если параметр у функции возможен только один, то в нем можно использовать запятые не обрамляя его в кавычки.  Также кавычки нужно использовать если в параметре имеется непарная закрывающая скобка.

.. code:: js

      MyFunc("parameter number 1, the second part of first paremeter")
      MyFunc(`parameter number 1, the second part of first paremeter`)

Если вы заключили параметр в кавычки, но там также используются кавычки, то можно использовать разные кавычки или дублировать их в тексте.

.. code:: js

      MyFunc("parameter number 1, ""the second part of first"" paremeter")
      MyFunc(`parameter number 1, "the second part of first" paremeter`)

При описании функций каждый параметр имеет определенное имя. Вы можете вызывать функции и указывать параметры в том порядке как они описаны, а можете явно указывать только нужные параметры по их именам в любом порядке как **Имя_параметра: Значение_параметра**. Такой подход позволяет безболезненно добавлять новые параметры в функции без нарушения совместимости с текущими шаблонами. Например, пусть у нас есть функция, которая описана как **MyFunc(Class,Value,Body)**, то все эти вызовы будут корректными с точки зрения языка.

.. code:: js

      MyFunc(myclass, This is value, Div(divclass, This is paragraph.))
      MyFunc(Body: Div(divclass, This is paragraph.))
      MyFunc(myclass, Body: Div(divclass, This is paragraph.))
      MyFunc(Value: This is value, Body: 
           Div(divclass, This is paragraph.)
      )
      MyFunc(myclass, Value without Body)
      
Некоторые функции возвращают просто текст, некоторые создают HTML элемент (например, *Input*), а некоторые функцию создают HTML элемент с вложенными HTML элементами (*Div, P, Span*). В последнем случае для определения вложенных элементов используется параметр с предопределенным именем **Body**. Например, два *div*, вложенные в другой *div*, могут выглядеть так:

.. code:: js

      Div(Body:
         Div(class1, This is the first div.)
         Div(class2, This is the second div.)
      )
      
Для указания вложенных элементов, которые описываются в параметре *Body* можно использовать слежующее представление: **MyFunc(...){...}**, где в фигурных скобках указываются вложенные элементы. 

.. code:: js

      Div(){
         Div(class1){
            P(This is the first div.)
            Div(class2){
                Span(This is the second div.)
            }
         }
      }
      
Если идет подряд несколько одинаковых функции, то вместо имен второй и следующих можно ставить только точку. Например, следующие две строчки эквивалентны

.. code:: js

     Span(Item 1)Span(Item 2)Span(Item 3)
     Span(Item 1).(Item 2).(Item 3)
     
В языке можно присваивать переменные с помощью функции **SetVar**. Для подстановки значений переменных используется запись **#varname#**.

.. code:: js

     SetVar(name, My Name)
     Span(Your name: #name#)

Для подстановки языковых ресурсов экосистемы можно использовать запись **$langres$**, где *langres* имя языкового ресурса.
.. code:: js

     Span($yourname$: #name#)
     
Существуют следующие предопределенные переменные:

* **#key_id#** - идентификатор-кошелёк пользователя.
* **#ecosystem_id#** - идентификатор текущей экосистемы.

********************************************************************************
Возвращаемое значение
********************************************************************************

Результирующее JSON дерево состоит из объектов **Node** со следующими параметрами:

* *tag* String - имя HTML элемента или специального объекта.
* *attr* Object - объект состоящий из пар ключ - значение передаваемых атрибутов. Как правило сюда попадают все параметры, с именами в нижнем регистре. Например, **class, value, id**.
* *text* String - обычный текст. В этом случае, *tag* равен **text**. 
* *children* Array - массив вложенных объектов *Node*. Сюда попадают все элементы, описанные в параметре **Body**.     

********************************************************************************
Функции
********************************************************************************

Address(Wallet)
==========================
Функция возвращает адрес кошелька в формате 1234-5678-...-7990 по числовому значению адреса; если адрес не указан, то в качестве аргумента принимается значение адреса текущего пользователя. 

.. code:: js

      Span(Your wallet: Address(#wallet#))

AddToolButton(Title, Icon, Page, PageParams)
==========================
Добавляет кнопку в панель кнопок. Создает элемент **addtoolbutton**. 

* *Title* - заголовок кнопки.
* *Icon* - иконка для кнопки.
* *Page* - название страницы для перехода.
* *PageParams* - параметры для перехода на страницу.

.. code:: js

      AddToolButton(Help, help, help_page)

And(parameters)
==========================
Функция возвращает результат выполнения логической операции **И** со всеми перечисленными в скобках через запятую параметрами. Значение параметра принимается как **false**, если он равен пустой строке (""), 0 или *false*. Во всех остальных случаях значение параметра считается **true**. Соответственно функция возвращает 1 в случае истины и в противном случае 0. Элемент с именем **and** создается только при запросе дерева для редактирования. 

.. code:: js

      If(And(#myval1#,#myval2#), Span(OK))


Button(Body, Page, Class, Contract, Params, PageParams) [.Alert(Text,ConfirmButton,CancelButton,Icon)] [.Style(Style)]
==========================
Создает HTML элемент **button**. Этот элемент должен создавать кнопку, которая будет отправлять на выполнение указанный контракт.

* *Body* - дочерний текст или элементы.
* *Page* - название страницы для перехода.
* *Class* - классы для данной кнопки.
* *Contract* - Имя вызываемого контракта.
* *Params* - список передаваемых в контракт значений. По умолчанию, значения параметров контракта (секция data) берутся из HTML элементов (скажем, полей формы) с одноименными идентификаторами (id). Если идентификаторы элементов отличаются от названий параметров контракта, то используется присваивание в формате *contractField1=idname1, contractField2=idname2*. Данный параметр возвращается в *attr* в виде объекта *{field1: idname1, field2: idname2}*.
**ПРИМЕЧАНИЕ** В случае, когда Inputs не указан, то реализация на фронтенде может брать все контролы в form, где находится кнопка или самостоятельно запрашивать из API список параметров и брать значения *input* c такими же идентификаторами.
* *PageParams* - параметры для перехода на страницу.

**Alert** - указывается для вывода сообщений.

* *Text* - текст сообщения.
* *ConfirmButton* - текст кнопки подтверждения.
* *CancelButton* - текст кнопки отмены.
* *Icon* - иконка.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      Button(Submit, default_page, mybtn_class).Alert(Alert message)
      Button(Contract: MyContract, Body:My Contract, Class: myclass, Params:"Name=myid,Id=i10,Value")

CmpTime(Time1, Time2) 
==============================
Функция сравнивает два значения времени в одинаковом формате (желательно стандартном - YYYY-MM-DD HH:MM:SS, но можно и в произвольном при условии соблюдения последовательности от годов к секундам, например, YYYYMMDD). Возвращает: 

* **-1** - Time1 < Time2, 
* **0** - Time1 = Time2, 
* **1** - Time1 > Time2.

.. code:: js

     If(CmpTime(#time1#, #time2#)<0){...}

 
Data(Source,Columns,Data) [.Custom(Column,Body)]
==========================
Создает элемент **data** и заполняет его указанными данными. В *attr* возвращаются три массива - *columns* c именами колонок, *types*, где для обычной колонки указан *text*, а для Custom колонок указан тип *tags* и массив *data* с записями. Последовательность в именах колонок соответствует последовательности значений в записях в *data*.

* *Source* - имя источника данных. Вы можете указать любое имя, которое потом будет указываться в других командах (например. *Table*) как источник данных.
* *Columns* - список колонок. Данные должны быть идти в таком же порядке. 
* *Data* - Данные по одной записи на строку с разделением на колонки через запятую. Можно заключать значения в двойные кавычки. Если нужно вставить кавычки, то их нужно удвоить.

* **Custom** - позволяет определять вычисляемые столбцы для данных. Например, можно указывать шаблон для кнопок и дополнительного оформления. Можно определять несколько таких вычисляемых столбцов. Как правило, такие поля определяются для вывода в *Table* и других командах, которые используют полученные данные.

  * *Column* - имя колонки. Нужно определить любое уникальное имя.
  * *Body* - укажите шаблон. В нем можно получать значения из других колонок в данной записи с помощью **#columnname#**.

.. code:: js

    Data(mysrc,"id,name"){
	"1",John Silver
	2,"Mark, Smith"
	3,"Unknown ""Person"""
     }

DateTime(DateTime, Format) 
==============================
Функция выводит значение даты и времени в заданном формате. 
 
*  *DateTime* - время в стандартном формате 2006-01-02T15:04:05.
*  *Format* -  шаблон формата : YY короткий год, YYYY полный год, MM - месяц, DD - день, HH - часы, MM - минуты, SS – секунды, например, YY/MM/DD HH:MM. Если формат не указан, то будет использовано значение параметра  *timeformat* определенное в таблице *languages*, если его нет, то YYYY-MM-DD HH:MI:SS.

.. code:: js

    DateTime(2017-11-07T17:51:08)
    DateTime(#mytime#,HH:MI DD.MM.YYYY)

DBFind(Name, Source) [.Columns(columns)] [.Where(conditions)] [.WhereId(id)] [.Order(name)] [.Limit(limit)] [.Offset(offset)] [.Ecosystem(id)] [.Custom(Column,Body)] [.Vars(Prefix)]
==========================
Создает элемент **dbfind** и возвращает данные из таблицы базы данных. В *attr* возвращаются три массива - *columns* c именами колонок, *types*, где для обычной колонки указан *text*, а для Custom колонок указан тип *tags* и массив *data* с записями. Последовательность в именах колонок соответствует последовательности значений в записях в *data*.

* *Name* - имя таблицы.
* *Source* - имя источника данных. Вы можете указать любое имя, которое потом будет указываться в других командах (например. *Table*) как источник данных.

* **Columns** - список возвращаемых колонок. Если не указано, то возвратятся все колонки. 
* **Where** - условие поиска. Например, *.Where(name = '#myval#')*
* **WhereId** - условие поиска по идентификатору. Достаточно указать значение идентификатора.  Например, *.WhereId(1)*
* **Order** - поле, по которому нужно отсортировать. 
* **Limit** - количество возвращаемыхх записей. По умолчанию, 25. Максимально возможно количество - 250.
* **Offset** - смещение возвращаемых записей.
* **Ecosystem** - идентификатор экосистемы. По умолчанию, берутся данные из таблицы в текущей экосистеме.
* **Custom** - позволяет определять вычисляемые столбцы для данных. Например, можно указывать шаблон для кнопок и дополнительного оформления. Можно определять несколько таких вычисляемых столбцов. Как правило, такие поля определяются для вывода в *Table* и других командах, которые используют полученные данные.

  * *Column* - имя колонки. Нужно определить любое уникальное имя.
  * *Body* - укажите шаблон. В нем можно получать значения из других колонок в данной записи с помощью **#columnname#**.

* **Vars** - Функция формирует множество переменных со значениями из записи таблицы базы данных, полученной по данному запросу. При указании этой функции, параметр *Limit* автоматически становится равным 1 и возвращается только одна запись.

* *Prefix* - префикс, используемый для образования имен переменных, в которые записываются значения полученной записи: переменные имеют вид *#prefix_id#, #prefix_name#*, где после знака подчеркивания указывается имя колонки таблицы.

.. code:: js

    DBFind(parameters,myparam)
    DBFind(parameters,myparam).Columns(name,value).Where(name='money')
    DBFind(parameters,myparam).Custom(myid){Strong(#id#)}.Custom(myname){
       Strong(Em(#name#))Div(myclass, #company#)
    }

Div(Class, Body) [.Style(Style)]
==========================
Создает HTML элемент **div**.

* *Class* - классы для данного *div*.
* *Body* - дочерние элементы.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      Div(class1 class2, This is a paragraph.)

EcosysParam(Name, Index, Source) 
==============================
Функция выводит значение параметра из таблицы parameters текущей экосистемы. Если есть языковый ресурс c полученным именем, то подставится его значение.
 
* *Name* - имя значения;
* *Index* - вы можете указать порядковый номер значения c 1, если параметр является список с элементами раззделенными запятыми. например, *gender = male,female*, тогда EcosysParam(gender, 2) возвратит *female*.  
* *Source* - вы можете получить значения параметра разделенными запятыми в виде объекта *data*. В дальнейшем этот список можно указывать в качестве источника данных как для *Table*, так и для *Select*. Если вы указывайте этот параметр, то функция не будет возвращать значение, а возвратит список в виде объекта *data*.

.. code:: js

     Address(EcosysParam(founder_account))
     EcosysParam(gender, Source: mygender)
 
Em(Body, Class)
==========================
Создает HTML элемент **em**.

* *Body* - дочерний текст или элементы.
* *Class* - классы для данного *em*.

.. code:: js

      This is an Em(important news).

ForList(Source, Body)
==========================
Выводит блок *Body* для каждого элемента из источника данных *Source*. Создает элемент **forlist**.

* *Source* - источник данных. Например от *DBFind* или *Data*.
* *Body* - шаблон, который необходимо вывести для каждого элемента.

.. code:: js

      ForList(mysrc){Span(#name#)}

Form(Class, Body) [.Style(Style)]
==========================
Создает HTML элемент **form**.

* *Class* - классы для данного *form*.
* *Body* - дочерние элементы.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      Form(class1 class2, Input(myid))
      
GetVar(Name)
==========================
Функция возвращает значение указанной переменной, если она существует, и возвращает пустую строку, если переменная с данным именем не определена. Элемент с именем **getvar** создается только при запросе дерева для редактирования. Отличие *GetVar(varname)* от использования *#varname#* состоит в том, что если *varname* не существует, то *GetVar* возвратит пустую строку, а *#varname#* так и останется.

* *Name* - имя переменной.

.. code:: js

     If(GetVar(name)){#name#}.Else{Name is unknown}
      
If(Condition){ Body } [.ElseIf(Condition){ Body }] [.Else{ Body }]
==========================
Условный оператор. Возвращаются дочерние элементы первого *If* или *ElseIf* у которого выполнено условие *Condition*. В противном случае, возвращаются дочерние элементы *Else*, если он присутствует.

* *Condition* - Условие. Считается не выполненным если равно *пустой строке*, *0* или *false*. В остальных случаях, условие считается истинным.
* *Body* - дочерние элементы.

.. code:: js

      If(#value#){
         Span(Value)
      }.ElseIf(#value2#){Span(Value 2)
      }.ElseIf(#value3#){Span(Value 3)}.Else{
         Span(Nothing)
      }

Image(Src,Alt,Class) [.Style(Style)]
==============================
Создает HTML элемент **image**.
 
* *Src* - источник изображения, файл или *data:...*;
* *Alt* - альтернативный текст для изображения; 
* *Сlass* - список классов.

.. code:: js

    Image(\images\myphoto.jpg)

ImageInput(Name, Width, Ratio, Format) 
==============================
Создает элемент **imageinput** для загрузки картинок. По желанию в третьем параметре можно указать либо высоту картинки, либо отношение сторон в виде *1/2*, *2/1*, *3/4* и т.п. По умолчанию берется ширина в 100 пикселей и отношение сторон *1/1*.

* *Name* - имя элемента;
* *Width* - ширина вырезаемого изображения;
* *Ratio* - отношение сторон (ширины к высоте) или высота картинки.
* *Format* - формат загружаемой картинки.

.. code:: js

   ImageInput(avatar, 100, 2/1)

Include(Name)
==========================
Команда вставляет шаблон с именем *Name* из таблицы *blocks*. При вставке происходит разбор шаблона и происходит вставка разобранных элементов.

* *Name* - Имя вставляемого шаблона из таблицы *blocks*.

.. code:: js

      Div(myclass, Include(mywidget))

Input(Name,Class,Placeholder,Type,Value) [.Validate(validation parameters)] [.Style(Style)]
==========================
Создает HTML элемент **input**.

* *Name* - имя элемента.
* *Class* - классы для данного *input*.
* *Placeholder* - *placeholder* для данного *input*.
* *Type* - типа для данного *input*.
* *Value* - значение элемента.

**Validate** - параметры валидации.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      Input(Name: name, Type: text, Placeholder: Enter your name)
      Input(Name: num, Type: text).Validate(minLength: 6, maxLength: 20)

InputErr(Name,validation errors)]
==========================
Создает элемент **inputerr** c текстами для ошибок валидации.

* *Name* - имя соответствующего элемента **Input**.

.. code:: js

      InputErr(Name: name, 
          minLength: Value is too short, 
          maxLength: The length of the value must be less than 20 characters)

Label(Body, Class, For) [.Style(Style)]
==========================
Создает HTML элемент **label**.

* *Body* - дочерний текст или элементы.
* *Class* - классы для данного *label*.
* *For* - значение *for* для данного *label*.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      Label(The first item).
      
LangRes(Name, Lang)
==========================
Возвращает указанный языковой ресурс. В случае запроса дерева для редактирования возвращается элемент **langres**. Кроме этой команды можно использовать записи вида **$langres$**, которые будут заменяться на значения указанного языкового ресурса.

* *Name* - имя языкового ресурса.
* *Lang* - по умолчанию, возвращается язык который определен в запросе в *Accept-Language*. При желании вы можете указать свой двухсивольный идентификатор языка.

.. code:: js

      LangRes(name)
      LangRes(myres, fr)

LinkPage(Body, Page, Class, PageParams) [.Style(Style)]
==========================
Создает элемент **linkpage** для ссылки на страницу. 

* *Body* - дочерний текст или элементы.
* *Page* - название страницы для перехода.
* *Class* - классы для данной кнопки.
* *PageParams* - параметры для перехода на страницу.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      LinkPage(My Page, default_page, mybtn_class)

MenuGroup(Title, Body, Icon) 
==============================
Функция формирует в меню вложенное подменю и возвращает элемент **menugroup**. 

* *Title* - имя пункта меню.
* *Body* - дочерние элементы подменю;
* *Icon* - иконка.

.. code:: js

      MenuGroup(My Menu){
          MenuItem(Interface, sys-interface)
          MenuItem(Dahsboard, dashboard_default)
      }

MenuItem(Title, Page, Params, Icon) 
==============================
Служит для создания пункта меню и возвращает элемент **menuitem**. 

* *Title* - имя пункта меню;
* *Page* - имя страницы перехода.;
* *Params* - параметры, передаваемые странице в формате *var:value* через запятую.
* *Icon* - иконка.

.. code:: js

       MenuItem(Interface, interface)

Now(Format, Interval) 
==============================
Функция возвращает текущее время в указанном формате, по умолчанию выводится  в UNIX-формате (число секунд с 1970 года). Если в качестве формата указано *datetime*, то дата и время выводится в виде YYYY-MM-DD HH:MI:SS. Во втором параметре можно указать интервал, например, *+5 days*.

* *Format* - формат вывода с комбинацией YYYY, MM, DD, HH, MI, SS или *datetime*;
* *Interval* - дополнтельный сдвиг времени назад или вперед;

.. code:: js

       Now()
       Now(DD.MM.YYYY HH:MM)
       Now(datetime,-3 hours)

Or(parameters)
==========================
Функция возвращает результат выполнения логической операции **ИЛИ** со всеми перечисленными в скобках через запятую параметрами. Значение параметра принимается как **false**, если он равен пустой строке (""), 0 или *false*. Во всех остальных случаях значение параметра считается **true**. Соответственно функция возвращает 1 в случае истины и в противном случае 0. Элемент с именем **or** создается только при запросе дерева для редактирования. 

.. code:: js

      If(Or(#myval1#,#myval2#), Span(OK))

P(Body, Class) [.Style(Style)]
==========================
Создает HTML элемент **p**.

* *Body* - дочерний текст или элементы.
* *Class* - классы для данного *p*.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      P(This is the first line.
        This is the second line.)


RadioGroup(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
==========================
Команда служит для создания набора радиокнопок. Создает элемент **radiogroup**.

* *Name* - имя элемента.
* *Source* - имя источника данных. Например, из команды *DBFind* или *Data*.
* *NameColumn* - Имя колонки, из которой будeт браться текст для элементов.
* *ValueColumn* - Имя колонки, из которой будут браться значения для элементов. В этом параметре нельзя указывать имена колонок созданных через Custom.
* *Value* - Значение по умолчанию.
* *Class* - Классы для элемента.

**Validate** - параметры валидации.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      DBFind(mytable, mysrc)
      RadioGroup(mysrc, name)

Select(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
==========================
Создает HTML элемент **select**.

* *Name* - имя элемента.
* *Source* - имя источника данных. Например, из команды *DBFind* или *Data*.
* *NameColumn* - Имя колонки, из которой будeт браться текст для элементов.
* *ValueColumn* - Имя колонки, из которой будут браться значения для элементов. В этом параметре нельзя указывать имена колонок созданных через Custom.
* *Value* - Значение по умолчанию.
* *Class* - Классы для элемента.

**Validate** - параметры валидации.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      DBFind(mytable, mysrc)
      Select(mysrc, name)

SetTitle(Title)
==========================
Устанавливает заголовок страницы. Создается элемент с именем **settitle**.

* *Title* - заголовок страницы.

.. code:: js

     SetTitle(My page)

SetVar(Name, Value)
==========================
Присваивает переменной с именем *Name* значение *Value*. Элемент с именем **setvar** создается только при запросе дерева для редактирования.

* *Name* - имя переменной.
* *Value* - значение переменной, может содержать ссылку на другие переменные.

.. code:: js

     SetVar(name, John Smith).(out, I am #name#)
     Span(#out#)

Span(Body, Class) [.Style(Style)]
==========================
Создает HTML элемент **span**.

* *Body* - дочерний текст или элементы.
* *Class* - классы для данного *span*.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      This is Span(the first item, myclass1).

Strong(Body, Class)
==========================
Создает HTML элемент **strong**.

* *Body* - дочерний текст или элементы.
* *Class* - классы для данного *strong*.

.. code:: js

      This is Strong(the first item, myclass1).

Table(Source, Columns) [.Style(Style)]
==========================
Создает HTML элемент **table**.

* *Source* - имя источника данных. Например, из команды *DBFind*.
* *Columns* - Заголовки и соответствующие имена колонок в виде **Title1=column1,Title2=column2**.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      DBFind(mytable, mysrc)
      Table(mysrc,"ID=id,Name=name")
