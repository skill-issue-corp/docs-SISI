# Создание отчёта о прогрессе
*Краткое руководство по созданию отчёта о прогрессе.*

### Контрольный список
- Укажите явные дату начала и дату окончания вкладов, которые хотите описать.
- Создайте доску Trello на основе шаблона Trello для отчёта о прогрессе.
- Запустите [инструмент](https://github.com/space-wizards/github2trello), чтобы сгенерировать карточки Trello для каждого пул-реквеста Content/Engine. Учтите, что пул-реквесты Content без changelog будут пропущены.
- Распределите карточки по их колонкам на своё усмотрение.
- Напишите описания для каждой карточки.
- Объедините содержимое карточек в markdown-файл на основе шаблона Markdown для отчёта о прогрессе, но обязательно обновите номер PR.
- Попросите у PJB список патронов.
- Добавьте список участников, см. ниже.
- Замените или удалите имена, как указано [здесь](https://github.com/space-wizards/space-station-14/blob/master/Tools/contribs_shared.ps1).
- Поместите markdown-файл в `website-content/content/post`. Изображения кладутся в новый каталог `website-content/static/images/post/pr_[number]`, а видео идут в новый каталог `website-content/static/video/pr_[number]`.
- Создайте PNG и MP4 для каждого раздела, которому они нужны.
- Запустите [этот скрипт](https://github.com/space-wizards/website-content/blob/master/Tools/pr-image-convert.ps1) в каталоге со всеми PNG. Требуются [ImageMagick](https://imagemagick.org/index.php) и [optipng](http://optipng.sourceforge.net/).
- Сделайте миниатюру (800x450) PNG и поместите её в `website-content/assets/images/thumbnails`.
- Создайте пул-реквест в [репозитории сайта](https://github.com/space-wizards/website-content). Дайте всем возможность проверить его пару дней, затем смёрджите его одновременно с релизом.

### Источники данных
- Список всех участников с помощью следующей команды: `git shortlog -s -n --since=<start> --until=<end>`.
- Количество коммитов: `git rev-list --count master --since=<start> --until=<end>`.

### Где публиковать
- Сайт. Публикуется автоматически при слиянии пул-реквеста в `website-content`.
- Steam (пинайте PJB или Smug).
- Discord.
- Patreon (пинайте PJB).
- Reddit (r/ss13, r/ss14 и r/linux_gaming).
- Twitter (пинайте PJB или Smug).
