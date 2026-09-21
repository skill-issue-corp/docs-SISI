# Команды сущностей
{{#title Команды сущностей}}

## Источники
{{#template 
    ../../../templates/toolshed-command-head.md
    name=entities
    typesig=[none] -> IEnumerable<EntityUid>
}}
Возвращает список всех сущностей в симуляции.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=ent &lt;entity&gt;
    typesig=[none] -> EntityUid
}}
Возвращает заданную сущность.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=spawn:at &lt;entity prototype&gt;
    typesig=IEnumerable?<EntityCoordinates> -> IEnumerable?<EntityUid>
}}
Создаёт новую сущность в заданных координатах.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=spawn:on &lt;entity prototype&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Создаёт новую сущность на другой заданной сущности.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=spawn:attached &lt;entity prototype&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Создаёт новую сущность, прикреплённую к другой заданной сущности.

## Фильтры
{{#template 
    ../../../templates/toolshed-command-head.md
    name=with &lt;component type&gt;
    typesig=IEnumerable<EntityUid> -> IEnumerable<EntityUid>
}}
Фильтрует входные данные, оставляя сущности с заданным компонентом.
Эту команду можно инвертировать с помощью `not`.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=prototyped &lt;prototype&gt;
    typesig=IEnumerable<EntityUid> -> IEnumerable<EntityUid>
}}
Фильтрует входные данные, оставляя сущности, созданные из заданного прототипа.
Эту команду можно инвертировать с помощью `not`.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=paused
    typesig=IEnumerable<EntityUid> -> IEnumerable<EntityUid>
}}
Фильтрует входные данные, оставляя сущности, которые сейчас приостановлены.
Эту команду можно инвертировать с помощью `not`.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=nearby &lt;range&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable<EntityUid>
}}
Фильтрует сущности, находящиеся рядом с входными данными, возвращая все сущности в пределах радиуса от них.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=named &lt;name&gt;
    typesig=IEnumerable<EntityUid> -> IEnumerable<EntityUid>
}}
Фильтрует входные данные, оставляя сущности, имя которых соответствует регулярному выражению `$regex^`.

## Преобразования
{{#template 
    ../../../templates/toolshed-command-head.md
    name=comp:has &lt;component type&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<bool>
}}
Возвращает true, если у входной сущности есть заданный компонент, иначе false.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=pos
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityCoordinates>
}}
Возвращает координаты входных сущностей относительно их родителя.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=mappos
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityCoordinates>
}}
Возвращает координаты входных сущностей относительно карты.

## Мутаторы
{{#template 
    ../../../templates/toolshed-command-head.md
    name=delete
    typesig=IEnumerable<EntityUid> -> [none]
}}
Удаляет входные данные из симуляции. Исчезли. Пуф.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=replace &lt;entity prototype&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Заменяет заданные сущности на другие из некоторого прототипа, сохраняя только их позицию и вращение.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=tp:coords &lt;entity coordinates&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Телепортирует входные данные в заданные координаты.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=tp:to &lt;entity&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Телепортирует входные данные к заданной сущности.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=tp:into &lt;entity&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Телепортирует входные данные внутрь заданной сущности.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=comp:add &lt;component type&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Добавляет заданный компонент сущности.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=comp:ensure &lt;component type&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Гарантирует, что у входных данных есть заданный компонент.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=comp:rm &lt;component type&gt;
    typesig=IEnumerable?<EntityUid> -> IEnumerable?<EntityUid>
}}
Удаляет заданный компонент у сущности.
