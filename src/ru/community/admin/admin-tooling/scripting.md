# Руководство по скриптам

Видели когда-нибудь тех администраторов, которые наблюдают, а затем сразу уменьшаются или растут, окрашиваются и начинают очень быстро летать вокруг — и всё это одновременно? Мы учимся писать скрипты, детка.

Скрипт — это последовательность команд, которые вы заставляете игру выполнить. Вы можете добавить столько команд, сколько захотите. У скриптов есть множество вариантов применения, и они отлично подходят для автоматизации задач, которые не требуют редактирования между использованиями.

Для начала введите %appdata% в строке поиска и прокрутите вниз, чтобы найти Space Station 14. Если вы уже потерялись, расположение каталога — \AppData\Roaming\Space Station 14\data. Если вы не на Windows, вам придётся найти эту папку самостоятельно. В любом случае откройте папку data и создайте текстовый файл (.txt). Подойдёт буквально любое имя, но вы будете вводить это имя в консоли, так что убедитесь, что сможете его узнать. Вы можете создать сколько угодно скриптов. Поместите свой скрипт в этот файл. Когда у вас есть все нужные команды в скрипте, выполните exec с /filename, чтобы запустить скрипт. У exec есть автозаполнение.

## Примеры
### Базовый скрипт aGhost
Этот скрипт полагается на toolshed для заполнения ID сущности, сначала выполняя "self". Вот «не зависящий от имени пользователя скрипт» от nikthechampiongr:

> self not prototyped AdminObserver do "aghost"
> self do "vvwrite entity/$ID/MovementSpeedModifier/BaseSprintSpeed 25"
> self do "vvwrite entity/$ID/MovementSpeedModifier/BaseWalkSpeed 6"
> self do "vvwrite entity/$ID/Description \"GHOST GANG!\""
> self do "vvwrite entity/$ID/Eye/VisibilityMask 7"
> self do "vvwrite entity/$ID/Ghost/color '#4D7AFF'"
> self do "addcomp $ID ShowCriminalRecordIcons"
> self do "addcomp $ID ShowJobIcons"
> self do "addcomp $ID ShowMindShieldIcons"
> self do "addcomp $ID ShowSyndicateIcons"

### Предметизация сущности
Вот скрипт от aquif, который предметизирует отмеченную сущность (которую нужно отметить, нажав правой кнопкой на сущность > admin > mark):

> marked comp:ensure Item
> marked comp:ensure MultiHandedItem
> marked comp:ensure CanEscapeInventory

Для этого скрипта вам следует добавить размер сущности. Эта команда меняется в зависимости от того, какой тип инвентаря у вашего сервера:

Размер для серверов со списковым инвентарём: > marked do "vvwrite /entity/$ID/Item/Size 120"
Размер для серверов с сеточным инвентарём: > marked do "vvwrite /entity/$ID/Item/Size Normal" для предмета размером 2x2 тайла, или используйте Ginormous, чтобы он был слишком большим для сумок. Просмотрите компонент Item других предметов, чтобы узнать больше размеров.

### Сделать всех призраков радужными

Это одна команда, однако превращение её в скрипт заставляет нас писать гораздо меньше слов, чем написание всей команды целиком (например: exec /RGBALL.txt)

> entities with Ghost comp:ensure RgbLightController comp:ensure PointLight do "vvwrite /entity/$ID/PointLight/Energy 0"
