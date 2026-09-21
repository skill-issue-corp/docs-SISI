# Таблицы сущностей

**Таблицы сущностей** — мощный способ определения сущностей для наполнения контейнеров и спавнеров.
Система состоит из рекурсивных наборов селекторов, каждый из которых может задавать пользовательское поведение для выбора сущностей.

## Использование

Вы можете получить спавны из таблицы с помощью `EntityTableSystem.GetSpawns`. 
Этот вызов функции принимает только `EntityTableSelector` с необязательным параметром для `System.Random`, что позволяет создавать детерминированные применения.

На момент написания этого руководства таблицы сущностей в основном поддерживаются двумя компонентами: `EntityTableContainerFillComponent` и `EntityTableSpawnerComponent`.

`EntityTableContainerFillComponent` служит прямой заменой `ContainerFillComponent`.
Он использует тот же общий синтаксис, за исключением замены списка `EntitySpawnEntry` на `EntityTableSelector`.

Аналогично, `EntityTableSpawnerComponent` является заменой `RandomSpawnerComponent`.
Однако они не используют одни и те же переменные.
Эта версия поддерживает только таблицу и смещение, а не отдельные списки и переменную шанса старой системы.
Того же эффекта можно добиться, используя вложенные `GroupSelectors` с общим определением `prob` и `weights`, соответствующими прежним шансам.

### Пример синтаксиса

```admonish warning "Указание типа"
При написании таблиц сущностей вы должны использовать синтаксис `!type:[Class Name]`, когда указываете селектор.

Единственное исключение — `EntSelector`.
Простое указание ID заставит линтер предположить, что указанный селектор является EntitySelector.
Это значительно сокращает yaml для больших таблиц.
```

Вот многоразовый прототип таблицы сущностей:
```yaml
- type: entityTable
  id: LockFillTable
  table: !type:AllSelector # <-- Это означает, что будут выбраны все дочерние элементы
    children:
    - id: ClothingMaskBreath # <-- Это просто единственный экземпляр сущности
    - !type:GroupSelector # <-- Это означает, что будет выбран только один из дочерних элементов
      children:
      - id: EmergencyOxygenTankFilled # <-- У всех селекторов вес по умолчанию равен 1
      - id: OxygenTankFilled
        weight: 2 # <-- Это означает, что этот элемент будет выбран вдвое вероятнее
    - id: ToolboxEmergencyFilled
      prob: 0.5 # <-- Это означает, что есть 50% шанс, что этот селектор будет использован.
```

Вот пример встроенного в прототип сущности:
```yaml
- type: entity
  id: ClosetFilled
  components:
  - type: EntityTableContainerFill
    containers:
      entity_storage: !type:AllSelector
        children:
        - !type:NestedSelector # <-- Это рекурсивно вызовет GetSpawns для EntityTablePrototype
          tableId: LockFillTable
        - id: ClothingMaskGas
          amount: !type:ConstantNumberSelector # <-- Это выберет ровно 3 указанные сущности
            value: 3
        - id: StrangePill
          amount: !type:RangeNumberSelector # <-- Это выберет от 2 до 6 (включительно с обеих сторон) указанных сущностей
            range: 2, 6
```

## EntityTableSelectors

### Общие переменные

Все EntityTableSelectors имеют следующие переменные:

- **Rolls:** Количество раз, которое выполняется данный селектор.
Шансы `Prob` будут проверяться при каждом броске
- **Weight:** Вес, используемый для определения того, какой селектор выбран для `GroupSelector`
- **Prob:** Простая вероятность, используемая для определения, будет ли селектор выполнен.
От 0 до 1

### EntSelector

- **Id:** ID EntityPrototype, который вы хотите выбрать.
Обязательно.
- **Amount:** `NumberSelector`, соответствующий тому, сколько `Id` вы хотите выбрать.
По умолчанию 1, если ничего не указано.

Пример:
```yml
root:
- id: PlushieLizard
  amount: !type:ConstantNumberSelector
    value: 5
    
# или

root: !type:EntSelector
- id: PlushieLizard
  amount: !type:ConstantNumberSelector
    value: 5
```

### NoneSelector

Ничего не возвращает. 
Это можно использовать вместе с `GroupSelector` для вероятности.

Пример:
```yml
containers:
- storage: !type:NoneSelector
```

### AllSelector

`AllSelector` позволяет осуществлять базовый контроль при выборе списка селекторов, которые вы хотите использовать вместе друг с другом.

- **Children:** Список других EntityTableSelectors, из которых будет производиться выбор.

Пример:
```yml
root: !type:AllSelector
  children:
  - id: PlushieLizard
  - id: ClothingMaskGas
```

### GroupSelector

`GroupSelector` служит заменой OrGroup из EntitySpawnEntry.
Поскольку выбирается только один из дочерних элементов, обычное применение — вложение нескольких GroupSelector друг в друга для настройки шанса выбора предмета без явного вычисления Weight.

Аналогично, EntityTable с GroupSelector в качестве корня можно использовать как общий пул.
Это делает выбор нескольких предметов из пула таким же простым, как увеличение значения `Rolls`.

- **Children:** Список других EntityTableSelectors, один из которых будет выбран и из него произведён выбор на основе шанса и значения `Weight` этого селектора

Пример:
```yml
root: !type:GroupSelector
  children:
  - id: PlushieNar
  - id: PlushieRatvar
```

### NestedSelector

`NestedSelector` существует в первую очередь для уменьшения дублирования yaml.
Поскольку таблицу (или часть таблицы) можно указать как прототип, это позволяет ссылаться на неё несколько раз во многих местах.

- **TableId:** Строковый ID EntityTableSelector. 

Пример:
```yml
- type: entityTable
  id: test
  table: !type:GroupSelector
    children:
    - id: PlushieLizard
    - id: PlushieNar
    - id: PlushieRatvar

- type: entity
  id: PlushSpawner
  components:
  - type: EntityTableSpawner
    table: !type:NestedSelector
      tableId: test
```

## ValueSelectors

### NumberSelectors

`NumberSelector` — это просто общий способ указания числового значения для различных селекторов.

#### ConstantNumberSelector

- **Value:** Возвращает указанное значение.

Пример:
```yml
amount: !type:ConstantNumberSelector
  value: 4
```

#### RangeNumberSelector

- **Range:** Возвращает значение в указанном диапазоне (включительно с обеих сторон)

Пример:
```yml
rolls: !type:RangeNumberSelector
  range: 6, 2019
```

## Пользовательские селекторы

Чтобы реализовать пользовательский селектор таблицы сущностей, создайте новый класс, наследующий от `EntityTableSelector`.
Затем просто переопределите `GetSpawns(System.Random, IEntityManager, IPrototypeManager)` и создайте свою реализацию.

Вы можете добавить любые datafields в класс, но не можете использовать внедрение зависимостей для добавления систем.
Если вы хотите вызвать Manager или System, проще всего создать новый `EntitySystem` с публичным API, который обрабатывает выбор, а затем получить систему через переданный `IEntityManager`.

NumberSelectors работают так же, просто используется `NumberSelector` вместо `EntityTableSelector`. 
