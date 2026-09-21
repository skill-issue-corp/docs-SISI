# Разное

## Прочее
{{#template 
    ../../../templates/toolshed-command-head.md
    name=explain &lt;command run&gt;
    typesig=[none] -> [none]
}}
Выводит в консоль разбор потока входного прогона команд.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=stopwatch &lt;command run&gt;
    typesig=[none] -> [none]
}}
Выводит в консоль время выполнения входного прогона команд.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=search &lt;query&gt;
    typesig=IEnumerable<T> -> IEnumerable<FormattedMessage>
}}
Выполняет простой поиск строки по входным данным с подсветкой совпадений.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=help
    typesig=[none] -> [none]
}}
Выводит текст справки в консоль.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=more
    typesig=[none] -> object?
}}
Возвращает содержимое переменной `$more`. `$more` автоматически присваивается оболочкой, когда ей приходится обрезать сообщение из-за его длины.

{{#template 
    ../../../templates/toolshed-command-head.md
    name=buildinfo
    typesig=[none] -> [none]
}}
Выводит в консоль информацию о сборке игры (название игры, версия движка, коммит сборки, хеш манифеста).

{{#template 
    ../../../templates/toolshed-command-head.md
    name=cmd:moo
    typesig=[none] -> string
}}
Самая важная команда.
