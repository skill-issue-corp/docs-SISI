# Публикация RobustToolbox

```admonish info
Эти инструкции — пошаговое руководство для сопровождающих движка.
```

1. Откройте терминал в каталоге RobustToolbox (`cd RobustToolbox`, если вы в каталоге space-station-14)
2. Получите последнюю версию master (`git fetch https://github.com/space-wizards/RobustToolbox.git`)
3. Переключитесь на удалённую ветку master (`git checkout -B master upstream/master`, с ЗАГЛАВНОЙ 'B', чтобы перезаписать master)
   - Этот шаг перезапишет вашу локальную ветку `master` удалённой.
4. Запустите version.py (`python ./Tools/version.py 0.1.0`, где 0.1.0 — желаемый номер версии, БЕЗ 'v')
   - Если вы используете `py` вместо этого в Windows, это может не работать из-за алиаса python из Microsoft Store.
5. Отправьте ваш коммит и тег в RobustToolbox (`git push` и `git push https://github.com/space-wizards/RobustToolbox.git v0.1.0`, С 'v')
   - НЕ запускайте `git push --tags`, так как это отправит все теги, которые у вас есть локально, даже те, что были удалены.
6. Вернитесь в каталог контента (`cd ..`)
7. Создайте новую ветку (`git checkout -b update/robust-0.1.0`)
8. Закоммитьте изменение движка (`git commit RobustToolbox -m "Update RobustToolbox"`)
9. Отправьте вашу ветку (`git push`)
10. Откройте PR в репозиторий контента и влейте его.

```admonish warning
Всегда полезно запустить игру с новой версией движка перед её публикацией и вливанием PR, чтобы проверить, что всё по-прежнему работает.
Вы также можете запустить тесты локально с помощью "dotnet test", это будет быстрее, чем ждать их выполнения в рабочих процессах GitHub.
```
