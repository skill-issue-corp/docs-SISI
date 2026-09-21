# FAQ для новичков

## Вклад в Space Station 14

### Я хочу начать вносить вклад в Space Station 14, но не знаю, с чего начать.

Перейдите к [задачам (Issues) в Space Station 14](https://github.com/space-wizards/space-station-14/issues)
- Выберите 1 или несколько из следующих меток в разделе [Labels](https://github.com/space-wizards/space-station-14/labels)
1. [Beginner Friendly](https://github.com/space-wizards/space-station-14/labels/DB%3A%20Beginner%20Friendly)
2. [Difficulty: 3 - Low](https://github.com/space-wizards/space-station-14/labels/D3%3A%20Low)
3. [No C#](https://github.com/space-wizards/space-station-14/labels/Changes%3A%20No%20C%23) — для работы над задачами, не требующими кода. 

После того как вы выбрали задачу для исправления, выполните шаги ниже, чтобы начать вносить и тестировать изменения кода в своей собственной копии кода Space Station 14: 
- [Настройка среды разработки](https://docs.spacestation14.com/en/general-development/setup/setting-up-a-development-environment.html) 
   - Это позволит вам запустить локальную копию игры, чтобы увидеть свои изменения в игровом процессе.
- [Git для разработчика SS14](https://docs.spacestation14.com/en/general-development/setup/git-for-the-ss14-developer.html)
   - Это позволит вам работать над кодом, который можно перенести с вашего компьютера в свой репозиторий Github и в конечном итоге в Space Station 14.



### Как код переносится между моим компьютером и основным репозиторием Space-Station-14? 

```admonish danger "Создайте новую ветку, чтобы не работать в ветке master!"
Это важно знать, чтобы случайно не удалить все свои изменения кода при обновлении своей копии Space Station 14.
- [Git для разработчика SS14](https://docs.spacestation14.com/en/general-development/setup/git-for-the-ss14-developer.html#3-setting-up-remotes)
```

**3 места, где находится код, которые нужно знать:**
1) Код на вашем компьютере (ваша копия)
2) Ваш репозиторий Github (локальный Github)
3) Github Space Station 14


**Как переносится код.**
1. **Github Space Station 14 => локальный Github**. Вы делаете форк репозитория Space Station 14, чтобы у вас была собственная копия кода в локальном Github.
2. **Локальный Github <==> Github Space Station 14**. НЕ вносите собственные изменения кода в ветке master.
- Ваша ветка master должна быть связана с веткой master Space Station 14.
- Каждый раз, когда Github Space Station 14 обновляет свой код, ваша ветка master в локальном репозитории Github тоже должна обновлять свой код, чтобы оставаться синхронизированной.
3. **Локальный Github => ваша копия**. Следуйте инструкциям [Настройка среды разработки](https://docs.spacestation14.com/en/general-development/setup/setting-up-a-development-environment.html) и [Git для разработчика SS14](https://docs.spacestation14.com/en/general-development/setup/git-for-the-ss14-developer.html)
4. **Ваша копия**. Создайте новую ветку в своём редакторе кода, где будете вносить изменения.
5. **Ваша копия**. Протестируйте свои изменения в игровом процессе.
6. **Ваша копия => локальный Github**. Когда код готов, сделайте коммит в свой репозиторий Github из ветки, отличной от master.
7. **Github Space Station 14 => локальный Github**. Sync Fork

![syncfork.png](../../../en/assets/images/general-development/tips/beginner-faq/syncfork.png)

- Чтобы ваш код работал, он должен вписываться в код в Github Space Station 14.
- Вам придётся поддерживать код в локальном Github в актуальном состоянии относительно Github Space Station 14.

8. **Локальный Github => ваша копия** Вытяните обновлённый код из локального репозитория Github в свою копию.

![syncfork.png](../../../en/assets/images/general-development/tips/beginner-faq/pullcode.png)

Возможно, вам придётся слить изменения, если вы пытаетесь изменить файлы, которые изменились при обновлении.
![mergechanges.png](../../../en/assets/images/general-development/tips/beginner-faq/mergechanges.png)


9. **Ваша копия => локальный Github**. Зафиксируйте свои изменения.
10. **Локальный Github <==> Github Space Station 14**. Создайте pull request.
- Если вы вносите изменения кода из ветки master и создаёте pull request из ветки master, и ваши изменения, и ваш pull request рискуют самоуничтожиться.
   - Единственный способ поддерживать ваш код в актуальном состоянии относительно реального кода Space Station 14 — синхронизировать код.
   - Это потребует от вас отбросить свои коммиты.
   - Как только вы отбросите коммиты, ваши изменения кода будут удалены, а ваш pull request будет закрыт. Что невесело.
11. Дождитесь, пока рецензенты кода сообщат, годится ли ваш код для слияния. 



### У меня есть идея новой функции или исправления проблемы, которой я не видел на Github.

1. Сначала попробуйте поиграть в игру, чтобы знать, какие функции уже реализованы.
- «Раздувание функций» происходит, если вы делаете функцию, которая уже существует.
   - Например, возможность переключать определённое действие нажатием «4», когда его уже можно переключать нажатием «z».
2. Предложите функцию перед тем, как начать её делать. [Предложения функций](https://docs.spacestation14.com/en/general-development/feature-proposals.html)
3. Откройте новую задачу в Space Station 14: [New Issue](https://github.com/space-wizards/space-station-14/issues/new/choose)



## Программирование Space Station 14

### Я не могу запустить локальную копию игры, потому что загружаются не все проекты.

Вы выполнили шаг 2.3 из [Git для разработчика SS14](https://docs.spacestation14.com/en/general-development/setup/git-for-the-ss14-developer.html#23-submodule-woes)?

Убедитесь, что используете команду `cd`, чтобы перейти в свой репозиторий space-station-14, прежде чем запускать `RUN_THIS.py`.
![cdcommand.png](../../../en/assets/images/general-development/tips/beginner-faq/cdcommand.png)



## Изучение Space Station 14

### Я ищу текст, который видел в игре, но не уверен, где его искать.

1. Много внутриигрового текста можно найти в XAML- и YML-файлах. Вы можете искать по XAML-файлам, чтобы найти части пользовательского интерфейса, над которыми хотите работать.
![xamlandymlsearch.png](../../../en/assets/images/general-development/tips/beginner-faq/xamlandymlsearch.png)
2. Если YML-файлы не отображаются, их можно найти на Github, перейдя в [репозиторий Space Station 14](https://github.com/space-wizards/space-station-14?search=1) и поискав файлы .yml.
3. Используйте свой редактор кода для поиска точного текста (для текста, не основанного на Project Fluent). Например, Visual Studio позволяет использовать Ctrl + Shift + F, чтобы найти определённый текст во всех файлах.

![codeeditor.png](../../../en/assets/images/general-development/tips/beginner-faq/codeeditor.png)

4. Не весь текст отображается точно так, как написан, потому что Space Station 14 использует Project Fluent для создания текста, который автоматически переводится на разные языки.
- [Project Fluent](https://docs.spacestation14.com/en/ss14-by-example/fluent-and-localization.html)
   - Это означает, что некоторый текст отображается как `Loc.GetString("id-that-references-fluent-file")`

![projectfluent.png](../../../en/assets/images/general-development/tips/beginner-faq/projectfluent.png)

5. Если вы хотите найти, где метод используется во всём коде, можно нажать на имя метода на Github, чтобы найти места его использования.

![methodname.png](../../../en/assets/images/general-development/tips/beginner-faq/methodname.png)
