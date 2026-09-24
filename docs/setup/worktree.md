# Worktree-сессии Agent Manager

Артефакт задачи D1 (ветка `d1/1.2.1-1.2.3-pilin-dev`). Работа шла параллельно в
двух копиях репозитория: основная сессия — в рабочей копии, агент-обзор — в
отдельной worktree.

## git worktree list

```
PS C:\Users\p.ilin\PycharmProjects\~\carmoney-lab> git worktree list
C:/Users/p.ilin/PycharmProjects/~/carmoney-lab                                        3329f96 [d1/1.2.1-1.2.3-pilin-dev]
C:/Users/p.ilin/PycharmProjects/~/carmoney-lab/.kilo/worktrees/flaxen-meat            3329f96 [flaxen-meat]
C:/Users/p.ilin/PycharmProjects/~/carmoney-lab/.kilo/worktrees/mountainous-payment    3329f96 [mountainous-payment]
C:/Users/p.ilin/PycharmProjects/~/carmoney-lab/.kilo/worktrees/righteous-grandparent  3329f96 (detached HEAD)
```

Две активные сессии Agent Manager:

- Основная (эта) — рабочая копия `~/carmoney-lab`, ветка `d1/1.2.1-1.2.3-pilin-dev`.
- «Обзор unit-тестов в tests/Unit» — worktree `.kilo/worktrees/mountainous-payment`, ветка `mountainous-payment`.

## Ответ агента из второй сессии

Промпт: «Не меняй файлы. Перечисли тесты в tests/Unit/: какой файл что проверяет,
одной строкой на файл. В конце скажи, в какой папке и на какой ветке ты сейчас
работаешь.»

Тесты в `tests/Unit/`:

- **VinValidatorTest.php** — проверяет формат VIN: длина 17, запрещённые символы (I/O/Q), регистр, спецсимволы, пустая строка.
- **LtvCalculatorTest.php** — проверяет расчёт LTV в процентах (половина/треть/больше стоимости) и исключения при нулевой стоимости или неположительной сумме.
- **DecisionEngineTest.php** — проверяет решение по LTV против порогов из конфига: approve ≤ 60, review ≤ 85, иначе reject, включая границы зон.
- **AssessmentServiceTest.php** — проверяет сквозную оценку заявки на реальных правилах из `rules.php`: LTV, решение (approve/review/reject), лимит и возраст авто.
- **ApplicationValidatorTest.php** — проверяет валидацию полей заявки: нормализация VIN в верхний регистр, отказ при годе из будущего, сумме ниже минимума, сбор всех ошибок сразу.

Работаю в папке `C:\Users\p.ilin\PycharmProjects\~\carmoney-lab\.kilo\worktrees\mountainous-payment`
на ветке `mountainous-payment`.

## Почему два агента в одной папке на одной ветке — плохая идея (★)

Оба агента писали бы в одни и те же файлы одной рабочей копии: их правки
перекрываются и затирают друг друга, а `git status` смешивает работу обоих, не
позволяя понять, кто что менял. Worktree даёт каждой сессии свою копию файлов
и свою ветку — конфликтов правок нет, и изменения каждой сессии видны отдельно.
