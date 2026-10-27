# Chat mode для DeepSeek Harness

Режим `Chat mode` — это **agent preset** («пресет агента»). Именно их список
показывает меню выбора режима на экране нового чата: Standard, PTC, Minimal,
Creator. Пресет добавляет в этот список ещё один режим — `Chat mode`.

Никакого кода исполнения здесь нет: пресет — это декларация, какими
инструментами и каким системным промптом пользуется агент в этом режиме.

## Что умеет Chat mode

Сделано по образцу веб-версии DeepSeek Chat: обычный диалог с моделью.

| Возможность | Реализация |
|---|---|
| Веб-поиск и открытие страниц | `@deepseek-ai/dsh-tool-web` (`web_search`, `web_fetch`) |
| Файлы и картинки, приложенные к сообщению | `@deepseek-ai/dsh-tool-fs` только в режиме чтения (`read`, `read_image`) |
| Дата и время («сегодня», «через неделю») | `@deepseek-ai/dsh-time-context` |
| Длинный диалог не упирается в контекст | `compaction-basic` + `/compact` + обрезка больших результатов |
| Разговорный системный промпт | `@deepseek-ai/dsh-persona` |

Чего в режиме **нет** намеренно: терминала (`bash`/`pwsh`), записи и правки
файлов (`write`/`edit`), `glob`/`grep`, скиллов, plan mode, целей, субагентов и
workflow. Это чат, а не работа с репозиторием; для кода есть Standard mode.

## Установка

Пресет — это бандл из двух файлов в этой папке:

```
chat-mode/
  package.json        # объявляет dsh.bundle.patch
  cordis.patch.yml    # сама декларация пресета
```

**Установка не требуется, если бандл уже подключён к вашему профилю.** Раздел
ниже описывает, как поставить его (после переустановки DSH, на другой машине или
после удаления).

### Штатная установка

Нужен `dsh` из Desktop-сборки: обычный CLI намеренно отказывается управлять
профилем `desktop`.

```powershell
# путь к CLI внутри установленного DeepSeek Harness
$dsh = "$env:LOCALAPPDATA\Programs\DeepSeek Harness\DeepSeek Harness.exe"
$cli = "$env:LOCALAPPDATA\Programs\DeepSeek Harness\resources\app.asar\dsh\node_modules\@deepseek-ai\dsh-desktop-host\lib\cli.js"

& $dsh --expose-internals $cli `
  plugin --profile desktop add "link:<путь к этой папке>"
```

`<путь к этой папке>` — абсолютный путь к скачанной папке плагина, например
`link:C:/plugins/dsh-chat-mode`. Прямые слэши обязательны.

Команда сама добавляет зависимость в `package.json`, обновляет
`pnpm-lock.yaml`, связывает пакет в `node_modules` и **выбирает бандл** в
`dsh.profile.bundles`. Приложение подхватывает новый бандл через HMR-перезагрузку
профиля; если список режимов не обновился — перезапустите приложение.

То же самое доступно без консоли: **Plugins → Add plugin** с путём к этой папке,
либо запрос агенту в режиме Creator.

### На других платформах

Установите бандл в свой профиль той же командой `dsh plugin add`, подставив свой
путь. Профиль `web`/`tui` и их пути к DSH берутся из вашей установки.

### Удаление

```powershell
& $dsh --expose-internals $cli `
  plugin --profile desktop remove dsh-chat-mode
```

и уберите `dsh-chat-mode` из `dsh.profile.bundles` в
`%USERPROFILE%\.dsh\profiles\desktop\package.json`, если он там остался.


## Как выбрать режим

1. Нажмите **New session** (новый чат).
2. Чип с названием текущего режима рядом с выбором рабочей папки — это и есть
   меню выбора. Откройте его и выберите **Chat mode**.

Меню появляется **до** первого сообщения: после первого хода набор инструментов
сессии зафиксирован, и Host не даёт его сменить (в заголовке сессии режим
показывается только для чтения).

Чтобы Chat mode был режимом по умолчанию для новых чатов: **Settings → Agent
presets → Chat mode → Set as new task default**.

## Проверка после установки

- **Settings → Agent presets**: карточка Chat mode в группе *Custom* с её
  описанием и кнопкой *View configuration*.
- **New session → меню режима**: пункт *Chat mode*.
- Внутри чата в Chat mode нет терминала и правки файлов, но работают
  веб-поиск и чтение приложенных файлов.
