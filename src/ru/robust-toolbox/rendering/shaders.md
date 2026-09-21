# Шейдеры

[Шейдеры](https://en.wikipedia.org/wiki/Shader) — это программы, используемые для реализации графических эффектов, выполняющихся на [GPU](https://en.wikipedia.org/wiki/Graphics_processing_unit). SS14 использует [пиксельные (фрагментные) шейдеры](https://en.wikipedia.org/wiki/Shader#Pixel_shaders) для реализации поэлементных эффектов спрайтов, включая невидимость; и оверлейных эффектов, включая космические наркотики, опьянение, слепоту и искривление пространства сингулярностью.

# Определение шейдера
У каждого шейдера есть YAML-прототип, хранящийся в `Resources/Prototypes/Shaders`. У всех шейдеров есть поля:

- `type`: должно быть `shader`
- `id`: уникальный ID прототипа (строка), идентифицирующий этот шейдер
- `kind`: либо `canvas`, либо `shader` (см. ниже)

## Шейдеры `canvas`
Это шейдер с предустановленными опциями, похожий на `CanvasItemMaterial` из Godot. У этого шейдера есть два обязательных свойства:

* `blend_mode`: Способ отрисовки объекта поверх сцены. Они такие же, как [эквивалент для Godot](https://godot.readthedocs.io/en/3.0/classes/class_canvasitemmaterial.html). Разве что названия немного отличаются. Возможные значения: `mix`, `add`, `subtract`, `multiply` и `premultiplied_alpha`
* `light_mode`: Способ взаимодействия объекта со светом. Опять же эквивалентен варианту Godot, со значениями `normal`, `unshaded` и `light_only`.

Например, шейдер `unshaded`, который используется для отрисовки освещённой части компьютерных дисплеев и индикаторов путём маскирования операций освещения, определяется так:

```yml
- type: shader
  id: unshaded
  kind: canvas
  light_mode: unshaded
```

Шейдер canvas на самом деле просто исходный шейдер по умолчанию (см. `/Shaders/Internal/default-sprite.swsl`) с применёнными опциями освещения, смешивания и трафарета.

## Шейдеры `source`
Это пользовательские шейдеры, написанные на Space Wizard Shader Language (SWSL). У шейдеров `source` есть одно обязательное свойство:

- `path`: Путь к исходному файлу шейдера SWSL относительно каталога `Resources/`.

Например:

```yml
- type: shader
  id: GreyscaleFullscreen
  kind: source
  path: "/Textures/Shaders/greyscale_fullscreen.swsl"
```

Пользовательские шейдеры обычно хранятся в файлах `.swsl` в `Resources/Textures/Shaders`.

## SWSL
Space Wizard Shader Language (SWSL) основан на [языке шейдеров Godot](https://godot.readthedocs.io/en/3.0/tutorials/shading/shading_language.html). Большинство различий совместимости между Godot и SWSL обрабатываются автоматически, кроме следующих различий:

- Вы ОБЯЗАНЫ убедиться, что все числовые типы (например, `float`, `vec3`) имеют квалификатор точности `highp` или `lowp`. См. [эту статью о том, почему](https://stackoverflow.com/questions/28540290/why-it-is-necessary-to-set-precision-for-the-fragment-shader).

- Избегайте имён переменных, которые являются зарезервированными словами в распространённых спецификациях шейдеров. Ваш компьютер может их проигнорировать, но на других машинах они сломаются. [Смотрите раздел с ключевыми словами (3.8) здесь](https://registry.khronos.org/OpenGL/specs/es/3.0/GLSL_ES_Specification_3.00.pdf)

Поскольку это 2D-игра, используется только фрагментный шейдер, т. е. шейдер состоит как минимум из:

```glsl
void fragment() {
    COLOR = vec4(r, g, b, a);
}
```

### Доступные переменные (фрагментные шейдеры)

| Имя                 | Тип           | Описание                               |
|---------------------|---------------|--------------------------------------|
| `FRAGCOORD`         | `highp vec4`  | Координаты внутри фрагмента. |
| `COLOR`             | `lowp vec4`   | Итоговый цвет пикселя. _(Думайте об этом как о возвращаемом значении фрагментного шейдера.)_ |
| `lightMap`          | `sampler2D`   | Карта освещения текущего фрагмента (применяется автоматически `base-default.frag`)|
| `modulate`          | `highp vec4`  | Цвет отрисовки (применяется автоматически `base-default.frag`)|
| `SCREEN_PIXEL_SIZE` | `highp vec2`  | Размер одного пикселя в локальных единицах.|
| `TIME`              | `highp float` | Количество секунд с момента запуска игры.|
<!--
| `projectionMatrix`  | `highp mat3`  | **TODO**                             |
| `viewMatrix`        | `highp mat3`  | **TODO**                             |
| `UV`                | `highp vec2`  | **TODO**                             |
| `Pos`               | `highp vec2`  | **TODO**                             |
| `TEXTURE`           | `sampler2D`   | **TODO**                             |
-->

## Параметры трафаретного теста

Шейдеры поддерживают трафаретные операции. Это продвинутая функция рендеринга, которая может быть полезна для некоторых вещей, если вы знаете, что делаете. Эта функция близко имитирует параметры трафарета, как они представлены в OpenGL (и, насколько я могу судить, в Vulkan тоже). См. [вики OpenGL](https://www.khronos.org/opengl/wiki/Stencil_Test), если нужна справка.

Параметры трафарета определяются в отдельном объекте `stencil`. Например:

```yml
- type: shader
  id: stencilDraw
  kind: canvas
  stencil:
    ref: 1
    op: Keep
    func: NotEqual
```

Действительно, параметры трафарета не зависят от `kind` шейдера.

Возможные варианты:

* `ref`: Опорное значение для сравнения / записи, передаваемое вторым параметром `glStencilFunc`.
  * По умолчанию 0.
* `op`: Операция, применяемая к буферу трафарета, если трафаретный тест пройден. Задать можно только операцию при успехе (третий параметр `glStencilOp`).
  * Варианты: `Keep`, `Zero`, `Replace`, `IncrementClamp`, `IncrementWrap`, `DecrementClamp`, `DecrementWrap`, `Invert`
  * По умолчанию `Keep`.
* `func`: Функция сравнения, используемая для проверки прохождения теста. (первый параметр `glStencilFunc`).
  * Варианты: `Always`, `Never`, `Less`, `LessOrEqual`, `Greater`, `GreaterOrEqual`, `NotEqual`, `Equal`.
  * По умолчанию `Always`.
* `readMask`: Маска, используемая при чтении из буфера трафарета (третий параметр `glStencilFunc`).
  * По умолчанию все 1.
* `writeMask`: Маска, используемая при записи в буфер трафарета (параметр `glStencilMask`).
  * По умолчанию все 1.
  
# Оверлеи
Оверлей применяет шейдер ко всему экрану или вьюпорту (в отличие от отдельных спрайтов). Эффекты наркотиков, слепота и искривление пространства сингулярностью — примеры оверлеев (написанных на C#), которые загружают прототипы шейдеров (определённые в YAML), которые могут загружать пользовательские шейдерные эффекты (написанные на SWSL).

Оверлеи наследуют `Overlay`, например:

```csharp
    public sealed class BlindOverlay : Overlay
    {
        [Dependency] private readonly IPrototypeManager _prototypeManager = default!;

        // Установите true, чтобы получить ScreenTexture. Иначе оно null.
        public override bool RequestScreenTexture => true;
        
        // Это нужно установить в соответствующий слой оверлея.
        public override OverlaySpace Space => OverlaySpace.WorldSpace;
        
        // Сохраняем ссылки на шейдеры.
        private readonly ShaderInstance _greyscaleShader;
        private readonly ShaderInstance _circleMaskShader;

        public BlindOverlay()
        {
            IoCManager.InjectDependencies(this);
            // Загружаем шейдеры из прототипов
            _greyscaleShader = _prototypeManager.Index<ShaderPrototype>("GreyscaleFullscreen").InstanceUnique();
            _circleMaskShader = _prototypeManager.Index<ShaderPrototype>("CircleMask").InstanceUnique();
        }
        
        protected override bool BeforeDraw(in OverlayDrawArgs args)
        {
            // Если это вернёт true, этот шейдер будет отрисован. Если этот
            // метод не переопределён, по умолчанию он возвращает true,
            // то есть шейдер всегда активен для всех всё время.
        }
        
        protected override void Draw(in OverlayDrawArgs args)
        {
            // Если вашему шейдеру нужны входные данные (пиксели, сейчас находящиеся на экране),
            // вы должны проверить, что оно не null, и передать его в ваш шейдер.
            if (ScreenTexture == null)
                return;
                
            _greyscaleShader?.SetParameter("SCREEN_TEXTURE", ScreenTexture);
            
            var handle = args.WorldHandle;
            var viewport = args.WorldBounds;
            // рисуем шейдер оттенков серого
            handle.UseShader(_greyscaleShader);
            handle.DrawRect(viewport, Color.White);
            // рисуем шейдер круговой маски
            handle.UseShader(_circleMaskShader);
            handle.DrawRect(viewport, Color.White);
            // прекращаем использовать этот шейдер
            handle.UseShader(null);
        }
    }
```

Шейдеру нужна область для отрисовки. Здесь это белый прямоугольник, равный **WorldBounds.** Учтите, что **WorldAABB** не скорректирован с учётом поворота и, вероятно, сломает многие шейдеры.

Наконец, чтобы оверлеи действительно отрисовывались, их нужно добавить в менеджер оверлеев:

```csharp
[Dependency] private readonly IOverlayManager _overlayMan = default!;
_overlayMan.AddOverlay(your_overlay_here);
_overlayMan.RemoveOverlay(your_overlay_here);
```

Если этот оверлей должен быть всегда активным, добавьте его в `PostInit()` в `Content.Client/Entry/EntryPoint.cs`.

# Тестирование и отладка

```admonish warning
Тестируйте свои шейдеры **как с режимом совместимости, так и без него**. Включить режим совместимости можно, запустив клиент с аргументом `--cvar display.compat=true`.
```

```admonish info
Вы можете использовать команду `/rldshader`, чтобы перезагрузить шейдеры `.swsl` без перезапуска игры. Это значит, что вы часто можете использовать цветовые выводы для интерактивной отладки шейдеров. 
```

В отличие от программ, написанных на C#, шейдеры компилируются вашим графическим драйвером *во время выполнения*, а это значит, что даже простые синтаксические ошибки в шейдерах **появятся только при запуске клиента**.

Поэтому важно тестировать свой шейдер, запуская клиент. Синтаксические ошибки шейдеров проявляются как исключения (вам придётся немного поискать, чтобы найти сообщение об ошибке) при загрузке прототипов шейдеров.

Для некоторых классов ошибок клиент выгружает файл `error.glsl` с постобработанным шейдером, который не удалось загрузить. Обычно также показывается лог вывода компиляции шейдера, определяющий проблему, но вы также можете передать файл `.glsl` в такой инструмент, как 

[glslang](https://github.com/KhronosGroup/glslang).

## Внешние инструменты
* [renderdoc](https://renderdoc.org/) - отличный инструмент для отладки рендеринга и шейдеров.
