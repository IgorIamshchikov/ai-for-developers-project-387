# Календарь звонков (продолжение)


[![hexlet-check](https://github.com/IgorIamshchikov/ai-for-developers-project-387/actions/workflows/hexlet-check.yml/badge.svg)](https://github.com/IgorIamshchikov/ai-for-developers-project-387/actions)

Интегрируйте работу агентов в GitHub проект

Учебный проект Хекслета: https://ru.hexlet.io/programs/ai-for-developers
Как это должно работать: https://files.hexlet.app/a/2ipc5m

## Стек

- Разное

## Установка

<!-- Опишите установку: клонирование, зависимости, переменные окружения -->

```bash
git clone https://github.com/IgorIamshchikov/ai-for-developers-project-387.git
cd ai-for-developers-project-387
```

## Использование

<!-- Добавьте примеры запуска и запись asciinema — именно это смотрит работодатель -->

---

## Агент в GitHub

| Воркфлоу                      | Событие                                                                  | Модель                                     | Назначение                                                        | Где смотреть прогоны                                                                                           |
| ------------------------------ | ------------------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `claude.yml`                  | `issue_comment`, `pull_request_review_comment`, `pull_request_review`, `issues` (по команде `@claude`) | дефолтная модель `claude-code-action`        | разбор issue, правки по ревью, пояснения в треде                  | [вкладка Actions → Claude Code](https://github.com/IgorIamshchikov/ai-for-developers-project-387/actions/workflows/claude.yml) |
| `claude-code-review.yml`      | `pull_request` (opened/synchronize/ready_for_review/reopened)             | дефолтная модель `claude-code-action`        | авторевью каждого pull request, замечания инлайн и в обсуждении   | [вкладка Actions → Claude Code Review](https://github.com/IgorIamshchikov/ai-for-developers-project-387/actions/workflows/claude-code-review.yml) |
| `claude-issue-triage.yml`     | `issues: opened`                                                         | дефолтная модель `claude-code-action`        | автотриаж новых issue, расстановка triage-метки                   | [вкладка Actions → Claude Issue Triage](https://github.com/IgorIamshchikov/ai-for-developers-project-387/actions/workflows/claude-issue-triage.yml) |
| `lighthouse-schedule.yml`     | `schedule` (раз в сутки) + `workflow_dispatch`                           | дефолтная модель `claude-code-action`        | разбор отчёта Lighthouse, issue по находкам                       | [вкладка Actions → Lighthouse Schedule](https://github.com/IgorIamshchikov/ai-for-developers-project-387/actions/workflows/lighthouse-schedule.yml) |

Модель везде оставлена дефолтной: отдельного подбора по стоимости не делали, явного `--model` в `claude_args` нет ни у одного воркфлоу, расходы на сегодня не требуют различать задачи по дешевизне модели.

**Команда вызова.** Используется дефолтный `trigger_phrase` действия `anthropics/claude-code-action` — `@claude`. Сужать до своей команды не стали: единая команда проще для всех четырёх воркфлоу и уже используется во всех задачах цикла.

**Права.** У `claude.yml`, `claude-code-review.yml` и `claude-issue-triage.yml` — только read-права плюс `id-token: write`: сами операции с репозиторием агент выполняет от имени GitHub App, а не токеном раннера. Исключение — `lighthouse-schedule.yml`: там нет человека, который мог бы среагировать на находки, поэтому у воркфлоу `contents: write`, `pull-requests: write`, `issues: write`, чтобы агент сам завёл issue по результатам проверки.

**Круг вызывающих.** Репозиторий публичный, поэтому `claude.yml` реагирует только на события от пользователей с `author_association` `OWNER`, `MEMBER` или `COLLABORATOR`, и игнорирует события от ботов (`github.event.sender.type != 'Bot'`) — это отсекает и посторонних вызывающих, и холостые срабатывания на собственные комментарии агента, не тратя минуты Actions на раннер. Автоматические воркфлоу (`claude-code-review.yml`, `claude-issue-triage.yml`, `lighthouse-schedule.yml`) такого ограничения не имеют намеренно: они обрабатывают любой pull request / issue / расписание, а не чужую команду.

**Публикация сессий.** У `claude-code-action`, в отличие от OpenCode, нет параметра `share` — но в публичном репозитории прогоны во вкладке Actions и ответы агента в issue/PR видны всем без него. Это принятое решение: ключ модели лежит только в Secrets репозитория и в логах не появляется, а остальной контекст (issue, diff, промпт) и так публичен — то же, что видно любому читателю репозитория и без агента.

**Самооценка.** С первого прохода закрылись: воркфлоу автотриажа issue (`claude-issue-triage.yml`) и ограничение круга вызывающих в `claude.yml` — последнее подтверждено живым тестом на [issue #7](https://github.com/IgorIamshchikov/ai-for-developers-project-387/issues/7): легитимный вызов от владельца репозитория прошёл (`success`), собственный ответ агента с буквальным `@claude` в тексте и комментарий со стороннего аккаунта без прав — оба остановлены ещё до старта раннера (`skipped`), без правок после первого коммита. Потребовали итерации: интерактивный воркфлоу и авторевью (`claude.yml`, `claude-code-review.yml`) — в обоих после первого мержа отдельным коммитом добавляли `persist-credentials: false` в `checkout`, упущенное в первом варианте; и расписание Lighthouse (`lighthouse-schedule.yml`) — порог находок подобрали только после первого реального прогона, по факту увиденных метрик.

---

<details>
<summary>Автоматические тесты Хекслета</summary>

Тесты запускаются на каждый коммит. За запуск отвечает файл `.github/workflows/hexlet-check.yml` — не удаляйте и не переименовывайте ни его, ни репозиторий.

</details>

## О Хекслете

[Хекслет](https://ru.hexlet.io/) — школа программирования: авторские программы обучения с практикой, поддержкой наставников и реальными проектами, которые остаются в резюме. Этот репозиторий — один из таких проектов.
