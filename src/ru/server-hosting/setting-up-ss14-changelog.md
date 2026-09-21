# Настройка SS14.Changelog

## Основная настройка

Смотрите репозиторий: https://github.com/space-wizards/SS14.Changelog

## RSS-лента

Система публикации может автоматически публиковать список изменений в RSS-ленте. Это делается из [`actions_changelog_rss.py`](https://github.com/space-wizards/space-station-14/blob/master/Tools/actions_changelog_rss.py), который запускается из [рабочего процесса публикации](https://github.com/space-wizards/space-station-14/blob/master/.github/workflows/publish.yml).

### Настройка сервера

Итоговый RSS-файл должен размещаться на отдельном сервере с доступом по SFTP. Рекомендую настроить пользователя с доступом только по SFTP и изоляцией chroot. Если вы знаете, как это сделать, вероятно, вам не нужно особо в это вчитываться.

```shell
# Создаём группу для пользователей только с SFTP
groupadd sftp-only
# Создаём пользователя для списка изменений
useradd changelog-rss --groups sftp-only --create-home

# Создаём каталог для chroot-сессии SFTP
mkdir --parents /var/sftp/home/changelog-rss
chown changelog-rss: /var/sftp/home/changelog-rss

# Настраиваем SSH-ключ для пользователя
mkdir /home/changelog-rss/.ssh
ssh-keygen -t ed25519 -f changelog_key -N ""
cat changelog_key.pub >> /home/changelog-rss/.ssh/authorized_keys
# Сохраните changelog_key, он понадобится для настройки на стороне GitHub.
chown -R changelog-rss: /home/changelog-rss/.ssh

# (пример, зависит от того, какой веб-сервер вы хотите использовать)
# Дайте nginx доступ к каталогу загрузок
setfacl -m u:nginx:rx /var/sftp/home/changelog-rss
```

Вам также потребуется добавить следующее в `/etc/ssh/sshd_config`:

```
Subsystem sftp internal-sftp

Match Group sftp-only
    ChrootDirectory /var/sftp
    X11Forwarding no
    AllowTcpForwarding no
    AllowAgentForwarding no
    ForceCommand internal-sftp
```

Вам всё ещё нужно убедиться, что файл списка изменений где-то доступен через веб-сервер. Это я оставлю на ваше усмотрение. Что бы вы ни делали, убедитесь, что ваш веб-сервер указывает `Content-Type` файла как `application/rss+xml`.

```admonish warning
SSH-ключ ДОЛЖЕН быть ed25519. Мне было лень делать скрипт более гибким.
```

### Настройка GitHub

В официальном рабочем процессе публикации `actions_changelog_rss.py` автоматически начинает выполняться, как только на GitHub задаётся секрет `CHANGELOG_RSS_KEY`. Это должен быть приватный ключ, который позволит вам подключаться по SFTP.

Однако прежде чем задавать его, есть ещё несколько изменений, которые следует внести в `actions_changelog_rss.py`:

```python
# Измените их под настройки вашего сервера
# https://docs.fabfile.org/en/stable/getting-started.html#run-commands-via-connections-and-run
SSH_HOST = "centcomm.spacestation14.io"
SSH_USER = "changelog-rss"
SSH_PORT = 22
RSS_FILE = "changelog.xml"
HOST_KEYS = [
    "AAAAC3NzaC1lZDI1NTE5AAAAIEE8EhnPjb3nIaAPTXAJHbjrwdGGxHoM0f1imCK0SygD"
]

# Параметры RSS-ленты, измените их
FEED_TITLE       = "Space Station 14 Changelog"
FEED_LINK        = "https://github.com/space-wizards/space-station-14/"
FEED_DESCRIPTION = "Changelog for the official Wizard's Den branch of Space Station 14."
FEED_LANGUAGE    = "en-US"
FEED_GUID_PREFIX = "ss14-changelog-wizards-"
FEED_URL         = "https://central.spacestation14.io/changelog.xml"
```

* Параметры `SSH_` следует задать такими, какие нужны вам для подключения по SSH.
* `RSS_FILE` — это имя файла назначения в каталоге SFTP.
* `HOST_KEYS` должен содержать ed25519 host-ключ вашей целевой системы. Его можно найти в `/etc/ssh/ssh_host_ed25519_key.pub`. Отрежьте начальный фрагмент `ssh-ed25519`.
* Параметры `FEED_` меняют содержимое RSS-ленты. Вероятно, их стоит изменить, чтобы они отличались от используемых Wizard's Den.

### Пользовательские данные XML

Созданная RSS-лента содержит как обычное HTML-описание изменений, так и более структурированную информацию из исходного списка изменений, которую могут отображать специализированные инструменты. Все эти данные находятся в пространстве имён XML `https://spacestation14.com/changelog_rss`.

Краткая сводка данных:
* Элемент `<item>` содержит `ss14:from-id` и `ss14:to-id`, описывающие диапазон списков изменений, охватываемых элементом RSS.
* Каждый элемент RSS может содержать серию элементов `<ss14:entry>` с записями, из которых состоит элемент RSS.
