# JUnit5 + Gradle + Test IT + GitHub Actions

Пример автотестов на JUnit5 (Gradle), которые запускаются из Test IT через GitHub Actions и отправляют результаты обратно в Test IT с помощью [testit-adapter-junit5](https://github.com/testit-tms/adapters-java/tree/main/testit-adapter-junit5).

**Проект в Test IT:** [team-0tm5.testit.software/projects/360/autotests](https://team-0tm5.testit.software/projects/360/autotests)

## Как это работает

1. Test IT отправляет webhook в GitHub — событие `repository_dispatch` с типом `run-tests`.
2. Запускается workflow [.github/workflows/.github-ci.yml](.github/workflows/.github-ci.yml).
3. Тесты выполняются через `./gradlew test`, результаты загружаются в Test IT.
4. К прогону в Test IT прикрепляется ссылка на пайплайн GitHub Actions (`actions/runs/<run_id>`).

Адаптер подключается последней релизной версией (`latest.release` в [build.gradle](build.gradle)).

### Режимы запуска

Режим задаётся полем `adapter_mode` в webhook.

| `adapter_mode` | Что происходит | Имя прогона |
|---|---|---|
| `0` | Результаты пишутся в существующий прогон, `test_run_id` берётся из webhook. | `GitHub Actions #<run_number> (adapterMode=0)` |
| `2` | Адаптер сам создаёт новый прогон, `test_run_id` не передаётся. | `GitHub Actions #<run_number> (adapterMode=2)` |

### Sync-storage

Sync-storage запускается в workflow в обоих режимах (порт `49152`):

* запускается с `TMS_SYNC_STORAGE_AUTOCOMPLETE_FALLBACK_S=600`, чтобы результат автотеста, удержанный в sync-storage, всё равно был применён;
* после тестов workflow вызывает `/wait-completion` и останавливает sync-storage только после применения всех результатов;
* лог sync-storage сохраняется в артефакт `syncstorage-log-<run_id>-<attempt>` и хранится 1 сутки.

### Данные из webhook

```json
{
  "event_type": "run-tests",
  "client_payload": {
    "adapter_mode": "0",
    "url": "https://team-0tm5.testit.software",
    "project_id": "<id проекта>",
    "configuration_id": ["<id конфигурации>"],
    "test_run_id": "<id прогона, только для adapter_mode=0>"
  }
}
```

### Секреты репозитория

| Секрет | Назначение |
|---|---|
| `TMS_PRIVATE_TOKEN` | Приватный токен пользователя Test IT |

## Структура проекта

* **.github/workflows/.github-ci.yml** – workflow запуска тестов по webhook из Test IT
* **src/test/java/examples/** – тесты
    * **AnnotationTests.java** – примеры [аннотаций testit-adapter-junit5](https://github.com/testit-tms/adapters-java/tree/main/testit-adapter-junit5#annotations)
    * **MethodTests.java** – примеры [методов testit-adapter-junit5](https://github.com/testit-tms/adapters-java/tree/main/testit-adapter-junit5#annotations)
    * **StepsTests.java** – примеры [шагов testit-adapter-junit5](https://github.com/testit-tms/adapters-java/tree/main/testit-adapter-junit5#annotations)
* **src/test/resources/** – ресурсы для тестов
    * **attachments/** – файлы вложений
* **build.gradle** – сборка проекта и подключение адаптера

Конфигурация адаптера (URL, токен, проект, конфигурация, режим) передаётся из workflow через `-Dtms*` параметры, файла `testit.properties` в проекте нет.
