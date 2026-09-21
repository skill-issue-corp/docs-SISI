# Перенос визуализаторов внешнего вида

Как объясняется в [документации по спрайтам](../making-a-sprite-dynamic.md), визуализаторы — это способ изменения клиентских спрайтов с использованием данных внешнего вида с сервера. 

Старый метод — использовать класс, наследующий от `AppearanceVisualizer`, который указывается в `AppearanceComponent`. Новый способ — просто использовать компонент для данных, настраивающих визуализатор, и систему для самой логики вместо одного класса. Преимущество в том, что они могут использовать всё, что могут ECS-системы (в том числе, что важно, подписки на события!)

Эта документация объясняет, как перенести `AppearanceVisualizer` в новый компонент и систему, используя простой пример из этого PR: https://github.com/space-wizards/space-station-14/pull/6571/files с очень небольшими отклонениями. Директивы using и прочее не приводятся, так что вам придётся добавить их самостоятельно

Вот полный визуализатор, который мы переносим:

```csharp
    [UsedImplicitly]
    public class ItemCabinetVisualizer : AppearanceVisualizer
    {
        [DataField(required: true)]
        private string _openState = default!;

        [DataField(required: true)]
        private string _closedState = default!;

        public override void OnChangeData(AppearanceComponent component)
        {
            base.OnChangeData(component);

            var entities = IoCManager.Resolve<IEntityManager>();
            if (entities.TryGetComponent(component.Owner, out SpriteComponent sprite)
                && component.TryGetData(ItemCabinetVisuals.IsOpen, out bool isOpen)
                && component.TryGetData(ItemCabinetVisuals.ContainsItem, out bool contains))
            {
                var state = isOpen ? _openState : _closedState;
                sprite.LayerSetState(ItemCabinetVisualLayers.Door, state);
                sprite.LayerSetVisible(ItemCabinetVisualLayers.ContainsItem, contains);
            }
        }
    }
```

## 1. Разделите данные и логику

Первая задача — скопировать данные в новый компонент, а логику — в новую систему. Не беспокойтесь о том, чтобы полностью перенести его сразу, или о возможных ошибках, мы займёмся этим позже.

Компонент:

```csharp
    [RegisterComponent]
    public sealed class ItemCabinetVisualsComponent : Component
    {
        [DataField(required: true)]
        private string _openState = default!;

        [DataField(required: true)]
        private string _closedState = default!;
    }
```

Система:

```csharp
    public sealed class ItemCabinetVisualizerSystem : VisualizerSystem<ItemCabinetVisualsComponent>
    {
        public override void OnChangeData(AppearanceComponent component)
        {
            base.OnChangeData(component);

            var entities = IoCManager.Resolve<IEntityManager>();
            if (entities.TryGetComponent(component.Owner, out SpriteComponent sprite)
                && component.TryGetData(ItemCabinetVisuals.IsOpen, out bool isOpen)
                && component.TryGetData(ItemCabinetVisuals.ContainsItem, out bool contains))
            {
                var state = isOpen ? _openState : _closedState;
                sprite.LayerSetState(ItemCabinetVisualLayers.Door, state);
                sprite.LayerSetVisible(ItemCabinetVisualLayers.ContainsItem, contains);
            }
        }
    }
```

## 2. Приведение данных к ECS-виду

Теперь нам нужно привести компонент к правильному состоянию ECS!

1) Сделайте все приватные поля публичными
2) Удалите все свойства и просто используйте поля хранения; любая логика должна находиться в методах-членах системы
3) Измените имена всех полей в соответствии с соглашениями об именовании
4) Добавьте дополнительные поля данных при необходимости
5) Перенесите соответствующее перечисление `VisualLayers`, если оно есть, также в класс компонента

Теперь это будет выглядеть так:

```csharp
    [RegisterComponent]
    public sealed class ItemCabinetVisualsComponent : Component
    {
        [DataField(required: true)]
        public string OpenState = default!;

        [DataField(required: true)]
        public string ClosedState = default!;
    }
```

## 3. Приведение логики к ECS-виду

Логику нужно перенести двумя способами:
- Метод `OnAppearanceChange` нужно преобразовать в правильное переопределение системы сущностей
- Любой метод `InitializeEntity` нужно преобразовать в новый обработчик события `ComponentInit`, направленный на созданный вами компонент

Нужно сделать ещё пару вещей:
- Все зависимости следует перенести в систему
- Все ручные resolve следует превратить в зависимости
- Все resolve `IEntityManager` должны использовать поле `EntityManager`, уже существующее в `EntitySystem`, или проксирующие методы
- Все TryGet для `SpriteComponent` должны вместо этого использовать поле `Sprite` из аргументов события
- Все ссылки на поля, которые раньше были в визуализаторе, нужно преобразовать в ссылки на поля компонента

В этом случае нужен только первый, но позже я также покажу пример второго

Теперь сигнатура изменения внешнего вида — `protected override void OnAppearanceChange(EntityUid uid, T component, ref AppearanceChangeEvent args)`, поэтому мы обновим функцию в соответствии с ней.

Нам также нужно убрать resolve `IEntityManager` и преобразовать вызовы к нему в проксирующие методы. Однако, поскольку используемые вызовы методов служат лишь для получения `SpriteComponent`, мы можем вместо этого использовать поле из аргументов события.

```csharp
    public sealed class ItemCabinetVisualizerSystem : VisualizerSystem<ItemCabinetVisualsComponent>
    {
        public override void OnChangeData(EntityUid uid, ItemCabinetVisualsComponent component, ref AppearanceChangeEvent args)
        {
            if (args.Sprite != null)
                && component.TryGetData(ItemCabinetVisuals.IsOpen, out bool isOpen)
                && component.TryGetData(ItemCabinetVisuals.ContainsItem, out bool contains))
            {
                var state = isOpen ? component.OpenState : component.ClosedState;
                args.Sprite.LayerSetState(ItemCabinetVisualLayers.Door, state);
                args.Sprite.LayerSetVisible(ItemCabinetVisualLayers.ContainsItem, contains);
            }
        }
    }
```

Здесь не используется `InitializeEntity`, но если бы он использовался, полный класс выглядел бы так:

```csharp
    public sealed class ItemCabinetVisualizerSystem : VisualizerSystem<ItemCabinetVisualsComponent>
    {
        public override void Initialize()
        {
            base.Initialize(); // это очень важно! в этот раз оно нужно

            SubscribeLocalEvent<ItemCabinetVisualsComponent, ComponentInit>(OnComponentInit);
        }

        private void OnComponentInit(Entity<ItemCabinetVisualsComponent> ent, ref ComponentInit args)
        {
            // поведение!
        }

        protected override void OnChangeData(EntityUid uid, ItemCabinetVisualsComponent component, ref AppearanceChangeEvent args)
        {
            if (args.Sprite != null
                && component.TryGetData(ItemCabinetVisuals.IsOpen, out bool isOpen)
                && component.TryGetData(ItemCabinetVisuals.ContainsItem, out bool contains))
            {
                var state = isOpen ? component.OpenState : component.ClosedState;
                args.Sprite.LayerSetState(ItemCabinetVisualLayers.Door, state);
                args.Sprite.LayerSetVisible(ItemCabinetVisualLayers.ContainsItem, contains);
            }
        }
    }
```


## 4. Обновите YAML и IgnoredComponents.cs

Теперь нам нужно обновить YAML для нашего нового компонента, а также список игнорируемых компонентов сервера.

Перейдите в `Content.Server/Entry/IgnoredComponents.cs` и добавьте строку с именем вашего нового компонента, вот так:

```csharp
        public static string[] List => new [] {
            ... snip ...
            "ItemCabinetVisuals",
        };
 ```
 
 Это делается автоматически для серверных компонентов, которых нет на клиенте, но не наоборот, поскольку обычно такая операция встречается реже, и нельзя сделать и то, и другое. Так что это просто говорит серверу не беспокоиться, когда он увидит этот компонент в YAML.
 
 Найдите использования старого визуализатора с помощью CTRL+SHIFT+F или аналогичной комбинации в любой IDE, вот так:
 
 ```yaml
     - type: Appearance
      visuals:
        - type: ItemCabinetVisualizer
          openState: open
          closedState: closed
```

Замените его так:

 ```yaml
     - type: Appearance
     - type: ItemCabinetVisuals
       openState: open
       closedState: closed
```

**Важно оставить компонент внешнего вида на месте!** Легко упустить этот баг. По сути, просто уберите отступ у блока visuals и измените имя компонента.

На этом всё!

## 5. По возможности обобщайте.

Вместо добавления множества отдельных систем и компонентов визуализаторов часто можно просто сделать визуализатор более общим, добавив дополнительное поле данных YAML. Для этого существует `GenericVisualizerSystem` и компонент, заменяющие старый `GenericEnumVisualizer`. Если всё, что нужно визуализатору, — это задавать данные слоя спрайта на основе простых записей данных внешнего вида, то вы, скорее всего, можете и должны просто использовать общий визуализатор вместо создания собственного. Однако если вам нужно делать навороченные вещи, например использовать анимации или более сложную логику, вам всё равно придётся создать собственный.

Например, функциональность визуализатора шкафчика выше просто задаёт состояния и видимость слоя спрайта на основе двух записей данных внешнего вида. Вместо этой системы и компонента той же функциональности можно добиться, используя общий визуализатор:
```yaml
     - type: Appearance
     - type: GenericVisualizer
       visuals:
         enum.ItemCabinetVisuals.IsOpen: # <- Ключ данных внешнего вида. Либо enum, либо обычная строка.
           enum.ItemCabinetVisualLayers.Door: # <- Ключ слоя спрайта. Либо enum, либо обычная строка.
            True: # <- Значение данных внешнего вида
              state: open # <- Данные слоя спрайта, которые следует использовать для этого значения внешнего вида
            False: { state: closed } # <- Можно также записать YAML в одну строку, что уменьшает отступы и улучшает читаемость.
         # и затем то же самое для другой записи внешнего вида:
         enum.ItemCabinetVisuals.ContainsItem:
           enum.ItemCabinetVisualLayers.ContainsItem:
             True: { visible: true}
             False: { visible: false}
```
Данные слоя спрайта могут задавать `sprite`, `state`, `texture`, `shader`, `scale`, `rotation`, `offset`, `visible` и `color`.
Обратите внимание, что YAML для значений внешнего вида — это просто результаты `ToString()` значений данных внешнего вида.
Так, `bool` становятся "True"/"False", а enum вроде `VentPumpState.Off` просто становится "Off".
