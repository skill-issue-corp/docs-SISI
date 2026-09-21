# Настройка среды разработки

Сначала понадобится некоторое программное обеспечение:

* [Git](https://git-scm.com/) или один из [многих](https://www.sourcetreeapp.com/) [сторонних](http://www.syntevo.com/smartgit/) [графических интерфейсов](https://tortoisegit.org/), упрощающих его использование. Обязательно разрешите установку в PATH, как [здесь](../../../en/assets/images/setup/git-path.png).
* [Python 3.7 или выше](https://www.python.org/). Обязательно установите его в [PATH в Windows](../../../en/assets/images/setup/python-path.png). Также убедитесь, что при установке в Windows включена опция «py launcher». Python следует брать с [python.org](https://www.python.org/). Версии, установленные из Microsoft Store, иногда вызывают проблемы со сборкой.
* [SDK .NET 10.0](https://dotnet.microsoft.com/download/dotnet/10.0). Visual Studio также устанавливает его, если вы на Windows.
  * Пользователи Mac на Apple Silicon (ARM64): некоторые старые кодовые базы работают только с x64 .NET, а не с ARM64. Вы можете либо скачать x64 dotnet, либо предложить своей кодовой базе обновить robust toolbox как минимум до 267.0.0 для поддержки (или просто обновить его самостоятельно).
* Желательно IDE, чтобы разработка не была мучением (все варианты бесплатны, если не указано иное):
  * Для **всех платформ** [Rider](https://www.jetbrains.com/rider/) — одна из лучших доступных IDE, и многие мейнтейнеры и участники SS14 предпочитают её Visual Studio. Раньше она была платной, но теперь бесплатна для некоммерческого использования.
  * Для **Windows** — [Visual Studio 2026 **Community**](https://www.visualstudio.com/). Для минимальной установки (господи, она огромная) вам понадобятся рабочая нагрузка «.NET desktop development», компилятор C#, поддержка C#, менеджер пакетов NuGet, MSBuild и .NET 10 SDK или что-то в этом духе.
  * Для **всех платформ** — [Visual Studio Code](https://code.visualstudio.com/) с расширением C#. Обычно даёт менее полноценный опыт IDE, чем полноценные IDE вроде обычной Visual Studio, но некоторым опытным программистам нравится минимализм.
    * **Только для VSCode/VSCodium**: вы можете установить созданное сообществом расширение [Robust YAML](https://marketplace.visualstudio.com/items?itemName=slava0135.robust-yaml) для лучшей работы с YAML Robust Toolbox поверх расширения [YAML Language Support](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml).
  * Для **всех платформ** — [VSCodium](https://vscodium.com/) с расширением C#. Открытый исходный код, без раздутости и слежки VSCode.

```admonish warning title="VScode/VsCodium и SLNX"
Сейчас расширение C# Dev tools, похоже, не полностью поддерживает SLNX.

Вы можете выполнить этот шаг от [Microsoft](https://devblogs.microsoft.com/dotnet/introducing-slnx-support-dotnet-cli/#c#-dev-kit), чтобы иметь возможность открывать проект dotnet SLNX.

VSCodium, похоже, вообще не имеет такой возможности. (https://github.com/muhammadsammy/free-vscode-csharp/issues/95) (Хотя описанное выше может сработать и в VSCodium.)

```

~~~admonish info title="Windows и winget"
Пользователям Windows рекомендуется использовать winget для более простой установки: просто откройте командную строку/PowerShell и введите следующее:

Обязательно:
```
winget install Git.Git
winget install Python.Python.3.13
winget install Microsoft.DotNet.SDK.10
```

И одну из следующих IDE:

``winget install JetBrains.Toolbox`` (замените на ``JetBrains.Rider``, если не хотите всё приложение toolbox)

``winget install Microsoft.VisualStudio.Community`` (Visual Studio 2026)

``winget install Microsoft.VisualStudioCode`` (Visual Studio Code)

``winget install VSCodium.VSCodium`` (VSCodium)
~~~

## Застряли?
Застряли? Не понимаете, как выполнить определённую часть? Это видео должно помочь

Это видео предназначено для просмотра *вместе с руководством* и может устареть.

{% embed youtube id="EUGl_zNS6Uk?t=3" loading="lazy" %}

Вы также всегда можете попросить помощи в официальном [discord Wizard's Den](https://discord.spacestation14.io) или в [discord разработчиков, независимом от форков](https://discord.gg/X3YCdGHpgF)

## 1. Клонирование

```admonish danger title="Не скачивайте как zip с GitHub"
Вам нужно использовать `git` в той или иной форме (командная строка или графический интерфейс), чтобы скачать/клонировать код.
Опция «Download zip» на GitHub НЕ сработает, так как она не содержит необходимых подмодулей (то есть игрового движка Robust Toolbox) и к тому же не содержит предыдущую историю.
А значит, без неё было бы невозможно даже сделать коммит.
```

**Даже если вы уже знаете Git, прокрутите вниз и прочитайте раздел о настройке подмодулей. Серьёзно.**

Если вы **знакомы с Git**, просто сделайте форк и клонируйте репозиторий, настройте remotes, а затем следуйте руководству по подмодулям ниже.

Если вы **не знакомы с Git** или просто не знаете, как продолжить, следуйте руководству [Git для разработчика SS14](./git-for-the-ss14-developer.md), которое подробно рассказывает, как вносить вклад в игру и как настроить свой первый репозиторий. Оно также затрагивает настройку подмодулей, но она включена и сюда из-за её важности.

## 2. Настройка подмодулей

У нас есть автоматический обновлятор подмодулей, так что вам не нужно беспокоиться о постоянном запуске `git submodule update --init --recursive`.

Запустите `RUN_THIS.py` внутри скачанного репозитория с помощью Python. Желательно тоже из терминала. Это должно занять несколько секунд, так что если оно мгновенно останавливается, проверьте, используете ли вы Python 3.7+, иначе читайте дальше.

**Если при запуске `RUN_THIS.py` окно сразу открывается и закрывается: не волнуйтесь.** Это не значит, что произошла ошибка. Скрипт закрывается автоматически по завершении, так что если хотите убедиться, что всё сработало правильно, проверьте подмодуль `/RobustToolbox/` и убедитесь, что все файлы на месте. Если нет, посмотрите раздел устранения неполадок внизу этой страницы.

Примечание: если при начале работы у вас возникают проблемы с отсутствующими файлами, рекомендуется один раз вручную выполнить `git submodule update --init --recursive`, на случай если что-то пошло не так с python.

Однако если вы *действительно* хотите напрямую изменять движок или обновлять подмодуль вручную (автообновление может докучать), создайте файл с именем `DISABLE_SUBMODULE_AUTOUPDATE` в каталоге `BuildChecker/`.

И на этом ваш репозиторий настроен как надо!

## 3. Настройка IDE

### Visual Studio

1. Скачайте Visual Studio Community (если у вас нет платной версии) [здесь](https://visualstudio.microsoft.com/vs/community/)
2. Запустите установщик, выберите `.net desktop development` и установите
3. Если установщик спросит вас о среде разработки, выберите `Visual C#`.
4. Откройте Visual Studio
5. Выберите `Open a project or solution`, затем перейдите к клонированному ранее репозиторию и откройте `SpaceStation14.slnx`

### JetBrains Rider
1. Установите Rider, рекомендуем использовать [JetBrains Toolbox](https://www.jetbrains.com/toolbox-app/), чтобы он также мог автоматически обновляться в будущем.
2. Пройдите настройку.
3. Нажмите «Open» и выберите `SpaceStation14.slnx`
4. Если вы планируете заниматься разработкой движка, нужно добавить Robust Toolbox в Directory Mappings, чтобы VCS Rider мог обнаруживать изменения в Robust.
   Откройте настройки Rider и перейдите в раздел Version Control > Directory Mappings, нажмите кнопку плюс (+). В поле Directory укажите папку `RobustToolbox` в проекте, а в качестве VCS — Git
5. Выберите ветку, которую хотите использовать, в левом верхнем углу.
6. Выберите, что хотите запустить, из выпадающего меню рядом с зелёной кнопкой воспроизведения в верхней части окна. Настройка завершена. Теперь можно нажать кнопку воспроизведения, чтобы скомпилировать и запустить.

### VSCodium
1. Скачайте [VSCodium здесь](https://vscodium.com/) или напрямую [на Github здесь](https://github.com/VSCodium/vscodium/releases) (в последнем релизе нажмите на выпадающий список assets и прокрутите до ZIP или .exe для вашей ОС).
2. Запустите установщик или распакуйте zip-файл в удобное место и после распаковки запустите .exe.
3. После установки перейдите на вкладку Extensions (чуть ниже в левой верхней панели, выглядит как 4 плитки) и поищите «C#». Нужное расширение — от «Muhammad-Sammy» с более чем 70 тыс. загрузок и зелёно-белым логотипом, установите его. ID расширения `muhammad-sammy.csharp`.
4. Выберите File > Open Folder, затем перейдите к клонированному ранее репозиторию и откройте эту папку целиком.
5. Когда вас попросят открыть решение, выберите `SpaceStation14.slnx`. Либо задайте параметр `dotnet.defaultSolution` равным `SpaceStation14.slnx` в настройках рабочего пространства.
6. Теперь вы можете запускать и отлаживать игру. Выберите значок выше «Extensions» из более раннего шага — «Run and Debug», и из выпадающего списка рядом с зелёной кнопкой воспроизведения выберите «Server/Client». Это запустит и клиент, и сервер, открыв игру для отладки. Внизу в режиме отладки появится соответствующая информация. Выбирайте процессы в стеке вызовов слева, чтобы изменить то, что отлаживаете.

## 4. Запуск SS14

Теперь можно перейти к компиляции! Откройте файл решения `SpaceStation14.slnx` в своей любимой IDE и соберите и запустите нужные сборки (и Content.Server, и Content.Client).
Нажмите Direct Connect в окне клиента, чтобы начать тестирование.

Чтобы скомпилировать без IDE, выполните `dotnet build` в каталоге репозитория Space Station 14. Затем вызовите следующие команды, чтобы запустить клиент и сервер.
* `dotnet run --project Content.Server`
* `dotnet run --project Content.Client`

Обе эти команды по умолчанию используют конфигурацию debug. Чтобы включить оптимизации release, добавьте `--configuration Release` к вызову dotnet.
 
Примечание: если у вас проблемы с тем, что dotnet не находит libssl (например, при использовании libressl), попробуйте задать переменную окружения `CLR_OPENSSL_VERSION_OVERRIDE` в соответствующую версию. Например, установите её в `48`, если в `/usr/lib` есть `libssl.so.48`.
Если это не поможет, можно также вместо этого попробовать выполнить `ln -s /usr/lib/libssl.so /usr/local/lib/libssl.so.1.0.0`.

## 5. Настройка параметров сборки

Клиент и сервер SS14 — независимые проекты, но оба можно запускать одной кнопкой где-нибудь в вашей IDE. Однако это нужно настроить. Примечание: **рекомендуется запускать `Content.Client` и `Content.Server` при разработке из IDE.** *Не* `Robust.Client` или `Robust.Server`. Причина в том, что запуск `Content.*` заставит вашу IDE правильно учитывать зависимости и гарантирует, что всё будет хорошо пересобрано. Если запускать `Robust.Client` напрямую, придётся каждый раз убеждаться, что решение полностью собрано, что раздражает и легко забывается. Если вы не уверены, что такое Robust или Content, посмотрите [эту страницу](../codebase-info/codebase-organization.md) об организации проекта.

### Visual Studio 2022

В Visual Studio 2022 можно настроить кнопку сборки так, чтобы запускать и сервер, и клиент: щёлкните правой кнопкой по решению, затем выберите `Configure StartUp Projects...`. В появившемся меню выберите `Multiple startup projects:` и установите действие для `Content.Client` и `Content.Server` в `Start`. После применения изменений нажатие большой кнопки `Start` с зелёной стрелкой рядом должно одновременно запустить и клиент, и сервер.

Примечание: если у вас проблемы с тем, что программа собирается неправильно, возможно, нужно включить сборку перед запуском. Перейдите в Options `Projects and Solutions/Build and Run` и измените `On Run, when projects are out of date` на `Always build`.

В VS также можно использовать клавиши F7 для сборки проекта и F5 для его запуска.

### Visual Studio Code

Расширение C# предоставляет тип запуска `"coreclr"`, который можно использовать для запуска исполняемых файлов `Content.Server` и `Content.Client` в их соответствующих каталогах `bin/`. [Составная конфигурация запуска](https://code.visualstudio.com/Docs/editor/debugging#_compound-launch-configurations) позволяет запускать сервер и клиент одновременно.

### Командная строка

Соберите с помощью `dotnet build` и запустите клиент и сервер в разных командных строках с помощью:

* `dotnet run --project Content.Server`
* `dotnet run --project Content.Client`

Определённо есть и какой-то способ запускать две команды одновременно, но, вероятно, стоит загуглить это.

### JetBrains Rider

Чтобы легче запускать или отлаживать тестовые сборки в Rider, можно создать [составную конфигурацию](https://www.jetbrains.com/help/rider/Run_Debug_Multiple.html#compound-configs), которая запускает клиент и сервер одновременно. Весьма удобно!
В проекте уже может быть конфигурация, которую можно выбрать из выпадающего списка сверху, но если у неё красный символ, она настроена неправильно и её нужно создать вручную, либо она ещё не загрузилась. После этого нажмите Shift+F10 или кнопку воспроизведения, чтобы запустить её. Готово!

![](../../../en/assets/images/setup-rider-configurations-1.jpg)
![](../../../en/assets/images/setup-rider-configurations-2.jpg)

## 6. Настройка каталогов IDE

C# IDE вроде Visual Studio и Rider не показывают автоматически папку `Resources` в проекте. Эта папка содержит все не-C# файлы: спрайты, звук и, что важнее всего, YAML-прототипы. Эти инструкции объясняют, как заставить эту папку отображаться в вашей IDE, чтобы с ней было удобно работать.

### Visual Studio 2022

В Visual Studio можно переключить **Solution Explorer** из представления «solution» (показывает только C# проекты) в представление «folder» (показывает все файлы в проекте). Нажмите кнопку переключения представлений, как показано ниже, затем выберите представление папок:

![](../../../en/assets/images/setup/vs-solution-explorer-switch-view-1.png)
![](../../../en/assets/images/setup/vs-solution-explorer-switch-view-2.png)

После этого Solution Explorer должен выглядеть примерно так, и вы сможете легко получить доступ к папке `Resources`:

![](../../../en/assets/images/setup/vs-solution-explorer-switch-view-3.png)

### JetBrains Rider

В Rider можно «прикрепить» каталог resources к решению. Для этого щёлкните правой кнопкой по решению в проводнике, затем «Add» -> «Existing Folder...». Выберите каталог «Resources» в диалоге выбора файлов.

![asdfs](../../../en/assets/images/setup/rider-attach-folder-1.png)
![](../../../en/assets/images/setup/rider-attach-folder-2.png)

После этого представление решения должно выглядеть примерно так, и вы сможете легко получить доступ к папке `Resources`:

![](../../../en/assets/images/setup/rider-attach-folder-3.png)

### Visual Studio Code

Visual Studio Code показывает все файлы по умолчанию, так что здесь дополнительная настройка не нужна.

# Воспроизводимая среда разработки с Nix/NixOS

Более простой способ настроить среду разработки для пользователей Linux — задействовать Nix. Nix — это менеджер пакетов и функциональный предметно-ориентированный язык, позволяющий объявлять что угодно: от сред разработки до целых систем. Чтобы избежать печально известной проблемы «у меня это работает», мы можем объявить среду разработки на Nix, которая порождает изолированную воспроизводимую оболочку.

## Настройка Nix/NixOS с flakes

Вы можете [установить Nix](https://nixos.org/download) либо установив сам дистрибутив NixOS, либо воспользовавшись скриптом, совместимым со всеми Linux-дистрибутивами, использующими systemd (Ubuntu, Fedora, Mint и т. д.). Ради простоты и удобства рекомендуется установить Nix в дистрибутиве, с которым вам комфортно, вместо того чтобы полностью переходить на другую операционную систему. Также возможно использовать Nix с MacOS через `nix-darwin`, хотя это ещё не тестировалось и потому не рассматривается в этой статье.

После установки Nix следует включить экспериментальные функции, такие как flakes. Если вы на не-NixOS дистрибутиве, просто добавьте следующее в свой `~/.config/nix/nix.conf`.

* `experimental-features = nix-command flakes`

Если вы используете NixOS, нужно лишь добавить эти параметры в файл `configuration.nix`.

* `nix.settings.experimental-features = [ "nix-command" "flakes" ];`

Дополнительную информацию о том, как включить flake в Nix, см. [здесь](https://nixos.wiki/wiki/Flakes).

## Использование flake Nix для среды разработки Robust

К сведению: технически требуется, чтобы у вас уже был установлен Git, но в случае большинства Linux-дистрибутивов он идёт предустановленным. В крайне маловероятном случае, если его нет:

* Используйте менеджер пакетов вашего дистрибутива

* Объявите его в файле `configuration.nix`, если используете NixOS. Рекомендуется проверить [соответствующий раздел в руководстве NixOS](https://nixos.org/manual/nixos/stable/#sec-configuration-file), но если кратко — добавьте `pkgs.git` в атрибут `environment.systemPackages`.

В терминале просто перейдите в корневой каталог вашего репозитория SS14 и выполните:

* `nix develop`

Nix автоматически обработает все зависимости, объявленные в `shell.nix` и вызываемые файлом `flake.nix`. У вас появится новая эфемерная оболочка (известная как `devShell`), в которой установлено всё необходимое для сборки SS14 из исходников.

Именно поэтому flakes настоятельно рекомендуются, несмотря на то что считаются экспериментальной функцией. Мы можем гарантировать, что у всех одинаковые версии зависимостей, указав версию коллекции nixpkgs в атрибуте input flake и зафиксировав версии в файле `flake.lock`. Таким образом все участники, использующие Nix/NixOS, получают абсолютно одинаковую среду разработки. Каламбур ненамеренный, но это довольно robust!

## (Необязательно) Запуск JetBrains Rider через Nix

Затем вы можете использовать редактор или IDE на свой выбор. Однако в уже порождённой оболочке достаточно указать, что вам нужен JetBrains Rider. Выполните эту команду в вашем devShell.

* `NIXPKGS_ALLOW_UNFREE=1 nix shell nixpkgs#jetbrains.rider --impure`

Из новой оболочки можно запустить «отсоединённый» процесс JetBrains Rider, выполнив что-то вроде:

* `nohup rider >/dev/null 2>&1 &`

И вуаля! Вы надёжно настроили среду разработки так, что она не приводит к назойливому накоплению «состояния». Вы можете практически работать над SS14 из любого Linux-дистрибутива (при условии, что они используют systemd), не изменяя необратимо свою систему.

# Устранение неполадок

Убедитесь, что [первые три пункта](#setting-up-a-development-environment) сверху скачаны.

## `RUN_THIS.py` не запускается
Проверьте, что python установлен с сайта, а не из Microsoft Store. Если он установлен из Microsoft Store, удалите его, затем скачайте и установите с сайта python.

Если вы на Windows и вас перенаправляет в Microsoft Store или в терминале появляется сообщение о том, что Python не установлен. Эта проблема может быть вызвана глупым ярлыком Microsoft. Его можно отключить, найдя `Manage App Execution Aliases` и отключив оба упоминания python

### py не найден
Если python установлен с сайта и команда `python` работает, но вы всё равно получаете ошибку «py is not installed», проверьте, работает ли `C:\WINDOWS\py.exe`. Если да, добавьте `C:\WINDOWS` в свой path.

## System.DllNotFoundException: Unable to load DLL 'freetype6' or one of its dependencies: The specified module could not be found.

```PS C:\Users\Larme\Downloads\space-station-14> dotnet run --project Content.Client
Unhandled exception. Robust.Shared.IoC.Exceptions.ImplementationConstructorException: Robust.Client.Graphics.FontManager threw an exception inside its constructor.
 ---> System.DllNotFoundException: Unable to load DLL 'freetype6' or one of its dependencies: The specified module could not be found. (0x8007007E)
   at SharpFont.FT.FT_Init_FreeType(IntPtr& alibrary)
   at SharpFont.Library..ctor()
   at Robust.Client.Graphics.FontManager..ctor(IClyde clyde) in C:\Users\Larme\Downloads\space-station-14\RobustToolbox\Robust.Client\Graphics\FontManager.cs:line 33
   --- End of inner exception stack trace ---
   at Robust.Shared.IoC.DependencyCollection.BuildGraph() in C:\Users\Larme\Downloads\space-station-14\RobustToolbox\Robust.Shared\IoC\DependencyCollection.cs:line 348
   at Robust.Shared.IoC.IoCManager.BuildGraph() in C:\Users\Larme\Downloads\space-station-14\RobustToolbox\Robust.Shared\IoC\IoCManager.cs:line 271
   at Robust.Client.GameController.InitIoC(DisplayMode mode) in C:\Users\Larme\Downloads\space-station-14\RobustToolbox\Robust.Client\GameController\GameController.IoC.cs:line 16
   at Robust.Client.GameController.ParsedMain(CommandLineArgs args, Boolean contentStart, IMainArgs loaderArgs, GameControllerOptions options) in C:\Users\Larme\Downloads\space-station-14\RobustToolbox\Robust.Client\GameController\GameController.Standalone.cs:line 49
```

Либо:
- Кодовая база, которую вы запускаете, не поддерживает процессоры arm64 (Apple Silicon, Snapdragon). Вам нужно попросить свою кодовую базу обновить robust toolbox или сделать это самостоятельно.
- Вы случайно установили x86-версию dotnet; в этом случае удалите .NET Core SDK x86. Установите .NET Core SDK x64.


## Клиент и сервер недоступны в Visual Studio для настройки в Multiple startup projects

Возможно, это потому, что вы открыли проект как папку, а не как решение. Убедитесь, что открываете его как решение и нажимаете на файл space station 14 .slnx.

## Система не может найти указанный файл RUN_THIS.py

Ошибка `The system cannot find the specified file` обычно означает, что OneDrive конфликтует с git-репозиторием. Клонируйте git-репозиторий вне OneDrive или отключите синхронизацию для клонированной папки.
