---
editable: false
sourcePath: ru/_api-ref/datalens/function-ref/TOP_CONCAT.md
---


# TOP_CONCAT



#### Синтаксис {#syntax}


```
TOP_CONCAT( expression, amount [ , separator ] )
```

#### Описание {#description}
Возвращает строку, которая содержит `amount` наиболее часто встречающихся значений из каждой группы `expression` с разделителем `separator` (по умолчанию разделитель — запятая).

**Типы аргументов:**
- `expression` — `Массив дробных чисел | Массив целых числел | Массив строк | Логический | Дата | Дата и время | Дробное число | Геоточка | Геополигон | Целое число | Строка | UUID`
- `amount` — `Целое число`
- `separator` — `Строка`


**Возвращаемый тип**: `Строка`

{% note info %}

Значения аргументов (`amount`, `separator`) должны быть константами.

{% endnote %}


#### Примеры {#examples}

```
TOP_CONCAT([Profit], 3)
```

```
TOP_CONCAT([Profit], 3, '; ')
```


#### Поддержка источников данных {#data-source-support}

`Материализованный датасет`, `ClickHouse 19.13`, `YDB`.
