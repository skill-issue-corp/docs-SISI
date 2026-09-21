# Реагенты

Реагенты — это вещества, из которых состоит раствор и которые могут реагировать, создавая новые реагенты.

Их определение состоит из:
`type`: Имя компонента. Всегда должен быть `reagent`.
`id`: Уникальный идентификатор этого реагента. Используется в реакциях и других местах для идентификации реагентов. Например `Iodine` или `WeldingFuel`. Они в `PascalCase`.
`name`: Идентификатор с пробелами для отображения во всплывающих подсказках. Например, `iodine` и `welding fuel`.
`parent`: От какого реагента наследовать свойства. Используется для таких вещей, как BaseDrink.
`desc`: Понятное человеку описание химического вещества.
`color`: Цвет реагента в hex. Используется для смешивания и отображения химического вещества в контейнерах.
`spritePath`: Спрайт, задающий иконку, используемую для реагента, начиная с `Textures/Objects/Consumable/Drinks`.
`physicalDesc`: Расплывчатое описание, потенциально полезное при его идентификации.
`boilingPoint`: Температура в градусах Цельсия, при которой реагент переходит из жидкости в газ.
`meltingPoint`: Температура в градусах Цельсия, при которой реагент переходит из твёрдого состояния в жидкое.
`metabolisms`: Словарь MetabolismGroup->List\<ReagentEffect\>, применяемый при потреблении реагента.

## Пример определения реагента

```yaml
- type: reagent
  id: Lemonade
  name: lemonade
  parent: BaseDrink
  desc: Drink using lemon juice, water, and a sweetener such as cane sugar or honey.
  physicalDesc: tart
  color: "#FFFF00"
  spritePath: lemonadeglass.rsi
  metabolisms:
    Drink:
      effects:
      - !type:SatiateThirst
        factor: 2
```

Дополнительную информацию смотрите: [SS14 Content: папка реагентов](https://github.com/space-wizards/space-station-14/tree/master/Resources/Prototypes/Reagents) или [SS14 Content:ReagentPrototype.cs](https://github.com/space-wizards/space-station-14/blob/ca50a5f9934a399826306659f298f0098251e4eb/Content.Shared/Chemistry/Reagent/ReagentPrototype.cs)

## Эффекты и условия реагентов

`ReagentEffect` описываются на C#. Например, `SatiateThirst` и `HealthChange`. Это позволяет легко переиспользовать код с похожими эффектами. Многие вещи используют эффекты реагентов — метаболизмы, метаболизмы растений и реакции сущностей-реагентов (эффекты на основе касания/инъекции). У `ReagentEffect` также есть поле `prob` — шанс, что эффект сработает, от 0 до 1. Причина, по которой они настолько обобщены, в том, что многие вещи, которым нужны эти данные, могут их использовать.

`ReagentEffectCondition` прикрепляются к каждому эффекту в списке метаболизатора. По умолчанию их нет, поэтому эффекты выполняются всегда. Однако можно легко добавить пользовательское поведение, чтобы определить, когда эффект действительно произойдёт. Например, `ReagentThreshold` позволяет очень легко задать поведение при передозировке. Вот метамфетамин:

```yml
  metabolisms:
    Poison:
      effects:
      - !type:HealthChange
        damage:
          types:
            Poison: 2.5
      - !type:HealthChange
        conditions:
        - !type:ReagentThreshold
          min: 10
        damage:
          types:
            Poison: 4 # это добавляется к базовому урону метамфетамина.
    Narcotic:
      effects:
      - !type:MovespeedModifier
        walkSpeedModifier: 1.3
        sprintSpeedModifier: 1.3
```

Это означает, что поведение передозировки HealthChange срабатывает только когда в вашей системе как минимум 10u. У него две разные группы, «Poison» и «Narcotic». Обе они, как правило, метаболизируются сердцем.
