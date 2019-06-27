# xsl:variable

Для объявления переменных в XSLT служит элемент **`xsl:variable`**, который может как присутствовать в теле шаблона, так и быть элементом верхнего уровня.

## Синтаксис

### XSLT 1.0

```xml
<xsl:variable
    name = "qname"
    select = "expression">
    <!-- Содержимое: шаблон -->
</xsl:variable>
```

Атрибуты:

- **`name`** — **обязательный** атрибут, задает имя переменной
- `select` — _необязательный_ атрибут, выражение

### XSLT 2.0

```xml
<xsl:variable
    name = qname
    select? = expression
    as? = sequence-type>
    <!-- Content: sequence-constructor -->
</xsl:variable>
```

### XSLT 3.0

```xml
<xsl:variable
    name = eqname
    select? = expression
    as? = sequence-type
    static? = boolean
    visibility? = "public" | "private" | "final" | "abstract" >
    <!-- Content: sequence-constructor -->
</xsl:variable>
```

## Описание и примеры

Для объявления переменных в XSLT служит элемент `xsl:variable`, который может как присутствовать в теле шаблона, так и быть элементом верхнего уровня. Элемент `xsl:variable` связывает имя, указанное в обязательном атрибуте `name`, со значением выражения, указанного в атрибуте `select` или с деревом, которое является результатом выполнения шаблона, содержащегося в этом элементе. В том случае, если объявление переменной было произведено элементом верхнего уровня, переменная называется глобальной переменной. Переменные, определенные элементами `xsl:variable` в шаблонах (то есть не на верхнем уровне) называются локальными переменными.

Таким образом, объявление переменной в XSLT происходит всего в два шага:

- сначала вычисляется значение присваиваемого выражения;
- затем полученное значение связывается с указанным именем.

Значение присваиваемого выражения вычисляется в зависимости от того, как был определен элемент `xsl:variable`:

- если в элементе `xsl:variable` определен атрибут `select`, то значением присваиваемого выражения будет результат вычисления выражения, указанного в этом атрибуте;
- если атрибут `select` не определен, но сам элемент `xsl:variable` имеет дочерние узлы (иными словами, содержит шаблон), значением определяемой переменной будет результирующий фрагмент дерева, полученный в результате выполнения содержимого `xsl:variable`;
- если атрибут `select` не определен и при этом сам элемент `xsl:variable` пуст, значением параметра по умолчанию будет пустая строка.

Использовать значения, присвоенные переменным при инициализации, можно, указывая впереди имени переменной символ "`$`", например для переменной `x` — `$x`. В XPath-выражениях синтаксис обращения к переменным соответствует продукции VariableReference.

Имя переменной соответствует синтаксическому правилу QName, иными словами, оно может иметь вид имя или префикс:имя. Как правило, имена переменным даются без префиксов, однако в том случае, если префикс все же указан, переменная ассоциирует с некоторым объектом не простое, а расширенное имя. Соответственно, обращение к объекту должно будет производиться также посредством расширенного имени.

### Область видимости переменных

Каждая из переменных имеет собственную область видимости (англ. scope) — область, в которой может быть использовано ее значение. Область видимости определяется следующим образом.

- Областью видимости глобальной переменной является все преобразование, то есть значение переменной, объявленной элементом верхнего уровня, может быть использовано в преобразовании где угодно. К такой переменной можно обращаться даже до ее объявления, единственным ограничением является то, что переменная не должна определяться через собственное значение — явно или неявно.
- Локальную переменную можно использовать только после ее объявления и только в том же родительском элементе, которому принадлежит объявляющий элемент `xsl:variable`. В терминах XPath область видимости локальной переменной будет определяться выражением

```
following-sibling:node()/descendant-or-self:node()
```

Для того чтобы до конца прояснить ситуацию, приведем несколько примеров.

Предположим, что мы определяем переменную с именем `ID` и значением `4` следующим образом:

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    ...
    <xsl:variable name="ID" select="4"/>
    ...
</xsl:stylesheet>
```

Несложно видеть, что здесь мы определили глобальную переменную, а значит, ее значение можно использовать в преобразовании в любом месте. Например, мы можем определить через нее другие глобальные переменные, либо использовать в шаблоне:

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    ...
    <xsl:variable name="leaf" select="//item[@id=$ID]"/>
    <xsl:variable name="ID" select="4"/>
    <xsl:variable name="path" select="$leaf/ancestor-or-self::item"/>
    ...
</xsl:stylesheet>
```

Причем, как уже было сказано, глобальная переменная может быть использована и до объявления: в нашем случае переменная `leaf` определяется через переменную `ID`, а `path` — через `leaf`. Конечно же, не следует забывать и то правило, что переменные не могут объявляться посредством самих себя, явно или неявно. Очевидно, что объявление:

```xml
<xsl:variable name="ID" select="$ID - 1"/>
```

было бы некорректным ввиду явного использования переменной при собственном определении. Точно так же были бы некорректны определения:

```xml
<xsl:variable name="ID" select="$id - 1"/>
<xsl:variable name="id" select="$ID + 1"/>
```

поскольку переменная `ID` определяется через переменную `id`, которая определяется через переменную `ID` и так до бесконечности.

Дела с локальными переменными обстоят чуть-чуть сложнее. Для того чтобы объяснить, что же такое область видимости, обратимся к следующему преобразованию.

Листинг 5.22. Преобразование, использующее переменные i, j, k и gt

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="... ">
    <xsl:template match="/">
        <xsl:variable name="i" select="2"/>
        <xsl:variable name="j" select="$i - 1"/>
        <xsl:if test="$i > $j">
            <xsl:variable name="k">
                <xsl:value-of select="$i"/>
                <xsl:value-of select="$gt"/>
                <xsl:value-of select="$j"/>
            </xsl:variable>
            <result>
                <xsl:copy-of select="$k"/>
            </result>
        </xsl:if>
    </xsl:template>
    <xsl:variable name="gt">
        is greater than
    </xsl:variable>
</xsl:stylesheet>
```

В этом преобразовании определены три локальные переменные — `i`, `j` и `k` и одна глобальная переменная — `gt`. На следующих четырех листингах мы выделим серой заливкой область видимости переменной (то есть область, где ее можно использовать), а само определение переменной отметим полужирным шрифтом.

Листинг 5.23. Области видимости переменных `i`, `j`, `k` и `gt`

Область видимости переменной i

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="... ">
    <xsl:template match="/">
        <xsl:variable name="i" select="2"/>
        <!-- Область видимости переменной i -->
        <xsl:variable name="j" select="$i - 1"/>
        <xsl:if test="$i > $j">
            <xsl:variable name="k">
                <xsl:value-of select="$i"/>
                <xsl:value-of select="$gt"/>
                <xsl:value-of select="$j"/>
            </xsl:variable>
            <result>
                <xsl:copy-of select="$k"/>
            </result>
        </xsl:if>
        <!-- /Область видимости переменной i -->
    </xsl:template>
    <xsl:variable name="gt">
        is greater than
    </xsl:variable>
</xsl:stylesheet>
```

Область видимости переменной j

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="... ">
    <xsl:template match="/">
        <xsl:variable name="i" select="2"/>
        <xsl:variable name="j" select="$i - 1"/>
        <!-- Область видимости переменной j -->
        <xsl:if test="$i > $j">
            <xsl:variable name="k">
                <xsl:value-of select="$i"/>
                <xsl:value-of select="$gt"/>
                <xsl:value-of select="$j"/>
            </xsl:variable>
            <result>
                <xsl:copy-of select="$k"/>
            </result>
        </xsl:if>
        <!-- /Область видимости переменной j -->
    </xsl:template>
    <xsl:variable name="gt">
        is greater than
    </xsl:variable>
</xsl:stylesheet>
```

Область видимости переменной k

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="... ">
    <xsl:template match="/">
        <xsl:variable name="i" select="2"/>
        <xsl:variable name="j" select="$i - 1"/>
        <xsl:if test="$i > $j">
            <xsl:variable name="k">
                <xsl:value-of select="$i"/>
                <xsl:value-of select="$gt"/>
                <xsl:value-of select="$j"/>
            </xsl:variable>
            <!-- Область видимости переменной k -->
            <result>
                <xsl:copy-of select="$k"/>
            </result>
            <!-- /Область видимости переменной k -->
        </xsl:if>
    </xsl:template>
    <xsl:variable name="gt">
        is greater than
    </xsl:variable>
</xsl:stylesheet>
```

Область видимости переменной gt

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="... ">
    <!-- Область видимости переменной gt -->
    <xsl:template match="/">
        <xsl:variable name="i" select="2"/>
        <xsl:variable name="j" select="$i - 1"/>
        <xsl:if test="$i > $j">
            <xsl:variable name="k">
                <xsl:value-of select="$i"/>
                <xsl:value-of select="$gt"/>
                <xsl:value-of select="$j"/>
            </xsl:variable>
            <result>
                <xsl:copy-of select="$k"/>
            </result>
        </xsl:if>
    </xsl:template>
    <xsl:variable name="gt">
        is greater than
    </xsl:variable>
    <!-- /Область видимости переменной gt -->
</xsl:stylesheet>
```

В XSLT действует то же правило, что и во многих других языках программирования: нельзя дважды определять переменную с один и тем же именем. Однако и тут есть свои особенности.

- Имена двух глобальных переменных могут совпадать в том и только том случае, когда они имеют разный порядок импорта. Например, если переменные с одинаковыми именами определены в разных преобразованиях, одно из них может быть импортировано. В этом случае переменная будет иметь значение, которое задано элементом `xsl:variable` со старшим порядком импорта.
- Допускается совпадение имен локальной и глобальной переменных — в этом случае в области видимости локальной переменной будет использоваться локальное значение, в области видимости глобальной (но не локальной) — глобальное значение. Иными словами, локальные переменные "закрывают" значения глобальных.
- Две локальные переменные могут иметь совпадающие имена в том и только том случае, если их области видимости не пересекаются.

Первое правило мы уже упоминали, когда разбирали порядок импорта: тогда мы сказали, что переменные со старшим порядком импорта переопределяют переменные с младшим порядком импорта. Это довольно важное обстоятельство, поскольку оно добавляет некоторые интересные возможности, но при этом также может породить скрытые ошибки.

### Пример

Предположим, что в следующем преобразовании в шаблоне с именем `choice` мы генерируем два элемента `input`.

Листинг 5.24. Преобразование `en.xsl`

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:variable name="submit" select="'Submit'"/>
    <xsl:variable name="reset" select="'Reset'"/>
    <xsl:template name="choice">
        <input type="button" value="{$submit}"/>
        <xsl:text> </xsl: text>
        <input type="reset" value="{$reset}"/>
    </xsl:template>
    <xsl:template match="/">
        <xsl:call-template name="choice"/>
    </xsl:template>
</xsl:stylesheet>
```

Результатом этого преобразования будет следующий фрагмент:

```xml
<input type="button" value="Submit"/>
<input type="reset" value="Reset"/>
```

Для того чтобы перевести надписи на этих кнопках на другой язык достаточно просто переопределить переменные. Например, результатом выполнения следующего шаблона.

Листинг 5.25. Преобразование `de.xsl`

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:import href="en.xsl"/>
    <xsl:variable name="submit" select="'Senden'"/>
    <xsl:variable name="reset" select="'Loeschen'"/>
</xsl:stylesheet>
```

будет тот же фрагмент, но уже на немецком языке:

```xml
<input type="button" value="Senden"/>
<input type="reset" value="Loeschen"/>
```

С другой стороны, переопределение переменных может быть и опасным: в случае, если отдельные модули разрабатывались независимо, совпадение имен глобальных переменных может привести к ошибкам. Для того чтобы такое было в принципе исключено, имя переменной следует объявлять в определенном пространстве имен, например:

```xml
<xsl:variable name="app:href" select="..."/>
<xsl:variable name="db:href" select="..."/>
```

В том случае, если префиксы `app` и `db` (которые, конечно же, должны быть объявлены) будут указывать на разные пространства имен, никакого конфликта между этими двумя переменными не будет.

Возвращаясь к теме совпадений имен переменных, продемонстрируем "скрытие" локальной переменной значения глобальной:

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:variable name="i" select="1"/>
    <xsl:template match="/">
        <xsl:text>i equals </xsl:text>
        <xsl:value-of select="$i"/>
        <xsl:text> </xsl:text>
        <xsl:variable name="i" select="$i + 1"/>
        <xsl:text>i equals </xsl:text>
        <xsl:value-of select="$i"/>
    </xsl:template>
</xsl:stylesheet>
```

Результатом выполнения этого шаблона будет:

```
    i equals 1
    i equals 2
```

Как можно видеть, объявление локальной переменной `i` "скрыло" значение глобальной переменной `i`. Более того, это преобразование любопытно еще и тем, что локальная переменная объявляется через глобальную — такое тоже допускается.

Рассмотрим теперь случай двух локальных переменных. Попробуем объявить две локальные переменные — одну за другой в следующем шаблоне:

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <xsl:variable name="i" select="1"/>
        <xsl:text>i equals </xsl:text>
        <xsl:value-of select="$i"/>
        <xsl:text> </xsl:text>
        <xsl:variable name="i" select="2"/>
        <xsl:text>i equals </xsl:text>
        <xsl:value-of select="$i"/>
    </xsl:template>
</xsl:stylesheet>
```

В тексте этого шаблона мы выделили две области видимости двух переменных: серой заливкой — область видимости первой и полужирным шрифтом — область действия второй переменной. Вследствие того, что эти области пересекаются, шаблон будет некорректным — процессор выдаст сообщение об ошибке вида:

```
    Failed to compile style sheet
    At xsl:variable on line 9 of file stylesheet.xsl:
    Variable is already declared in this template
```

Приведем теперь другое преобразование, в котором элементы `xsl:variable` принадлежат двум братским элементам:

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <p>
            <xsl:variable name="i" select="1"/>
            <xsl:text>i equals </xsl:text>
            <xsl:value-of select="$i"/>
        </p>
        <p>
            <xsl:variable name="i" select="2"/>
            <xsl:text>i equals </xsl:text>
            <xsl:value-of select="$i"/>
        </p>
    </xsl:template>
</xsl:stylesheet>
```

В этом случае никакого пересечения областей видимости нет, поэтому и ошибкой наличие в одном шаблоне двух локальных переменных с одинаковыми именами тоже не будет. Приведенное выше преобразование возвратит результат вида:

```xml
<p>
    i equals 1
</p>
<p>
    i equals 2
</p>
```

### Использование переменных

Как правило, первой реакцией на известие о том, что переменные в XSLT нельзя изменять является реплика: "Да зачем они вообще тогда нужны?!".

Претензия со стороны процедурного программирования вполне серьезна и обоснованна — изменяемые переменные являются большим подспорьем в сложных многошаговых вычислениях. Тем не менее, переменные, даже в том виде, в каком они присутствуют в XSLT, являются очень важным и полезным элементом языка. Ниже мы перечислим наиболее типичные случаи использования переменных.

- Переменные могут содержать значения выражений, которые много раз используются в преобразовании. Это избавит процессор от необходимости пересчитывать выражение каждый раз по-новому.
- Переменной может присваиваться результат преобразования, что позволяет манипулировать уже сгенерированными частями документа.
- Переменные могут использоваться для более прозрачного доступа к внешним документам.

### Примеры

Первый случай использования совершенно очевиден: если в преобразовании многократно используется какое-либо сложное для вычисления или просто громоздкое для записи выражение, переменная может служить для сохранения единожды вычисленного результата. Например, если мы много раз обращаемся ко множеству ссылок данного документа посредством выражения вида:

```
    //a[@href]
```

гораздо удобней и экономней с точки зрения вычислительных ресурсов объявить переменную вида

```xml
<xsl:variable name="a" select="//a[@href]"/>
```

и использовать ее в преобразовании как `$a`. Фильтрующие выражения языка XPath (продукция FilterExpr) позволяют затем обращаться к узлам выбранного множества так же, как если бы мы работали с изначальным выражением. Например, `$a[1]` будет эквивалентно `//a[@href][1]`, а `$a/@href` — выражению `//a[@href]/@href`. При этом при обращении к `$a` процессор не будет заново искать все элементы `a` с атрибутом `href`, что, по всей вероятности, положительно скажется на производительности.

Другой иллюстрацией этому же случаю использования переменной может быть следующая ситуация: в некоторых случаях выражения могут быть просты для вычисления, но слишком громоздки для записи. Гораздо элегантней один раз вычислить это громоздкое выражение, сохранить его в переменной и затем обращаться по короткому имени. Например, следующий элемент `xsl:variable` вычисляет и сохраняет в переменной `gv` длинную и громоздкую ссылку:

```xml
<xsl:variable name="gv"
    select="concat('http://host.com:8080/GeoView/GeoView.jsp?',
    'Language=en&',
    'SearchText=Select&',
    'SearchTarget=mainFrame&',
    'SearchURL=http://host.com:8080/servlet/upload')"/>
```

После такого определения применение этой ссылки в преобразовании становится удобным и лаконичным:

```xml
<a href="{$gv}" target="_blank">Launch GeoBrowser</a>
```

Второй типовой случай использования переменных также заметно облегчает создание выходящего дерева, делая этот процесс гибким и легко конфигурируемым. Примером формирования HTML-документа при помощи такого подхода может быть следующий шаблон:

```xml
<xsl:template match="/">
    <html>
        <xsl:copy-of select="$head"/>
        <xsl:copy-of select="$body"/>
    </html>
</xsl:template>
```

Достоинство этого подхода в том, что переменные, содержащие фрагменты деревьев как бы становятся модулями, блоками, из которых в итоге собирается результирующий документ.

Более практичным применением возможности переменных содержать фрагменты деревьев является условное присвоение переменной значения. Представим себе следующий алгоритм:

```
	если
		условие1
	то
		присвоить переменной1 значение 1
	иначе
		присвоить переменной1 значение2
```

Для процедурного языка с изменяемыми переменными это не проблема. На Java такой код выглядел бы элементарно:

```
    переменная1 = условие1 ? значение1 : значение2;
```

или чуть в более длинном варианте:

```
    if (условие1)
        переменная1 = значение1;
    else
        переменная1 = значение2;
```

Однако если бы в XSLT мы написали что-нибудь наподобие:

```xml
<xsl:choose>
    <xsl:when test="условие1">
        <xsl:variable name="переменная1" select="значение1"/>
    </xsl:when>
    <xsl:otherwise>
        <xsl:variable name="переменная1" select="значение2"/>
    </xsl:otherwise>
</xsl:choose>
```

то требуемого результата все равно не достигли бы по той простой причине, что определенные нами переменные были бы доступны только в своей области видимости, которая заканчивается закрывающими тегами элементов `xsl:when` и `xsl:otherwise`. Правильный шаблон для решения этой задачи выглядит следующим образом:

```xml
<xsl:variable name="переменная1">
    <xsl:choose>
        <xsl:when test="условие1">
            <xsl:copy-of select="значение1"/>
        </xsl:when>
        <xsl:otherwise>
            <xsl:copy-of select="значение2"/>
        </xsl:otherwise>
    </xsl:choose>
<xsl:variable>
```

Конечно, это не точно то же самое — на самом деле мы получаем не значение, а дерево, содержащее это значение, но для строковых и численных значений особой разницы нет: дерево будет вести себя точно так же, как число или строка. Для булевых значений и множеств узлов приходится изыскивать другие методы. Булевое значение можно выразить через условие, например:

```xml
<xsl:variable name="переменная 1"
    select="(значение 1 and условие1) or (значение2 and not(условие2))"/>
```

Для множества узлов можно использовать предикаты и операции над множествами:

```xml
<xsl:variable name="переменная1"
    select="значение1[условие1] | значение2[not(условие2)]"/>
```

Заметим, что шаблон, содержащийся в элементе `xsl:variable`, может включать в себя такие элементы, как `xsl:call-template`, `xsl:apply-templates` и так далее. То есть переменной можно присвоить результат выполнения одного или нескольких шаблонов.

Использование внешних документов в преобразовании, как правило, сопровождается громоздкими выражениями вида:

```
    document('http://www.xmlhost.com/docs/а.xml')/page/request/param
```

и так далее.

Для того чтобы при обращении к внешнему документу не использовать каждый раз функцию [`document`](/xpath/document/), можно объявить переменную, которая будет содержать корневой узел этого документа, например:

```xml
<xsl:variable
    name="a.xml"
    select="document('http://www.xmlhost.com/docs/a.xml')"/>
```

После этого к документу `http://www.xmlhost.com/docs/a.xml` можно обращаться посредством переменной с именем `a.xml`, например:

```xml
<xsl:value-of select="$a.xml/page/request/param"/>
```

## См. также

- [xsl:param](/xslt/xsl-param/)

## Ссылки

- [`xsl:variable`](https://developer.mozilla.org/en-US/docs/Web/XSLT/variable) на MDN
- [`xsl:variable`](https://msdn.microsoft.com/en-us/library/ms256447.aspx) на MSDN