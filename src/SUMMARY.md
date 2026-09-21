Вики разработки Space Wizards
=====================

[Книга Robust](index.md)

Гитхаб страницы
====

----------------------
- [Руководство по редактированию документации](ru/meta/guide-to-editing-docs.md)
- [Пример страницы документации](ru/meta/docs-example-page.md)
- [Документация нужна для обнаруживаемости](ru/meta/docs-are-for-discoverability.md)

Общая разработка
===================

----------------------

- [Настройка](ru/general-development/setup.md)
  - [Как мне программировать?](ru/general-development/setup/howdoicode.md)
  - [Настройка среды разработки](ru/general-development/setup/setting-up-a-development-environment.md)
  - [Git для разработчика SS14](ru/general-development/setup/git-for-the-ss14-developer.md)
  - [Руководство по хостингу сервера](ru/general-development/setup/server-hosting-tutorial.md)
- [Информация о кодовой базе](ru/general-development/codebase-info.md)
  - [Соглашения](ru/general-development/codebase-info/conventions.md)
  - [Руководство по pull request'ам](ru/general-development/codebase-info/pull-request-guidelines.md)
  - [Организация кодовой базы](ru/general-development/codebase-info/codebase-organization.md)
  - [Аббревиатуры и номенклатура](ru/general-development/codebase-info/acronyms-and-nomenclature.md)
  - [Процесс релиза SS14](ru/general-development/codebase-info/releases.md)
- [Советы](ru/general-development/tips.md)
  - [FAQ для новичков](ru/general-development/tips/beginner-faq.md)
  - [FAQ по устранению неполадок](ru/general-development/tips/troubleshooting-faq.md)
  - [Инструменты отладки](ru/general-development/tips/debugging-tools.md)
  - [PR с изменениями движка](ru/general-development/tips/prs-with-engine-changes.md)
  - [Написание записей руководства](ru/general-development/tips/writing-guidebook-entries.md)
  - [Справочник по файлам конфигурации](ru/general-development/tips/config-file-reference.md)
  - [Ускоренный курс по YAML](ru/general-development/tips/yaml-crash-course.md)
  - [Советы по форкам](ru/general-development/tips/forking.md)

SS14 на примерах
===============

----------------------

- [Введение в SS14 на примерах](ru/ss14-by-example/introduction-to-ss14-by-example.md)
- [Добавление простого велосипедного клаксона](ru/ss14-by-example/adding-a-simple-bikehorn.md)
- [Как сделать спрайт динамическим](ru/ss14-by-example/making-a-sprite-dynamic.md)
  - [Перенос визуализаторов внешнего вида](ru/ss14-by-example/making-a-sprite-dynamic/porting-appearance-visualizers.md)
- [Основы сетевого взаимодействия](ru/ss14-by-example/basic-networking-and-you.md)
- [Руководство по предсказанию](ru/ss14-by-example/prediction-guide.md)
- [Fluent и локализация](ru/ss14-by-example/fluent-and-localization.md)
- [UI и вы](ru/ss14-by-example/ui-and-you.md)
  - [Кулинарная книга UI](ru/ss14-by-example/ui-and-you/ui-cookbook.md)
- [Руководство по выживанию в UI](ru/ss14-by-example/ui-survival-guide.md)
- [Перевод Oldbody на Nubody](ru/ss14-by-example/converting-oldbody-to-nubody.md)

Robust Toolbox
==============

----------------------

- [ECS](ru/robust-toolbox/ecs.md)
- [Сетевой код]()
  - [Сетевые сущности](ru/robust-toolbox/netcode/net-entities.md)
  - [Последовательность подключения](ru/robust-toolbox/netcode/connection-sequence.md)
- [Системы координат](ru/robust-toolbox/coordinate-systems.md)
- [Трансформация]()
  - [Координаты сущностей](ru/robust-toolbox/transform/entity-coordinates.md)
  - [Физика](ru/robust-toolbox/transform/physics.md)
  - [Сетки](ru/robust-toolbox/transform/grids.md)
- [Toolshed](ru/robust-toolbox/toolshed.md)
  - [Переменные](ru/robust-toolbox/toolshed/variables.md)
  - [Блоки](ru/robust-toolbox/toolshed/blocks.md)
  - [Типы](ru/robust-toolbox/toolshed/types.md)
  - [Команды](ru/robust-toolbox/toolshed/commands.md)
    - [Emplace и Do](ru/robust-toolbox/toolshed/commands/emplace.md)
    - [Сущности](ru/robust-toolbox/toolshed/commands/entity-control.md)
    - [Общие](ru/robust-toolbox/toolshed/commands/general.md)
    - [Разное](ru/robust-toolbox/toolshed/commands/misc.md)
  - [Примеры Toolshed](ru/robust-toolbox/toolshed/toolshed-examples.md)
  - [Разработка](ru/robust-toolbox/toolshed/development.md)
    - [Toolshed и (S)CSI](ru/robust-toolbox/toolshed/toolshed-and-scsi.md)
    - [Окружения](ru/robust-toolbox/toolshed/environments.md)
    - [Контексты вызова](ru/robust-toolbox/toolshed/invocation-contexts.md)
- [Пользовательский интерфейс](ru/robust-toolbox/user-interface.md)
  - [Контейнеры](ru/robust-toolbox/user-interface/containers.md)
- [IoC](ru/robust-toolbox/ioc.md)
- [Рендеринг]()
  - [Освещение и FOV](ru/robust-toolbox/rendering/lighting-and-fov.md)
  - [Шейдеры](ru/robust-toolbox/rendering/shaders.md)
  - [Спрайты и иконки](ru/robust-toolbox/rendering/sprites-and-icons.md)
- [Сериализация](ru/robust-toolbox/serialization.md)
- [Песочница](ru/robust-toolbox/sandboxing.md)
- [Манифесты контента](ru/robust-toolbox/content-manifests.md)
- [Каталог пользовательских данных](ru/robust-toolbox/user-data-directory.md)
- [Модули Robust](ru/robust-toolbox/robust-modules.md)
- [HTTP API сервера](ru/robust-toolbox/server-http-api.md)
- [Конфигурации сборки](ru/robust-toolbox/build-configurations.md)
- [Препроцессорные определения](ru/robust-toolbox/preprocessor-defines.md)
- [MIDI](ru/robust-toolbox/midi.md)
- [Automatic Client Zip (ACZ)](ru/robust-toolbox/acz.md)
- [Упаковка ассетов](ru/robust-toolbox/asset-packaging.md)
- [Публикация новой версии Robust Toolbox](ru/robust-toolbox/publishing-robusttoolbox.md)
- [Версионирование и совместимость](ru/robust-toolbox/versioning-compatibility.md)

Space Station 14
================

----------------------

- [Базовые технологии]()	
	- [Разрушаемые](ru/space-station-14/core-tech/destructible.md)
	- [Строительство](ru/space-station-14/core-tech/construction.md)
	- [Сети узлов](ru/space-station-14/core-tech/node-networks.md)
	- [Карты смещения](ru/space-station-14/art/displacement-maps.md)
	- [Сеть устройств](ru/space-station-14/core-tech/device-network.md)
	- [NPC](ru/space-station-14/core-tech/npcs.md)
	- [Таблицы сущностей](ru/space-station-14/core-tech/entity-tables.md)
	- [Тело](ru/space-station-14/core-tech/body.md)
	- [Химия](ru/space-station-14/core-tech/chemistry.md)
		- [Метаболизм](ru/space-station-14/core-tech/chemistry/metabolism.md)
		- [Реакции](ru/space-station-14/core-tech/chemistry/reactions.md)
		- [Реагенты](ru/space-station-14/core-tech/chemistry/reagents.md)
		- [Контейнеры растворов](ru/space-station-14/core-tech/chemistry/solution-containers.md)

	- [Предложения]()

- [Арт](ru/space-station-14/art.md)

- [Маппинг](ru/space-station-14/mapping.md)
	- [Рекомендации по PR](ru/space-station-14/mapping/guidelines.md)
	
	- [Подземелья](ru/space-station-14/mapping/dungeons.md)
	
	- [Руководства]()
		- [Общее руководство](ru/space-station-14/mapping/guides/general-guide.md)

	- [Предложения]()

Общие предложения
================

----------------------

- [RobustHub](ru/general-proposals/robusthub.md)

Хостинг серверов
==============

----------------------

- [Проброс портов](ru/server-hosting/port-forwarding.md)
- [Запись повторов сервера](ru/server-hosting/server-replay-recording.md)
- [Настройка Robust.Cdn](ru/server-hosting/setting-up-robust-cdn.md)
- [Настройка SS14.Admin](ru/server-hosting/setting-up-ss14-admin.md)
- [Настройка SS14.Changelog](ru/server-hosting/setting-up-ss14-changelog.md)
- [Настройка SS14.Watchdog](ru/server-hosting/setting-up-ss14-watchdog.md)
- [Настройка передачи с высокой пропускной способностью](ru/server-hosting/setting-up-high-bandwidth-transfer.md)
- [OAuth](ru/server-hosting/oauth.md)
- [Настройка интеграции с Discord](ru/server-hosting/setting-up-discord-integration.md)
- [Обслуживание]()
  - [Отладка зависаний сервера](ru/server-hosting/maintenance/debugging-server-lockups.md)

Другие проекты
==============

----------------------

- [Лаунчер]()
  - [Пакеты контента](ru/other-projects/launcher/content-bundles.md)
  - [Дельта-обновления и манифесты](ru/other-projects/launcher/delta-updates-and-manifests.md)

Спецификации
==============

----------------------

- [Robust Station Image](ru/specifications/robust-station-image.md)
- [Robust Generic Attribution](ru/specifications/robust-generic-attribution.md)

Сообщество
========================

----------------------

- [Ссылки по инфраструктуре](ru/community/infrastructure-references.md)
  - [Инфраструктура Wizard's Den](ru/community/infrastructure-reference/wizards-den-infrastructure.md)
  - [Дашборды Grafana](ru/community/infrastructure-reference/grafana-dashboards.md)
- [Правила хаба Space Wizards](ru/community/space-wizards-hub-rules.md)
- [Репозиторий Discord Rich Presence](ru/community/discord-rich-presence-repository.md)
- [Администрирование](ru/community/admin.md)
  - [Инструменты администратора](ru/community/admin/admin-tooling.md)
    - [Админ-кулинарная книга](ru/community/admin/admin-tooling/admin-command-cookbook.md)
	- [Скриптинг](ru/community/admin/admin-tooling/scripting.md)

Разработка движка
========================

------------------

- [Тестирование с лаунчером](ru/engine-development/testing-against-launcher.md)
- [Обновление CEF](ru/engine-development/updating-cef.md)
