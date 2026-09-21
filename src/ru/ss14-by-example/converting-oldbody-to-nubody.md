# КАК ПЕРЕВЕСТИ РАСУ НА NUBODY

от mqole

```admonish info
Это неофициальное и неполное руководство. В будущем оно будет обновлено и заменено официальной документацией.
```

Всё это — догадки, основанные на реверс-инжиниринге nubody. если что-то неверно, то потому что я не понимаю, как работает nubody.

ПРИМЕЧАНИЕ: когда я говорю Species в имени сущности (например, AppearanceSpecies, MobSpecies), я ожидаю, что это будет заменено фактическим названием расы (например, AppearanceDwarf, MobDwarf).

## КРАТКИЙ ЧЕК-ЛИСТ

Body/Species/species.yml должен содержать:

- [] MarkingsGroup
- [] AppearanceSpecies, изменённый из Body/Prototypes/species.yml
- [] MobSpecies, изменённый из Entities/Mobs/Species/species.yml
- [] Внешние органы (ранее «части»), изменённые из Body/Organs/species.yml
- [] Внутренние органы, изменённые из Body/Parts/species.yml

Затем вы можете удалить все файлы, из которых вы что-то перенесли:

- [] Body/Prototypes/species.yml
- [] Entities/Mobs/Species/species.yml
- [] Body/Organs/species.yml
- [] Body/Parts/species.yml

Species/species.yml должен содержать:

- [] Species. и всё

При изменении Species/species.yml следует:

- [] Удалить `sprites` расы
- [] Удалить `markingLimits` расы
- [] Заменить сущность `dollPrototype` `MobSpeciesDummy` на сущность `AppearanceSpecies`
- [] Удалить `speciesBaseSprites`
- [] Удалить `humanoidBaseSprite`
- [] Вырезать `markingPoints`, вставить в начало Body/Species/species.yml. Это наша новая MarkingsGroup

Чтобы обновить маркировки:

- [] Удалить `markingCategory`
- [] Удалить `followSkinColor`
- [] Переименовать `speciesRestriction` в `groupWhitelist`.

## Маркировки

Она наследуется от markingsGroup `Undergarments` и определяет ограничения того, какие маркировки может иметь раса. числовые ограничения, bool-требования и маркировки по умолчанию имеют более или менее тот же синтаксис, что и прежние `MarkingPoints` — просто замените визуальные слои гуманоида на `enum.HumanoidVisualLayers.`whatever и измените `points` на `limit`.

Вам нужно будет разделить руки и ноги на LArm RArm и т. д. и т. п. Ограничения на них были уменьшены вдвое для компенсации. К сожалению, я не уверен, как обрабатывать расы, которые используют одну маркировку для изменения обеих рук или ног сразу.

## Внешний вид расы

Насколько я могу судить, это заменяет систему, соединяющую части тела друг с другом. Эта сущность должна наследоваться от `BaseSpeciesAppearance` и содержать информацию о `InventoryComponent`, `InitialBodyComponent` и `HumanoidProfile`.

Поле `organs` у `InitialBodyComponent` содержит словарь типов органов, соответствующих внутренним или внешним органам моба. Если вам нужно определить новый тип органа, вы можете сделать это где-нибудь в отдельном yml-файле, во многом как в прежней системе.

`HumanoidProfile` — замена `HumanoidAppearance`. Просто измените имя и дело с концом.

## Собственно моб

Должен наследоваться от `BaseSpeciesMobOrganic` и соответствующего внешнего вида расы. Всё со старого моба идёт сюда, кроме того, что вы перенесли во внешний вид. Убедитесь, что вы избавились именно от `InventoryComponent`.

## Органы

Итак, части теперь тоже органы. Мы называем части «внешними органами», в отличие от существующих органов, которые мы называем «внутренними органами».

Вам понадобятся:

- [] OrganSpecies, наследующий OrganBase, с суффиксом расы
- [] OrganSpeciesMetabolizer с MetabolizerComponent. Возможно, вам это не понадобится.
- [] OrganSpeciesInternal (мозг, глаза и т. д.), наследующий OrganSpecies, путь к спрайту organs.rsi
- [] OrganSpeciesExternal (голова, ступня и т. д.), наследующий OrganSpecies, путь к спрайту parts.rsi
- [] OrganSpeciesVisual, наследующий OrganSpecies, с `VisualOrganComponent` и `VisualOrganMarkingsComponent`.

Все создаваемые вами органы (внутренние и внешние) должны наследоваться либо от OrganSpeciesExternal, либо от OrganSpeciesInternal в зависимости от того, что вам нужно, а также наследовать базовый орган для нужной цели (OrganBaseFootLeft, OrganBaseAppendix и т. д., следуйте этому синтаксису). Поместите в них нужные вам компоненты, как вы делали бы для прежних органов.

Если у вас уникальный желудок, он должен наследоваться от OrganSpeciesMetabolizer. Если вы просто используете существующий желудок, вам не нужно создавать новый OrganSpeciesMetabolizer — просто наследуйте его от той расы, чью группу метаболизма вы используете.

У OrganSpeciesVisual `VisualOrganComponent` должен указывать на parts.rsi, а `VisualOrganMarkings` — на созданную нами ранее группу маркировок (может быть 'None`). Глаза (и другие внутренние органы, влияющие на маркировки, если они у вас есть, полагаю) также должны наследоваться от OrganSpeciesVisual.

## Маркировки

Вы можете просто удалить `markingCategory` и `followSkinColor`, переименовать `speciesRestriction` в `groupWhitelist` — и всё должно быть в порядке. К сожалению, в upstream нет ничего с пользовательскими визуальными слоями, так что тут я в тупике.

## Пользовательские порядки слоёв

не спрашивай меня, брат, я сам едва понимаю эту херню
