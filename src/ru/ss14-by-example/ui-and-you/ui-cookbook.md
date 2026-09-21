# Кулинарная книга UI

В этом разделе рассматриваются распространённые UI-паттерны, которые вы, возможно, захотите реализовать сами.

```admonish note
Если вы разобрались с чем-то хитрым в UI или просто хотите, чтобы люди узнали
о существовании того, что вы сделали, подумайте о том, чтобы поделиться и добавить это сюда!
```

## Цвета состояния

`Palettes.Status` позволяет показать состояние чего-либо с помощью цвета (красный = плохо,
янтарный = нормально, зелёный = хорошо). Вам просто нужно передать число от 0 до 1, и
он смешает цвета (используя навороченное смешивание OKLAB!!).

```cs
Palettes.Status.GetStatusColor(0.0f); // красный
Palettes.Status.GetStatusColor(0.5f); // янтарный
Palettes.Status.GetStatusColor(1.0f); // зелёный

Palettes.Status.GetStatusColor(0.25f); // смешивается между красным и янтарным
```

## Несколько стилевых классов

Чтобы добавить несколько стилевых классов к элементу в его XAML-файле, требуется
немного дополнительного шаблонного кода:

```xml
<Control xmlns:sys="clr-namespace:System;assembly=System.Runtime">
    <Button Text="delete">
        <Button.StyleClasses>
            <sys:String>ButtonSmall</sys:String>
            <sys:String>negative</sys:String>
        </Button.StyleClasses>
    </Button>
</Control>
```
