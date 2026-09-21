
# Разработка

Этот раздел предназначен для разработчиков, которые хотят создавать свои собственные команды Toolshed, и по большей части состоит из множества примеров.

## Создание новой команды

Чтобы создать новую команду Toolshed, нужно создать новый класс, который:
* Наследуется от `ToolshedCommand`
* Имеет имя класса, оканчивающееся на "Command"
* Помечен атрибутом `ToolshedCommandAttribute`
* Имеет один или несколько (нестатических) методов, помеченных атрибутом `CommandImplementationAttribute`. 

Минимальный рабочий пример, определяющий команду `foo`, выглядел бы так:
```cs
[ToolshedCommand]
public sealed class FooCommand : ToolshedCommand
{
    [CommandImplementation]
    public void Bar()
    {
    }
}
```
Имя команды в предыдущем примере берётся автоматически из имени класса, имя метода не имеет значения. То есть класс `FooCommand` сопоставляется с `foo`. Как вариант, имя команды можно указать с помощью атрибута класса, например `[ToolshedCommand(Name = "foo")]`. Если имя указано явно, имя класса **не** обязано оканчиваться на "Command", но всё же это хорошее соглашение, которого стоит придерживаться.


```admonish note "Соглашение об именовании"
Автоматически сгенерированные имена команд можно настроить для каждого проекта на использование snake_case. Поэтому для поддержки преобразования имён классов CamelCase следует избегать имён классов с аббревиатурами, содержащими идущие подряд заглавные буквы. То есть используйте что-то вроде `GetNpcCommand` вместо `GetNPCCommand`, так как последнее будет преобразовано в "get_n_p_c".
```

## Аргументы и возвращаемые значения
Чтобы определить команду, возвращающую некоторое значение, которое затем можно передать по конвейеру в другую команду, вам нужно просто задать у метода некоторое возвращаемое значение. Чтобы дать команде аргументы, вам нужно просто добавить аргументы в метод. Например,
```cs
[ToolshedCommand]
public sealed class FooCommand : ToolshedCommand
{
    [CommandImplementation]
    public string Bar(string text, float number, EntityUid entity, BodyType physicsBodyType)
    {
        return $"{text}, {number}, {entity}, {physicsBodyType}";
    }
}
```
```
> foo "A!" 42 10 Dynamic
A!, 42, 10, Dynamic

> foo "A!" 42 10 Dynamic | join ", suffix" 
A!, 42, 10, Dynamic, suffix
```

Любой аргумент метода, не имеющий атрибута (и не являющийся типом `IInvocationContext`), считается обычным аргументом команды, и Toolshed попытается разобрать его из строки команды. При желании их также можно явно пометить атрибутом `CommandArgumentAttribute`.

Toolshed также поддерживает методы с необязательными аргументами и аргументами `params []`. Например,
```cs
[ToolshedCommand]
public sealed class SumCommand : ToolshedCommand
{
    [CommandImplementation]
    public int Sum(params int[] values)
    {
        return values.Sum();
    }
}
```

## Парсеры аргументов
Toolshed может разбирать любой тип аргумента, для которого есть соответствующая реализация `TypeParser<T>`. Например, строковые аргументы разбираются классом `StringTypeParser : TypeParser<string>`. Парсер отвечает за генерацию вариантов и подсказок автодополнения консольных команд. Если какой-то тип пока не поддерживается, вы всегда можете просто создать свой собственный парсер.

Если вы хотите получить больше контроля над тем, как разбирается один из ваших аргументов, или больше контроля над предложениями автодополнения, вы также можете использовать атрибут аргумента, чтобы указать, что он должен использовать [пользовательский парсер](#custom-type-parsers).

## Аргументы конвейерного ввода
Чтобы создать команду, которая может принимать входные значения, передаваемые по конвейеру из другой команды, вам нужно дать методу аргумент, помеченный атрибутом `PipedArgumentAttribute`. Например, так создаётся простая команда сложения:
```cs
[ToolshedCommand]
public sealed class AddCommand : ToolshedCommand
{
    [CommandImplementation]
    public int Add([PipedArgument] int x, int y)
    {
        return x + y;
    }
}
```

```
> i 2 | add 3
5
```

## Инвертируемые команды

Чтобы создать команду, поведение которой можно инвертировать, поставив перед ней ключевое слово "not", нужно дать методу аргумент `bool`, помеченный атрибутом `CommandInvertedAttribute`. Например, это простая команда, которая ищет определённое число в последовательности:
```cs
[ToolshedCommand]
internal sealed class ContainsintCommand : ToolshedCommand
{
    [CommandImplementation]
    public bool Containsint([PipedArgument] IEnumerable<int> input, int value, [CommandInverted] bool inverted)
    {
        var result = input.Contains(value);
        return inverted ? !result : result;
    }
}
```

```
> i 1 to 5 | containsint 2
true

> i 1 to 5 | not containsint 2
false
```

## Контексты вызова

Если вы хотите создать команду, которая пишет вывод в консоль или может читать и записывать переменные Toolshed, вам нужен метод, принимающий аргумент `IInvocationContext`. Этот аргумент также можно при желании пометить атрибутом `CommandInvocationContextAttribute`. Например, это простая команда, которая выдаёт одноразовые приветствия:
```cs
[ToolshedCommand]
public sealed class HelloCommand : ToolshedCommand
{
    [CommandImplementation]
    public void Hello(IInvocationContext ctx)
    {
        if (ctx.ReadVar("greeted") is true)
            return;

        ctx.WriteLine("Hello World!"); // Or WriteMarkup, or WriteError
        ctx.WriteVar("greeted", true);
    }
}
```
В настоящее время наиболее распространённым контекстом вызова является `OldShellInvocationContext`, где у каждого игрока есть свой собственный контекст, сохраняющийся между отключениями и повторными подключениями, но не между перезапусками сервера. Контекст также не передаётся по сети, поэтому команды, выполняемые на клиенте и на сервере, будут использовать разные контексты вызова.

## Зависимости

Команды Toolshed поддерживают обычное внедрение зависимостей `EntitySystem` и менеджеров. Поэтому, если вашей команде нужно работать с трансформами сущностей, вы можете просто добавить в класс обычное поле зависимости, то есть
```cs
[Dependency] private readonly SharedTransformSystem _sys = default!;
```
Базовый класс `ToolshedCommand` уже предоставляет зависимости `ToolshedManager`, `ILocalizationManager` и `IEntityManager`. Он также определяет некоторые полезные прокси-методы `IEntityManager` (например, `TryComp<T>`, `Spawn` и т. д.). Так что в целом вы можете просто писать код так, как обычно делали бы это внутри `EntitySystem`.


## Несколько реализаций и подкоманды

До сих пор все примеры определяли команду с единственным методом реализации. Однако команды могут иметь более одной реализации, но каждая реализация должна принимать другой конвейерный тип. Например, это дало бы допустимую команду, которая может принимать либо целое число, либо число с плавающей запятой:
```cs
[ToolshedCommand]
public sealed class ToStringCommand : ToolshedCommand
{
    [CommandImplementation]
    public string Impl([PipedArgument] int x)
    {
        return x.ToString();
    }
    
    [CommandImplementation]
    public string Impl([PipedArgument] float x)
    {
        return x.ToString();
    }
}
```

Однако одно из ограничений Toolshed состоит в том, что комбинация имени команды и типа конвейерного ввода должна быть уникальной. Поэтому нельзя определить две реализации, принимающие один и тот же конвейерный тип, но с разными аргументами. То есть это **не** допустимый способ определить команду, принимающую либо координаты карты, либо координаты сущности:
```cs
public sealed class TpCommand : ToolshedCommand
{
    [Dependency] private readonly SharedTransformSystem _sys = default!;

    [CommandImplementation]
    public void Teleport([PipedArgument] EntityUid uid, EntityUid parent, Vector2 pos)
    {
        _sys.SetCoordinates(uid, new EntityCoordinates(parent, pos));
    }

    [CommandImplementation]
    public void Teleport([PipedArgument] EntityUid uid, MapId map, Vector2 pos)
    {
        _sys.SetCoordinates(uid, _sys.ToCoordinates(new MapCoordinates(pos, map)));
    }
}
```

Это ограничение в основном связано с тем, что Toolshed не смог бы определить, какую команду или аргументы ему следует пытаться разобрать. Вместо этого, если вы хотите ввести такие варианты команды, вам нужно использовать **подкоманды**. 

В некотором смысле подкоманды — это просто именованные реализации/методы команды, где имя задаётся через `CommandImplementationAttribute`. Обратите внимание, что если команда содержит **любые** именованные реализации, то всем им нужно дать имя. В качестве примера нашу прежнюю команду можно исправить, назвав реализации:
```cs
public sealed class TpCommand : ToolshedCommand
{
    [Dependency] private readonly SharedTransformSystem _sys = default!;

    [CommandImplementation("ent")]
    public void Teleport([PipedArgument] EntityUid uid, EntityUid parent, Vector2 pos)
    {
        _sys.SetCoordinates(uid, new EntityCoordinates(parent, pos));
    }

    [CommandImplementation("map")]
    public void Teleport([PipedArgument] EntityUid uid, MapCoordinates mapCoords)
    {
        _sys.SetCoordinates(uid, _sys.ToCoordinates(mapCoords));
    }
}
```
Тогда это определит "под"-команды `tp:ent` и `tp:map`

```admonish note "Соглашение об именовании"
По соглашению все новые команды должны использовать snake_case при именовании команд или подкоманд.
```

## Дженерики

Команды Toolshed имеют некоторую поддержку дженериков C#, хотя есть несколько ограничений. Наиболее распространённый случай использования — когда вы хотите определить метод, принимающий некоторый произвольный конвейерный тип ввода и использующий тип ввода в качестве дженерик-аргумента. В этом случае вы просто помечаете свой дженерик-метод атрибутом `TakesPipedTypeAsGenericAttribute`. Например, это часть определения настоящей команды сложения:
```cs
public sealed class AddCommand : ToolshedCommand
{
    [CommandImplementation, TakesPipedTypeAsGeneric]
    public T Operation<T>([PipedArgument] T x, T y) where T : IAdditionOperators<T, T, T>
    {
        return x + y;
    }
}
```

Атрибут `TakesPipedTypeAsGeneric` также поддерживает извлечение дженерик-типа, даже если он напрямую не соответствует типу конвейерного аргумента. Например, если конвейерный аргумент — это `IEnumerable<T>`, он всё ещё может извлечь дженерик-тип `T` из переданного по конвейеру значения. Например, вот что делает команда append:
```cs
[ToolshedCommand]
public sealed class AppendCommand : ToolshedCommand
{
    [CommandImplementation, TakesPipedTypeAsGeneric]
    public IEnumerable<T> Append<T>([PipedArgument] IEnumerable<T> x, T y)
    {
        return x.Append(y);
    }
}
```

Однако в более сложных ситуациях это, вероятно, не сработает. Например, сигнатура вида `Foo<T>([PipedArgument] Dictionary<int, List<(T, string)>> input)`, вероятно, не сможет извлечь `T` из заданного переданного по конвейеру значения. 

Также отсутствует поддержка автоматического определения нескольких дженерик-аргументов из конвейерного ввода. Если вам нужны команды, использующие более сложные дженерики, как правило, вам придётся определить команду с явными типовыми аргументами.

## Типовые аргументы

Если вам нужно создать команду, использующую несколько дженерик-аргументов или имеющую дженерики, которые нельзя автоматически вывести из конвейерного ввода, вам нужно использовать явные типовые аргументы. При написании команды оболочки типовые аргументы выглядят как обычные аргументы, но они всегда предшествуют любому другому аргументу и используются для определения типов для дженерик-реализации. 

Чтобы команда требовала типовые аргументы, нужно переопределить свойство `TypeParameterParsers` команды. Оно должно возвращать массив типов, наследующихся от `TypeParser<Type>`, и будет использоваться для фактического разбора типовых аргументов из строки команды. Поскольку это свойство уровня класса, это означает, что **все** реализации или подкоманды должны требовать одинаковое число типовых аргументов. Вы также можете комбинировать явные типовые аргументы с `TakesPipedTypeAsGenericAttribute`. Обратите внимание, что автоматически выведенный типовой аргумент всегда должен быть последним типовым аргументом этой функции.

Например, эти две команды используют явные типовые аргументы для вывода синтаксиса вызова метода в стиле C#:
```cs
[ToolshedCommand]
public sealed class FooCommand : ToolshedCommand
{
    public override Type[] TypeParameterParsers { get; } = [typeof(TypeTypeParser), typeof(TypeTypeParser)];

    [CommandImplementation]
    public string Foo<T1, T2>(int x)
    {
        return $"Foo<{typeof(T1).Name}, {typeof(T2).Name}>({x})";
    }
}

[ToolshedCommand]
public sealed class BarCommand : ToolshedCommand
{
    public override Type[] TypeParameterParsers { get; } = [typeof(TypeTypeParser)];

    [CommandImplementation, TakesPipedTypeAsGeneric]
    public string Bar<TExplicit, TAuto>([PipedArgument] TAuto x)
    {
        return $"Bar<{typeof(TExplicit).Name}, {typeof(TAuto).Name}>({x})";
    }
}

```

```
> foo string int 123
Foo<String, Int32>(123)

> i 123 | bar string 
Bar<String, Int32>(123)

> f 1.23 | bar String 
Bar<String, Single>(1.23)
```

## Автоматические преобразования типов

Как упоминалось в других местах документации, Toolshed выполняет некоторые автоматические преобразования типов. Наиболее заметно то, что любая команда, ожидающая `IEnumerable<T>`, также примет передачу по конвейеру `T`, так как Toolshed автоматически преобразует его в `IEnumerable<T>` с одним элементом.

Toolshed также автоматически приводит любой тип, реализующий интерфейс `IAsType<T>`. Например, `Entity<T>` реализует `IAsType<EntityUid>`. Поэтому Toolshed позволит вам передать по конвейеру вывод `Entity<T>` в метод, ожидающий ввод `EntityUid`.

## Пользовательские парсеры типов

Если вы хотите создать метод, использующий пользовательский парсер, вы можете указать пользовательский парсер через `CommandArgumentAttribute` аргумента. Это полезно, если вы хотите получить больше контроля над разбором или над вариантами/подсказками автодополнения консоли. 

Например, следующее определяет метод, использующий пользовательский парсер для получения целого числа из двоичной строки. Хотя в этом конкретном случае вы с тем же успехом могли бы сделать так, чтобы команда принимала строку, и выполнять преобразование внутри собственного метода команды, но тогда аргумент нужно было бы заключать в кавычки (все строковые аргументы нужно заключать в кавычки).
```cs
[ToolshedCommand]
public sealed class BinaryCommand : ToolshedCommand
{
    [CommandImplementation]
    public int FromBinary([CommandArgument(typeof(BinaryParser))] int value) => value;
}

public sealed class BinaryParser : CustomTypeParser<int>
{
    public override bool TryParse(ParserContext ctx, out int result)
    {
        var binaryText = ctx.GetWord();
        try
        {
            result = Convert.ToInt32(binaryText, 2);
            return true;
        }
        catch
        {
            result = 0;
            return false;
        }
    }

    public override CompletionResult? TryAutocomplete(ParserContext ctx, CommandArgument? arg)
        => CompletionResult.FromHint("<binary number>");
}
```

```
> binary 10101
21
```


## Разрешения

```admonish warning
Этот раздел относится конкретно к SS14, так как RobustToolbox не поставляется с реализацией разрешений команд.
```

Всем командам Toolshed нужно указать некоторые разрешения, чтобы их можно было выполнять, и существует интеграционный тест, проверяющий, что это так (`AdminTest.AllCommandsHavePermissions`). Разрешения для команд, определённых в движке, указаны в `/Resources/toolshedEngineCommandPerms.yml`, а командам из Content можно дать разрешения, пометив класс команды обычными атрибутами (`AnyCommandAttribute`, `AdminCommandAttribute`). Разрешения нельзя указывать для отдельных подкоманд, все подкоманды должны иметь одинаковые разрешения.


## Автодополнение, подсказки и локализация

У каждой команды Toolshed должно быть локализованное описание. Ключ для локализованной строки основан на имени (под)команды. Например, `foo` или `foo:bar` использует "command-description-foo" или "command-description-foo-bar". Если имя команды содержит не-ASCII символы, вместо этого будет использоваться имя класса. Например, команда сложения (+) определена в классе `AddCommand`, поэтому она использует "command-description-AddCommand".

Toolshed автоматически генерирует строки справки для команд в виде сигнатуры метода. Автоматически сгенерированную строку справки можно переопределить, определив локализованную строку. Например, справку команды foo можно переопределить, определив локализованную строку с ключом "command-help-foo".

### Подсказки аргументов

Большинство парсеров аргументов Toolshed автоматически генерируют подсказки автодополнения консоли при вводе аргументов команды. Например, при вводе аргумента для метода вида `Foo(int myNumber)` будет сгенерирована подсказка `[myNumber (int)]`. Если вы хотите переопределить автоматически сгенерированную подсказку, вы можете сделать это, определив локализованную строку с ключом "command-arg-hint-foo-myNumber". Если вы хотите получить больше контроля над подсказкой или предложениями автодополнения, вы можете использовать пользовательский парсер.


## Отчёты об ошибках

TODO

## Блоки команд

TODO 