# lang()

Функция **`lang`** возвратит "истину", если идентификатор языка, который передан ей в виде строкового параметра, соответствует языковому контексту контекстного узла.

## Синтаксис

### XPath 1.0

```
boolean lang( string )
```

## Описание и примеры

Функция `lang` может использоваться для того, чтобы определить языковой контекст контекстного узла. В элементах XML можно использовать атрибут `lang` пространства имен `xml` для определения языка содержимого узла, например;

```xml
<text xml:lang="en-gb">Yet no living human being have been ever blessed with seeing...</text>
```

Пространство имен, соответствующее префиксу `xml`, не требуется объявлять. Это служебное пространство имен, которое неявно задано во всех XML-документах.

Функция `lang` возвратит "истину", если идентификатор языка, который передан ей в виде строкового параметра, соответствует языковому контексту контекстного узла. Это определяется следующим образом.

- Если ни один из предков контекстного узла не имеет атрибута `xml:lang`, функция возвращает "ложь".
- Иначе строковый параметр проверяется на соответствие значению атрибута `xml:lang` ближайшего предка. Если эти значения равны в любом регистре символов, или атрибут начинается как значение параметра функции и имеет суффикс, начинающийся знаком "`-`", функция возвращает "истину".
- В противном случае функция возвращает "ложь".

### Примеры

Функция `lang('en')` возвратит "истину" в контексте любого из следующих элементов:

```xml
<body xml:lang="EN"/>
<body xml:lang="en-GB"/>
<body xml:lang="en-us"/>
<body xml:lang="EN-US"/>
```

Функция `lang('de')` возвратит "истину" в контексте элемента `b` и "ложь" — в контексте элементов `a` и `c`:

```xml
<а>
    <b xml:lang="de">
        <c xml:lang="en"/>
    </b>
</a>
```

## Ссылки

- [lang()](https://developer.mozilla.org/en-US/docs/Web/XPath/Functions/lang) на MDN