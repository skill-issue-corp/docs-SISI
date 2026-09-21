# Настройка SS14.Admin
[SS14.Admin](https://github.com/space-wizards/SS14.Admin/) — это веб-панель для администрирования серверов SS14, предоставляющая различные [возможности](../community/admin/admin-tooling.md#ss14admin), критически важные для серьёзных серверов.

В этом документе объясняется, что понадобится для настройки собственного экземпляра SS14.Admin.

## Обзор

SS14.Admin подключается напрямую к базе данных вашего игрового сервера, что позволяет ему просматривать все данные об игроках, банах, администраторах и т. д. Затем это становится доступно администраторам игрового сервера через их веб-браузер. Администраторы входят в SS14.Admin напрямую со своей учётной записью SS14 через [OAuth](../server-hosting/oauth.md).

## Предварительные требования

- Ваш игровой сервер должен быть настроен на использование **PostgreSQL** в качестве движка базы данных. Использовать SS14.Admin с SQLite невозможно.
- Доменное имя, под которым будет размещён SS14.Admin.
- Сервер обратного прокси, такой как Nginx или Caddy.

## Установка

Мы предоставляем официальные образы контейнеров SS14.Admin через GitHub Container Registry. Кроме того, у нас есть инструкции по ручной сборке проекта через .NET SDK.

### Образ контейнера

Последний стабильный образ SS14.Admin — `ghcr.io/space-wizards/ss14.admin:1`. Информация о контейнере:

* Слушает порт 8080
* UID/GID по умолчанию — 1654
* Важные тома для монтирования:
  * `/app/appsettings.yml`: основной файл конфигурации.

Вот пример `docker-compose.yml`, измените его под свои нужды.

```
services:
  ss14_admin:
    image: ghcr.io/space-wizards/ss14.admin:1
    container_name: ss14_admin
    user: 1654:1654
    volumes:
      - ./appsettings.yml:/app/appsettings.yml
    ports:
      - 8080:8080
    restart: unless-stopped
```
### Ручная компиляция

Если вы ненавидите контейнеры, вы можете вручную опубликовать SS14.Admin и развернуть файлы самостоятельно. Для этого вам понадобятся Git и .NET 10 SDK. На сервере, который будет запускать сборку, должна быть установлена соответствующая ASP.NET Core Runtime, но сам SDK не нужен.

Клонируйте git-репозиторий, затем опубликуйте:

```sh
git clone https://github.com/space-wizards/SS14.Admin.git --recurse-submodules
cd SS14.Admin
dotnet publish -c Release -r linux-x64 --no-self-contained
```

Готовая сборка будет помещена в `SS14.Admin/bin/Release/net9.0/linux-x64/publish`. Вы можете скопировать их в какое-нибудь случайное место по вкусу, например `/opt`, и запускать `SS14.Admin` оттуда. Например:

```
/opt/ss14_admin/
├── appsettings.yml
├── bin
│   ├── SS14.Admin
│   ├── SS14.Admin.dll
    .
```

Сами файлы программы находятся во вложенной папке, а мы запускаем её из родительского каталога, чтобы вы не снесли файлы конфигурации обновлениями или вроде того.

Затем вы можете автоматически запускать SS14.Admin с помощью следующего определения службы systemd:

```ini
# /etc/systemd/system/ss14-admin.service
[Unit]
Description=SS14.Admin

[Service]
Type=notify
WorkingDirectory=/opt/ss14_admin/
ExecStart=/opt/ss14_admin/bin/SS14.Admin
User=ss14_admin

[Install]
WantedBy=multi-user.target
```

## Конфигурация

SS14.Admin — это приложение ASP.NET Core, поэтому оно поддерживает конфигурацию как через файл конфигурации, так и через другие источники, например переменные окружения. Более подробный обзор можно найти в [документации ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-9.0).

Большая часть конфигурации SS14.Admin выполняется через файл конфигурации `appsettings.yml`. Вот полный справочник по его содержимому:

```yaml
Serilog:
    Using: [ "Serilog.Sinks.Console" ]
    MinimumLevel:
        Default: Information
        Override:
            SS14: Debug
            Microsoft: "Warning"
            Microsoft.Hosting.Lifetime: "Information"
            Microsoft.AspNetCore: Warning
            IdentityServer4: Warning
    WriteTo:
        - Name: Console
          Args:
              OutputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3} {SourceContext}] {Message:lj}{NewLine}{Exception}"

    Enrich: [ "FromLogContext" ]

    #Loki:
    #    Address: "http://localhost:3102"
    #    Name: "centcomm"

ConnectionStrings:
    # Подключите это к той же базе данных PostgreSQL, что и ваш сервер SS14
    DefaultConnection: "Server=127.0.0.1;Port=5432;Database=ss14;User Id=ss14-admin;Password=foobar"

# Задайте здесь доменное имя, под которым вы будете размещать SS14.Admin.
AllowedHosts: "ss14-admin.spacestation14.com"

# Если хотите изменить порт веб-сервера, измените его здесь; рекомендую поставить это за обратный прокси для SSL
urls: "http://localhost:27689/"

# Подпуть, под которым будет размещён сайт.
# Можно не указывать, если вы размещаете его под собственным поддоменом.
# PathBase: "/admin"

# Убедитесь, что это указывает на wwwroot, он должен находиться в том же каталоге, что и исполняемый файл
WebRootPath: "/opt/ss14_admin/bin/wwwroot"

# IP-адреса, которым разрешено обратное проксирование сайта.
# Измените это, если ваш обратный прокси приходит не с localhost,
# например, если SS14.Admin работает в контейнере,
# вам следует добавить сюда IP хоста в сети контейнера.
ForwardProxies:
    - 127.0.0.1

# Информация о клиенте OAuth, чтобы администраторы могли аутентифицироваться, см. ниже.
Auth:
    Authority: "https://account.spacestation14.com/"
    ClientId: "9e2ce26f-EDIT-THIS-b4d9-8cc08993b33e"
    ClientSecret: "foobar"

authServer: "https://auth.spacestation14.com"
```

```admonish warning
Из-за того, как работает oauth, для успешного входа потребуется SSL/HTTPS-соединение. Неважно, от центра сертификации вроде Let's Encrypt или самоподписанный. Просто нужно, чтобы ему доверяли, чтобы ваш браузер действительно отправил необходимые данные.
```

## Настройка аутентификации

Чтобы администраторы могли входить напрямую со своей учётной записью SS14, вам нужно зарегистрировать [клиент OAuth](../server-hosting/oauth.md) следующим образом:

1. Войдите и перейдите на https://account.spacestation14.com/Identity/Account/Manage/Developer и нажмите «New OAuth App».
2. Введите имя приложения, это может быть что угодно.
3. Задайте «Authorization callback URL» равным адресу вашего экземпляра с добавленным `/signin-oidc`. Примеры:
  * Если ваш экземпляр доступен по адресу `https://admin.example.com`, укажите `https://admin.example.com/signin-oidc`.
  * Если ваш экземпляр доступен по адресу `https://example.com/admin`, укажите `https://example.com/admin/signin-oidc`.
4. Задайте «Homepage URL» равным основному адресу, по которому доступен ваш экземпляр, например `https://admin.example.com` или `https://example.com/admin`.
5. Скопируйте **Client ID** в `ClientId` в файле конфигурации.
6. Нажмите «Generate new secret» и скопируйте его в `ClientSecret` в файле конфигурации.

```admonish warning
Ваш клиентский секрет будет показан только один раз; если вы его потеряете, создайте новый.
```

## Конфигурация веб-сервера

Скорее всего, вы захотите запустить SS14.Admin за обратным прокси, таким как Nginx или Caddy, чтобы обеспечить терминацию TLS и позволить себе запускать несколько сервисов с одного IP-адреса. Вот несколько примеров и инструкций для некоторых веб-серверов.

Учтите, что SS14.Admin, вероятно, не будет работать без HTTPS из-за проблем с безопасностью cookie.

### Caddy

Caddy рекомендуется, если у вас ещё не установлен веб-сервер, так как её очень легко настраивать и она предоставляет встроенную функциональность вроде сертификатов Let's Encrypt.

```caddy
admin.example.com {
    log {
        output file /var/log/caddy/access-ss14-admin.log
    }

    reverse_proxy 127.0.0.1:<CHANGE TO YOUR PORT>
}
```

### Nginx

```nginx
location / {
    proxy_pass          http://localhost:<CHANGE TO YOUR PORT>;
    proxy_http_version  1.1;
    proxy_set_header    Upgrade $http_upgrade;
    proxy_set_header    Connection keep-alive;
    proxy_set_header    Host $host;
    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto https;
    proxy_cache_bypass  $http_upgrade;

    # Необходимо, чтобы избежать ошибок из-за слишком больших заголовков вследствие больших cookie.
    proxy_buffer_size        128k;
    proxy_buffers            4 256k;
    proxy_busy_buffers_size  256k;
}
```

## Устранение неполадок

### Ошибка на стороне авторизации «An error occurred while processing your request.» при входе

*См. также: [устранение неполадок на основной странице OAuth](./oauth.md#auth-side-an-error-occurred-while-processing-your-request).*

Это общий код ошибки, если конфигурация OAuth настроена неправильно. Возможные причины смотрите в статье по ссылке выше. Если URI перенаправления действительно неверен, проверьте ниже.

### Неверный URI перенаправления

Неверный URI перенаправления часто вызван неправильной конфигурацией обратного прокси, либо на стороне прокси, либо на стороне приложения:

* Убедитесь, что ваш обратный прокси отправляет все необходимые заголовки, как показано в примерах конфигурации выше.
* Если ваш обратный прокси отправляет не на `localhost`, например если вы работаете в контейнере, убедитесь, что `ForwardProxies` в файле конфигурации SS14.Admin задан правильно.
