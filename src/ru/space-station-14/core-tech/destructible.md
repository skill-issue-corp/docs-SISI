# Разрушаемые

## Как сделать сущность разрушаемой
Сделать сущность разрушаемой можно, добавив ей компоненты Damageable и Destructible в YAML.
Компонент Destructible отвечает за определение списка порогов, которые у него есть, каждый с триггером (когда он сработает) и списком поведений (что происходит при срабатывании).

```yaml=
- type: entity
  id: Wall
  name: wall
  description: Keeps the air in and the greytide out.
  components:
  - type: Damageable
    resistances: metallicResistances
  - type: Destructible
    thresholds: # Список порогов, которых может достичь эта сущность
    # Первый и единственный порог, задающий триггер и список поведений
    - trigger: # Триггер этого порога
        !type:DamageTrigger # Срабатывает при суммарном уроне...
        damage: 300 # ... равном или превышающем 300
      behaviors: # Поведения этого порога
      - !type:SpawnEntitiesBehavior # Первое поведение, спавнит сущности
        spawn:
          Girder: # Спавнит каркасы...
            min: 1 # ... от минимум 1...
            max: 1 # ... до максимум 1
      - !type:DoActsBehavior # Второе поведение, активирует список действий
        acts: ["Destruction"] # В данном случае действие Destruction
```

## Как добавить новый триггер разрушения
Все триггеры реализуют интерфейс `IThresholdTrigger`.
Новые можно определить так:

```csharp=
[DataDefinition]
public partial class DamageTrigger : IThresholdTrigger
{
    /// <summary>
    ///     Количество урона, при котором сработает этот порог.
    /// </summary>
    [DataField]
    public int Damage { get; set; }

    public bool Reached(IDamageableComponent damageable, DestructibleSystem system)
    {
        return damageable.TotalDamage >= Damage;
    }
}
```

## Как добавить новое поведение при разрушении
Все поведения реализуют интерфейс `IThresholdBehavior`.
Новые можно определить так:

```csharp=
[DataDefinition]
public partial class PlaySoundBehavior : IThresholdBehavior
{
    /// <summary>
    ///     Звук, проигрываемый при разрушении.
    /// </summary>
    [DataField]
    public string Sound { get; set; } = string.Empty;

    public void Execute(IEntity owner, DestructibleSystem system)
    {
        if (string.IsNullOrEmpty(Sound))
        {
            return;
        }

        var pos = owner.Transform.Coordinates;
        SoundSystem.Play(Filter.Pvs(pos), Sound, pos, AudioHelpers.WithVariation(0.125f));
    }
}
```
