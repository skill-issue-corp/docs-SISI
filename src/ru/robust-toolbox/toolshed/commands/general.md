# Общие

## Источники
{{#template 
    ../../../templates/toolshed-command-head.md
    name=iota
    typesig=T -> IEnumerable<T> where T: INumber<T>
}}
Возвращает <code>1..N</code>

{{#template 
    ../../../templates/toolshed-command-head.md
    name=to &lt;dest&gt;
    typesig=T -> IEnumerable<T> where T: INumber<T>
}}
Возвращает <code>N..M</code>

## Фильтры
{{#template 
    ../../../templates/toolshed-command-head.md
    name=where &lt;block (T -> bool)&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Фильтрует входные данные с помощью предоставленного блока кода.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=unique
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Фильтрует входные данные по уникальности, устраняя повторяющиеся значения.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=take &lt;amount&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Убирает N значений из входных данных, отбрасывая остальные.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=select &lt;quantity&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Случайным образом убирает из входных данных некоторое количество значений (абсолютное или в процентах), отбрасывая остальные.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=sort
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Сортирует входные данные от меньшего к большему.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=sortby &lt;block (T -> TOrd)&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Сортирует входные данные от меньшего к большему, используя заданное значение упорядочивания.

```
    entities sortby { allcomps count }
```

{{#template 
    ../../../templates/toolshed-command-head.md
    name=sortmapby &lt;block (T -> TOrd)&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Сортирует входные данные от меньшего к большему, используя заданное значение упорядочивания, и возвращает значения упорядочивания.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=sortdown
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Сортирует входные данные от большего к меньшему.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=sortdownby &lt;block (T -> TOrd)&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Сортирует входные данные от большего к меньшему, используя заданное значение упорядочивания.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=sortmapdownby &lt;block (T -> TOrd)&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Сортирует входные данные от большего к меньшему, используя заданное значение упорядочивания, и возвращает значения упорядочивания.

## Преобразования
{{#template 
    ../../../templates/toolshed-command-head.md
    name=isempty
    typesig=IEnumerable<T> -> bool
}}
Возвращает true, если входные данные пусты, иначе false.
Эту команду можно инвертировать с помощью `not`.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=isnull
    typesig=object? -> bool
}}
Возвращает true, если входные данные равны null, иначе false.
Эту команду можно инвертировать с помощью `not`.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=count
    typesig=IEnumerable<T> -> int
}}
Подсчитывает число значений во входных данных.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=iterate &lt;block (T -> T)&gt; &lt;times&gt;
    typesig=IEnumerable<T> -> IEnumerable<T>
}}
Повторяет заданный блок кода N раз над его собственным выводом, что фактически даёт `f(f(...N f(x)))`

{{#template 
    ../../../templates/toolshed-command-head.md
    name=first
    typesig=IEnumerable<T> -> T
}}
Возвращает первое значение во входных данных или выдаёт ошибку, если его нет.


## Мутаторы
{{#template 
    ../../../templates/toolshed-command-head.md
    name=rep &lt;amount&gt;
    typesig=T -> IEnumerable<T>
}}
Повторяет входное значение N раз.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=bin
    typesig=IEnumerable<T> -> IDictionary<T, int>
}}
Подсчитывает, сколько раз каждый уникальный экземпляр T встречается во входных данных, возвращая словарь вида уникальный экземпляр : количество.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=map &lt;block (TIn -> TOut)&gt;
    typesig=IEnumerable<TIn> -> IEnumerable<TOut>
}}
Применяет заданный блок кода к каждому элементу входных данных.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=reduce &lt;block (T -> T)&gt;
    typesig=IEnumerable<TIn> -> T
}}
Свёртывает входные данные, что фактически даёт `f(x1, f(x2, ...f(xn-1, xn)))`
