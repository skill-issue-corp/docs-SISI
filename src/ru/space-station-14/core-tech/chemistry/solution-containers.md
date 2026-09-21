# Контейнеры растворов

Чтобы сделать сущность контейнером раствора, добавьте ей `SolutionContainerManager`. Например, вот пустой раствор желудка:
```yaml
  - type: SolutionContainerManager
    solutions:
      stomach:
      	maxVol: 200
```

а вот полный раствор напитка: 

```yaml
  - type: SolutionContainerManager
    solutions:
      drink:
        maxVol: 20
        reagents:
        - ReagentId: Cola
          Quantity: 20
```

`type`: Тип компонента. Всегда должен быть `SolutionContainerManager`.
`solutions`: Словарь из имени раствора и `Solution`, описанного ниже.

У растворов есть несколько полей, все они необязательны.
`maxVol`: Максимальный объём раствора, который он может вместить. Как только сумма Quantity реагентов достигает `maxVol`, больше реагентов добавить нельзя. Он полон.
`reagents`: Список `Reagent`, которые он должен содержать по умолчанию. Каждый `Reagent` имеет строковый `ReagentId` и `Quantity`, который является FixedPoint2 (по сути float, ограниченный двумя знаками после запятой).

## Возможности и указание целевого раствора

С появлением `SolutionContainerManager` больше нет простого отображения 1:1 между сущностью и списком реагентов. 
Каждый `SolutionContainerManager` может содержать любое количество растворов. Чтобы решить эту проблему, вместо флага Capabilities введён набор компонентов, похожих на возможности. Каждая такая возможность имеет строковое поле `solution`, которое сообщает `SolutionContainerManager`, на какой раствор она нацелена.

Вот их список:
- `DrainableSolutionComponent` для растворов, которые можно легко извлечь через любой контейнер для реагентов. Например, слив воды из бака с водой
- `DrawableSolutionComponent` для растворов, которые можно набирать шприцами. Например, люди или флаконы для шприцев с резиновыми крышками.
- `ExaminableSolutionComponent` для растворов, которые можно рассмотреть вручную.
- `FitsInDispenserComponent` отмечает, что компонент помещается в дозатор, и сообщает ReagentDispenser/ChemMaster, на какой раствор нацелиться.
- `InjectableSolutionComponent` для растворов, в которые можно делать инъекции шприцами. 
- `RefillableSolutionComponent` для растворов, которые можно легко пополнить.

Другие компоненты, такие как `DrinkComponent` или `FoodComponent`, могут иметь специальные предопределённые растворы, на которые они нацелены и которые ожидают увидеть. Например, `DrinkComponent` попытается создать раствор `drink`, если ещё нет легкодоступного `DrainableSolutionComponent`.

## Пример раствора
Вот полный пример сущности с несколькими растворами:

```yaml
- type: entity
  id: DrinkColaCan
  name: space cola
  description: A refreshing beverage.
  components:
  - type: Sprite
    sprite: Objects/Consumable/Drinks/cola.rsi
  - type: SolutionContainerManager
    solutions:
      drink:
        reagents:
        - ReagentId: Cola
          Quantity: 20
        maxVol: 20
  - type: Drink    
```