# Админ-кулинарная книга

## Удалить УЧ и генератор сингулярности
```
    entities named "PA .*" do delete $ID
    deleteewi SingularityGenerator
```
## Навсегда добавить питание станции
`Admin menu -> objects -> select grids in dropdown -> right click the station -> tricks -> click the battery with the ∞`
## Снять проклятие со всех шкафчиков
`entities with CursedEntityStorage do rmcomp $ID CursedEntityStorage; addcomp $ID EntityStorage`
## Удалить все роли призраков
`entities with GhostTakeoverAvailable do rmcomp $ID GhostTakeoverAvailable; rmcomp $ID Mind`
## Убрать разные части тела
`entities with MapGrid children with BodyPart do delete $ID`
## Разоружить запас калия на станции
`entities hasreagent Potassium do delete $ID`
## Удалить все изолированные перчатки конкретного персонажа
`entities with Mind named "NAME GOES HERE" rchildren named "insulated gloves" do delete $ID`

# Роли призраков
## Разумные торговые автоматы
`entities with Advertise do makeghostrole $ID "$NAME" "You're a vending machine, use your speaker to annoy people."; rmcomp $ID Advertise`
## Разумное мыло
`entities with Slippery prototyped SoapOmega do makeghostrole $ID "$NAME" "You're a bar of soap. Slip absolutely everyone."; addcomp $ID MobState; addcomp $ID PlayerMobMover; addcomp $ID PlayerInputMover`
## Пожалуйста, не надо
`entities with Item do makeghostrole $ID "$NAME" "You're a random, talking item. What the fuck?"; addcomp $ID MobState; addcomp $ID PlayerMobMover; addcomp $ID PlayerInputMover`
## Твоё сердце хочет кое-что сказать.
`entities with Mechanism do makeghostrole $ID "$NAME" "You are some unfortunate soul's $NAME"`
## Гостулярность
`entities with Singularity do makeghostrole $ID "Singularity" "FUCK"; addcomp $ID MobState; addcomp $ID MovementIgnoreGravity; addcomp $ID PlayerInputMover; addcomp $ID PlayerMobMover`

# Вероятно, сделать всех несчастными
## Злоупотребление проклятыми шкафчиками
### Проклятые шкафчики
`entities with EntityStorage named ".*closet$|^.*locker" do rmcomp $ID EntityStorage; addcomp $ID CursedEntityStorage`
### Шкафчики с привидениями
`entities with EntityStorage named ".*closet$|^.*locker" do rmcomp $ID EntityStorage; addcomp $ID CursedEntityStorage; makeghostrole $ID "$NAME" "You're a haunted locker. Consume people."; addcomp $ID MobState; addcomp $ID PlayerMobMover; addcomp $ID PlayerInputMover`
## Играться с доступом на станции
### День полного доступа
`entities with AccessReader do rmcomp $ID AccessReader`
### Полный доступ к мостику на Saltern
`entities with Airlock named "Bridge" near 6 with Airlock do rmcomp $ID AccessReader`
## Поменять должности/одежду местами
### День клоуна
`entities with Mind prototyped MobHuman do setoutfit $ID ClownGear; addcomp $ID Clumsy`
### Клоунит
`entities with Clumsy near 1 with Body prototyped MobHuman not with Clumsy do setoutfit $ID ClownGear; addcomp $ID Clumsy`
### Все, кроме нас двоих, — клоуны, кто это
`entities with Mind alive prototyped MobHuman not with Clumsy not select 2 do setoutfit $ID ClownGear; addcomp $ID Clumsy`
### День мима
`entities with Mind prototyped MobHuman do setoutfit $ID MimeGear; rmcomp $ID Speech`
## Контейнеры
### Удалить чьи-то лёгкие
`entities with Body prototyped MobHuman named "NAME GOES HERE" rchildren named "lungs" do delete $ID`
или уронить их на пол:
`entities with Body prototyped MobHuman named "NAME GOES HERE" rchildren named "lungs" do rmmechanism $ID`
## Играться со структурой станции
### Сделать станцию прозрачной
`entities named ".*wall" do spawn ReinforcedWindow $ID; spawn Grille $ID; spawn CableApcExtension $ID; delete $ID`
### Электрический проспект
`entities named ".*wall" do spawn spawn Grille $ID; spawn CableApcExtension $ID`

# Сделать всех несчастными
## Призвать Бога
`entities with Body prototyped MobHuman alive select 1 do addcomp $ID Singularity; addcomp $ID MovementIgnoreGravity; godmode $ID`
## бабах
`entities with MapGrid rchildren do explode $WX $WY 1 1 1 1`
## бабах, но смешнее
`entities with MapGrid children with Item not anchored do addcomp $ID RoguePointingArrow`
