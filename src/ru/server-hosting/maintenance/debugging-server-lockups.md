# Отладка зависаний сервера

Если `SS14.Watchdog` обнаружит, что ваш сервер завис, он убьёт сервер и перезапустит его. Если это действительно произойдёт на живом сервере, шанс просто посмотреть логи уже упущен. Это руководство даст быстрый ликбез о том, как подступиться к отладке такого рода проблем.

```admonish info
Просто чтобы было ясно: вы можете получить похожую ошибку, если игровой сервер не может связаться с watchdog из-за неверной конфигурации; в таком случае игровой сервер надёжно будет убиваться после запуска, циклически. Эта статья не об этом, а о настоящем, реально мерзком баге с падением.
```

## Предыстория: пинги Watchdog

Watchdog ожидает, что игровой сервер регулярно отправляет ему сообщения-*пинги*, указывающие, что игровой сервер ещё жив. Игровой сервер отправляет эти пинги с постоянным интервалом в 15 секунд (сейчас это не настраивается, о чём я тогда думал), а watchdog ожидает хотя бы один пинг каждые *`TimeoutSeconds`* из конфигурации экземпляра. Если игра застрянет в бесконечном цикле какого-либо рода, она перестанет отправлять эти пинги, и watchdog быстро убьёт её и перезапустит.

## Итак, что у нас есть?

Если вы хотите воспроизвести это очень легко, подключитесь к своему серверу и выполните что-то вроде этого в `scsi`:

![приглашение scsi перед запуском 'while (true) { }'](../../../en/assets/images/hosting/scsi-while-true.png)

Развернув фрагмент лога выше, мы получим что-то вроде этого:

```
[WARN] net.ent: Got late MsgEntity! Diff: -12, msgT: 0, cT: 12, player: PJB
[WARN] eng: MainLoop: Cannot keep up!
[22:50:20 WRN SS14.Watchdog.Components.ServerManagement.ServerInstance] test: timed out, killing
[22:50:20 INF SS14.Watchdog.Components.ServerManagement.ServerInstance] test: making on-kill process dump of type Normal
[createdump] Gathering state for process 91738 Robust.Server
[createdump] Writing minidump to file /home/luna/ss14_watchdog_test/instances/test/dumps/dump_2023-10-24_22-50-20
[createdump] Written 120233984 bytes (29354 pages) to core file
[createdump] Target process is alive
[createdump] Dump successfully written in 306ms
[22:50:20 INF SS14.Watchdog.Components.ServerManagement.ServerInstance] test: Process dump written to /home/luna/ss14_watchdog_test/instances/test/dumps/dump_2023-10-24_22-50-20
[22:50:20 INF SS14.Watchdog.Components.ServerManagement.ServerInstance] test: killing process...
[22:50:21 INF SS14.Watchdog.Components.ServerManagement.ServerInstance] test shut down with status ProcessExitStatus { Reason = ExitCode, Status = 137, IsClean = False }
[22:50:21 WRN SS14.Watchdog.Components.ServerManagement.ServerInstance] test shut down before sending ping on attempt 1
[22:50:21 INF SS14.Watchdog.Components.ServerManagement.ServerInstance] test: Restarting server after exit...
```

```admonish info
Если вы вообще читаете эту страницу, надеюсь, у вас достаточно опыта хостинга сервера, чтобы понимать это, но просто чтобы было ясно: эти два сообщения `[WARN]` — **не то**.
```

Сам игровой сервер не выдал осмысленных логов (как он редко делает в таком случае), единственное, на что мы можем ориентироваться, — это то, что watchdog убил нас. К счастью, watchdog по умолчанию настроен создавать **дамп памяти** процесса игрового сервера, если ему приходится его убивать, и указал путь, куда был выгружен дамп, в выводе лога.

Что такое дамп памяти? Это файл, содержащий память процесса в момент его падения. По умолчанию этот дамп включает достаточно памяти, чтобы получить информацию о трассировке стека с помощью отладчика. Если хотите большего, вы можете изменить свойство `TimeoutDumpType` в конфигурации экземпляра watchdog на [одно из этих значений](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/microsoft-diagnostics-netcore-client#dumptype-enum). Учтите, что дампы памяти довольно большие, а отчёт с большим количеством информации может сделать их *прямо-таки огромными*.

```admonish danger
Дампы памяти могут содержать конфиденциальную информацию с вашего сервера (например, пароли к базе данных) и их **не следует** передавать людям, которым вы не доверяете!
```

«Ладно, у меня есть файл весом пару сотен мегабайт, что мне с ним делать? Перетащить его в Rider?» Ах ты, милое летнее дитя.

У вас есть два известных мне варианта для отладки этого: lldb и WinDBG. Один из них — консольный отладчик. Другой — консольный отладчик Windows, который, по крайней мере, любезно предоставляет базовый интерфейс. Эй, по крайней мере, вам не нужно использовать gdb!

```admonish failure
Дампы памяти — хрупкие маленькие создания и по умолчанию не включают весь контекст, необходимый для их самостоятельной отладки. В общем, отлаживать их намного проще всего, если вы делаете это на той системе, где они возникли, и *ни один из лежащих в основе файлов не изменился*, например, из-за обновления игрового сервера.

При наличии нужного опыта и/или инфраструктуры (серверы символов, любимые мои) возможно разобраться с ними гораздо позже или на другой системе, но это далеко за рамками этого руководства, и даже у меня нет такого опыта.[^2]
```

### Использование lldb

lldb — приличный[^1] отладчик для Linux и macOS. Установите его:

```sh
# Или какой там у вас менеджер пакетов 🤗
$ sudo apt install lldb
```

Вам также понадобится [SOS](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/sos-debugging-extension). Он расширит lldb, чтобы сделать возможной отладку трассировки .NET:

```sh
luna@Mimas:~/ss14_watchdog_test
$ dotnet tool install -g dotnet-sos
You can invoke the tool using the following command: dotnet-sos
Tool 'dotnet-sos' (version '7.0.447801') was successfully installed.
luna@Mimas:~/ss14_watchdog_test
$ dotnet sos install
Installing SOS to /home/luna/.dotnet/sos
Creating installation directory...
Copying files from /home/luna/.dotnet/tools/.store/dotnet-sos/7.0.447801/dotnet-sos/7.0.447801/tools/net6.0/any/linux-x64
Copying files from /home/luna/.dotnet/tools/.store/dotnet-sos/7.0.447801/dotnet-sos/7.0.447801/tools/net6.0/any/lib
Creating new /home/luna/.lldbinit file - LLDB will load SOS automatically at startup
SOS install succeeded
```

Теперь вы готовы загрузить дамп памяти в lldb:
```
$ lldb -c instances/test/dumps/dump_2023-10-24_23-07-16
Current symbol store settings:
-> Cache: /home/luna/.dotnet/symbolcache
-> Server: https://msdl.microsoft.com/download/symbols/ Timeout: 4 RetryCount: 0
(lldb) target create --core "instances/test/dumps/dump_2023-10-24_23-07-16"
Core file '/home/luna/ss14_watchdog_test/instances/test/dumps/dump_2023-10-24_23-07-16' (x86_64) was loaded.
(lldb)
```

```admonish tip
Вам понадобится [шпаргалка](https://github.com/carolanitz/DebuggingFun/blob/d0b21847d1ad1506bbaaded915c1625a21165b9c/lldb%20cheat%20sheet.pdf) для этого приглашения `(lldb)`.
```

На этом этапе у вас открыт нативный отладчик, и вы вполне можете по-настоящему отладить всё. Чёрт, если игровой сервер умирает из-за нативных крашей, именно так вы бы его и отлаживали. Вы можете использовать обычные команды `lldb` для обычной нативной отладки нативных модулей и всего такого. Хотя это всё ещё не для слабонервных и может потребовать дополнительной настройки, например, для получения символов нативных библиотек.

Чтобы получить осмысленную информацию из *управляемой* трассировки стека, мы можем выполнить команду `clrstack` из SOS:

```
(lldb) clrstack
OS Thread Id: 0x172fa (1)
        Child SP               IP Call Site
00007FFFD245BFB0 00007F15096C2F7B Submission#0+<<Initialize>>d__0.MoveNext()
00007FFFD245C000 00007F150962B9FE System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[System.__Canon, System.Private.CoreLib]](System.__Canon ByRef)
00007FFFD245C060 00007F150962B940 System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1[[System.__Canon, System.Private.CoreLib]].Start[[System.__Canon, System.Private.CoreLib]](System.__Canon ByRef)
00007FFFD245C0A0 00007F15096C2E92 Submission#0.<Initialize>()
00007FFFD245C0E0 00007F15096C2CD3 Submission#0.<Factory>(System.Object[])
00007FFFD245C110 00007F150962B06C Microsoft.CodeAnalysis.Scripting.ScriptExecutionState+<RunSubmissionsAsync>d__9`1[[System.__Canon, System.Private.CoreLib]].MoveNext()
00007FFFD245C1C0 00007F150962AD2B System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[Microsoft.CodeAnalysis.Scripting.ScriptExecutionState+<RunSubmissionsAsync>d__9`1[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](<RunSubmissionsAsync>d__9`1<System.__Canon> ByRef)
00007FFFD245C220 00007F150962AC50 System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1[[System.__Canon, System.Private.CoreLib]].Start[[Microsoft.CodeAnalysis.Scripting.ScriptExecutionState+<RunSubmissionsAsync>d__9`1[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](<RunSubmissionsAsync>d__9`1<System.__Canon> ByRef)
00007FFFD245C260 00007F150962AB6F Microsoft.CodeAnalysis.Scripting.ScriptExecutionState.RunSubmissionsAsync[[System.__Canon, System.Private.CoreLib]](System.Collections.Immutable.ImmutableArray`1<System.Func`2<System.Object[],System.Threading.Tasks.Task>>, System.Func`2<System.Object[],System.Threading.Tasks.Task>, System.Runtime.CompilerServices.StrongBox`1<System.Exception>, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
00007FFFD245C320 00007F150962A4CC Microsoft.CodeAnalysis.Scripting.Script`1+<RunSubmissionsAsync>d__21[[System.__Canon, System.Private.CoreLib]].MoveNext()
00007FFFD245C500 00007F150962A21B System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[Microsoft.CodeAnalysis.Scripting.Script`1+<RunSubmissionsAsync>d__21[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](<RunSubmissionsAsync>d__21<System.__Canon> ByRef)
00007FFFD245C560 00007F150962A140 System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1[[System.__Canon, System.Private.CoreLib]].Start[[Microsoft.CodeAnalysis.Scripting.Script`1+<RunSubmissionsAsync>d__21[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](<RunSubmissionsAsync>d__21<System.__Canon> ByRef)
00007FFFD245C5A0 00007F150962A03F Microsoft.CodeAnalysis.Scripting.Script`1[[System.__Canon, System.Private.CoreLib]].RunSubmissionsAsync(Microsoft.CodeAnalysis.Scripting.ScriptExecutionState, System.Collections.Immutable.ImmutableArray`1<System.Func`2<System.Object[],System.Threading.Tasks.Task>>, System.Func`2<System.Object[],System.Threading.Tasks.Task>, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
00007FFFD245C670 00007F15065AF51D Microsoft.CodeAnalysis.Scripting.Script`1[[System.__Canon, System.Private.CoreLib]].RunAsync(System.Object, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
00007FFFD245C6D0 00007F15096C2BCA Microsoft.CodeAnalysis.Scripting.Script`1[[System.__Canon, System.Private.CoreLib]].CommonRunAsync(System.Object, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
00007FFFD245C720 00007F15096A8336 Robust.Server.Scripting.ScriptHost+<ReceiveScriptEval>d__12.MoveNext() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Scripting/ScriptHost.cs @ 209]
00007FFFD245C800 00007F15096A79F3 System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[Robust.Server.Scripting.ScriptHost+<ReceiveScriptEval>d__12, Robust.Server]](<ReceiveScriptEval>d__12 ByRef)
00007FFFD245C840 00007F15096A795C System.Runtime.CompilerServices.AsyncVoidMethodBuilder.Start[[Robust.Server.Scripting.ScriptHost+<ReceiveScriptEval>d__12, Robust.Server]](<ReceiveScriptEval>d__12 ByRef)
00007FFFD245C860 00007F15096A7913 Robust.Server.Scripting.ScriptHost.ReceiveScriptEval(Robust.Shared.Network.Messages.MsgScriptEval)
00007FFFD245C8F0 00007F150658947F Robust.Shared.Network.NetManager+<>c__DisplayClass106_0`1[[System.__Canon, System.Private.CoreLib]].<RegisterNetMessage>b__0(Robust.Shared.Network.NetMessage)
00007FFFD245C960 00007F1506587964 Robust.Shared.Network.NetManager.DispatchNetMessage(Lidgren.Network.NetIncomingMessage) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Shared/Network/NetManager.cs @ 912]
00007FFFD245CA30 00007F15059608DF Robust.Shared.Network.NetManager.ProcessPackets() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Shared/Network/NetManager.cs @ 492]
00007FFFD245CBE0 00007F15053F906A Robust.Server.BaseServer.Input(Robust.Shared.Timing.FrameEventArgs) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/BaseServer.cs @ 686]
00007FFFD245CD80 00007F1505963FA4 Robust.Shared.Timing.GameLoop.Run() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Shared/Timing/GameLoop.cs @ 135]
00007FFFD245D760 00007F150525A0F5 Robust.Server.BaseServer.MainLoop() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/BaseServer.cs @ 565]
00007FFFD245D7A0 00007F14F9729475 Robust.Server.Program.ParsedMain(Robust.Server.CommandLineArgs, Boolean, Robust.Server.ServerOptions) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Program.cs @ 78]
00007FFFD245D8E0 00007F14F971B8D2 Robust.Server.Program.Start(System.String[], Robust.Server.ServerOptions, Boolean) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Program.cs @ 46]
00007FFFD245D930 00007F14F9718901 Robust.Server.Program.Main(System.String[]) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Program.cs @ 25]
```

Вот с этим уже можно работать! Внизу — `main` программы, наверху — *`Submission#0`*, что является внутренней деталью C# Interactive, в котором мы запустили `while (true) { }`. Хотя я не могу показать вам байты IL или код C# в lldb (ну, уверен, первое возможно с SOS), я могу показать вам ассемблерный код, который выглядит как довольно простой бесконечный цикл:

```
(lldb) disassemble -s 0x7f15096c2f72 -F intel
    0x7f15096c2f72: nop
    0x7f15096c2f73: nop
    0x7f15096c2f74: mov    dword ptr [rbp - 0x1c], 0x1
->  0x7f15096c2f7b: nop
    0x7f15096c2f7c: jmp    0x7f15096c2f72
```

Надеюсь, это помогло начать использовать нативный отладчик для подобных вещей.

### Использование WinDBG

WinDBG — это отладчик Windows, который вы достаёте, когда всё остальное не помогло (а здесь так и есть). [WinDBG Preview можно получить в Microsoft Store.](https://www.microsoft.com/store/productid/9PGJGD53TN86)

Вам также понадобится [SOS](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/sos-debugging-extension). Он расширит WinDBG, чтобы сделать возможной отладку трассировки .NET:

```
C:\Users\Luna
> dotnet tool install -g dotnet-sos
Skipping NuGet package signature verification.
You can invoke the tool using the following command: dotnet-sos
Tool 'dotnet-sos' (version '7.0.447801') was successfully installed.
C:\Users\Luna
> dotnet sos install
Installing SOS to C:\Users\Luna\.dotnet\sos
Creating installation directory...
Copying files from C:\Users\Luna\.dotnet\tools\.store\dotnet-sos\7.0.447801\dotnet-sos\7.0.447801\tools\net6.0\any\win-x64
Copying files from C:\Users\Luna\.dotnet\tools\.store\dotnet-sos\7.0.447801\dotnet-sos\7.0.447801\tools\net6.0\any\lib
Execute '.load C:\Users\Luna\.dotnet\sos\sos.dll' to load SOS in your Windows debugger.
SOS install succeeded
```

Вы можете загрузить созданный файл дампа в WinDBG, выбрав File -> Start debugging -> Open dump file, а затем выбрав файл. Если у файла нет расширения, вам нужно изменить фильтр в диалоге открытия, чтобы его выбрать.

![Навигация в WinDBG для открытия файла дампа](../../../en/assets/images/hosting/windbg-open.png)

Вам нужно будет выполнить команду `.load`, упомянутую в выводе выше, чтобы загрузить SOS:

```
0:000> .load C:\Users\Luna\.dotnet\sos\sos.dll
```

```admonish tip
Вам, вероятно, всё ещё нужна нормальная шпаргалка по WinDBG Preview. У него есть немного интерфейса, но использование внутренней командной строки всё ещё очень необходимо. В любом случае, за 5 секунд поиска в интернете я не нашёл ничего похожего на ту хорошо оформленную lldb'шную, так что поищите сами, что ли.
```

Не буду повторяться со стороны lldb: у вас полноценный нативный отладчик. Он может делать всё, если вы умеете им пользоваться.

Вы можете использовать `!clrstack`, чтобы показать управляемую трассировку стека:

```
0:000> !clrstack
OS Thread Id: 0x9034 (0)
        Child SP               IP Call Site
000000F7BF57D0F0 00007ffab591c72a Submission#0+<>d__0.MoveNext()
000000F7BF57D150 00007ffaf84334b8 System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[System.__Canon, System.Private.CoreLib]](System.__Canon ByRef)
000000F7BF57D1B0 00007ffab591c642 Submission#0.()
000000F7BF57D220 00007ffab591c484 Submission#0.()
000000F7BF57D270 00007ffab4bb043c Microsoft.CodeAnalysis.Scripting.ScriptExecutionState+d__9`1[[System.__Canon, System.Private.CoreLib]].MoveNext()
000000F7BF57D330 00007ffab4bb0113 System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[Microsoft.CodeAnalysis.Scripting.ScriptExecutionState+d__9`1[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](d__9`1<System.__Canon> ByRef)
000000F7BF57D3A0 00007ffab4bb0040 System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1[[System.__Canon, System.Private.CoreLib]].Start[[Microsoft.CodeAnalysis.Scripting.ScriptExecutionState+d__9`1[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](d__9`1<System.__Canon> ByRef)
000000F7BF57D3E0 00007ffab4baff72 Microsoft.CodeAnalysis.Scripting.ScriptExecutionState.RunSubmissionsAsync[[System.__Canon, System.Private.CoreLib]](System.Collections.Immutable.ImmutableArray`1<System.Func`2<System.Object[],System.Threading.Tasks.Task>>, System.Func`2<System.Object[],System.Threading.Tasks.Task>, System.Runtime.CompilerServices.StrongBox`1<System.Exception>, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
000000F7BF57D490 00007ffab4baf932 Microsoft.CodeAnalysis.Scripting.Script`1+d__21[[System.__Canon, System.Private.CoreLib]].MoveNext()
000000F7BF57D680 00007ffab4baf6a3 System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[Microsoft.CodeAnalysis.Scripting.Script`1+d__21[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](d__21<System.__Canon> ByRef)
000000F7BF57D6F0 00007ffab4baf5d0 System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1[[System.__Canon, System.Private.CoreLib]].Start[[Microsoft.CodeAnalysis.Scripting.Script`1+d__21[[System.__Canon, System.Private.CoreLib]], Microsoft.CodeAnalysis.Scripting]](d__21<System.__Canon> ByRef)
000000F7BF57D730 00007ffab4baf4d0 Microsoft.CodeAnalysis.Scripting.Script`1[[System.__Canon, System.Private.CoreLib]].RunSubmissionsAsync(Microsoft.CodeAnalysis.Scripting.ScriptExecutionState, System.Collections.Immutable.ImmutableArray`1<System.Func`2<System.Object[],System.Threading.Tasks.Task>>, System.Func`2<System.Object[],System.Threading.Tasks.Task>, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
000000F7BF57D7F0 00007ffab30cd6b6 Microsoft.CodeAnalysis.Scripting.Script`1[[System.__Canon, System.Private.CoreLib]].RunAsync(System.Object, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
000000F7BF57D860 00007ffab591c3ba Microsoft.CodeAnalysis.Scripting.Script`1[[System.__Canon, System.Private.CoreLib]].CommonRunAsync(System.Object, System.Func`2<System.Exception,Boolean>, System.Threading.CancellationToken)
000000F7BF57D8B0 00007ffab58f6f0d Robust.Server.Scripting.ScriptHost+d__12.MoveNext() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Scripting/ScriptHost.cs @ 209]
000000F7BF57D9C0 00007ffab58f6606 System.Runtime.CompilerServices.AsyncMethodBuilderCore.Start[[Robust.Server.Scripting.ScriptHost+d__12, Robust.Server]](d__12 ByRef)
000000F7BF57DA20 00007ffab58f657c System.Runtime.CompilerServices.AsyncVoidMethodBuilder.Start[[Robust.Server.Scripting.ScriptHost+d__12, Robust.Server]](d__12 ByRef)
000000F7BF57DA50 00007ffab58f653c Robust.Server.Scripting.ScriptHost.ReceiveScriptEval(Robust.Shared.Network.Messages.MsgScriptEval)
000000F7BF57DAE0 00007ffab30c3755 Robust.Shared.Network.NetManager.DispatchNetMessage(Lidgren.Network.NetIncomingMessage) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Shared/Network/NetManager.cs @ 810]
000000F7BF57DBA0 00007ffab265bfb7 Robust.Shared.Network.NetManager.ProcessPackets() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Shared/Network/NetManager.cs @ 439]
000000F7BF57DD70 00007ffab265acea Robust.Server.BaseServer.Input(Robust.Shared.Timing.FrameEventArgs) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/BaseServer.cs @ 683]
000000F7BF57DE20 00007ffab2670b47 Robust.Shared.Timing.GameLoop.Run() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Shared/Timing/GameLoop.cs @ 135]
000000F7BF57E5E0 00007ffab25d6685 Robust.Server.BaseServer.MainLoop() [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/BaseServer.cs @ 565]
000000F7BF57E610 00007ffaa93f24f0 Robust.Server.Program.ParsedMain(Robust.Server.CommandLineArgs, Boolean, Robust.Server.ServerOptions) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Program.cs @ 78]
000000F7BF57E6A0 00007ffaa93f12ca Robust.Server.Program.Start(System.String[], Robust.Server.ServerOptions, Boolean) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Program.cs @ 46]
000000F7BF57E700 00007ffaa93f06f2 Robust.Server.Program.Main(System.String[]) [/home/runner/work/space-station-14/space-station-14/RobustToolbox/Robust.Server/Program.cs @ 25]
```

То же самое, что и выше, опять. Вау!

Надеюсь, это помогло начать использовать нативный отладчик для подобных вещей.

```admonish note
🤫 На самом деле я использовал `dotnet dump collect`, чтобы получить дамп для Windows, потому что мне было слишком лень настраивать watchdog дважды только ради этой статьи. Работает так же, в основном. Чёрт, я почти уверен, что он использует ту же самую библиотеку для создания дампа, что и watchdog!
```

[^2]: Я пытался открыть дамп памяти Linux в WinDBG, что поддерживается в некоторой степени... но он разрыдался из-за кучи отсутствующих файлов и других болей, и я сдался. Попробуйте убрать папку `bin/` вашего watchdog с пути и отладить дамп памяти с помощью lldb... Да, результаты получаются совершенно разные, ага?

[^1]: Примерно так хорошо, как это бывает с инструментами разработки на Linux, а это не очень-то здорово.
