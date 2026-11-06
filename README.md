# Verdict

> DAST-сканер с доказуемыми находками: не просто находит, а подтверждает.

Verdict обходит веб-приложение, запускает набор проверок безопасности и выдаёт отчёт, в котором каждая находка воспроизводима: сырой запрос, сырой ответ и уровень уверенности (`Suspect` / `Likely` / `Confirmed`).

Отличие от ZAP и Nuclei не в количестве проверок, а в том, что находка считается подтверждённой только после эксперимента с контрольным отличием. Эвристика без доказательств не выдаётся за результат.

> **Статус:** проект в активной разработке (MVP). Список реализованного см. в [Coverage matrix](#coverage-matrix).

## Зачем это нужно

- **Изучающим ИБ:** наглядно показывает, как устроены OWASP-проверки, HTTP и верификация уязвимостей.
- **Разработчикам:** быстрый чек в CI через SARIF и `--fail-on high`.
- **Для портфолио:** проект с измеренным качеством (таблица на DVWA и Juice Shop).

## Возможности

- Verification-first: `Confirmed` требует минимум двух запросов с воспроизводимым различием. Это проверяется контрактным тестом.
- Каждая находка содержит `Evidence` (сырой запрос и ответ на каждом шаге) и стабильный `fingerprint` для сравнения сканов.
- Отчёты в JSON, SARIF 2.1.0 и текстовом виде.
- Фильтр `--fail-on` для использования в CI.
- Ядро без Qt: его можно собрать и запустить на сервере без GUI.
- GUI на Qt6/QML (в разработке, делается последним).

## Пример

```bash
verdict --url https://example.test --output report.sarif --format sarif --fail-on high
```

Пример фрагмента находки:

```json
{
  "id": "xss.reflected",
  "severity": "High",
  "confidence": "Confirmed",
  "url": "https://example.test/search?q=",
  "evidence": [
    { "step": "probe", "request": "GET /search?q=VERDICT_A1", "response_status": 200 },
    { "step": "control", "request": "GET /search?q=VERDICT_B2", "response_status": 200 }
  ],
  "fingerprint": "3f9a1c..."
}
```

## Coverage matrix

Verdict не обещает покрытие всего OWASP Top 10. Ниже честный статус.

| Категория OWASP | Проверки | Статус |
|---|---|---|
| A02 Cryptographic Failures | TLS-версия, срок сертификата | planned |
| A05 Security Misconfiguration | Заголовки безопасности, CORS, экспозиция `.env` / `.git` | planned |
| A01 Broken Access Control | Open redirect | planned |
| A03 Injection | Reflected XSS (двойной маркер), SQLi (true/false) | planned |
| Прочее | Cookie flags (Secure, HttpOnly, SameSite) | planned |

Статусы обновляются по мере реализации: `implemented` / `planned` / `out of scope`.

## Архитектура

```
cli, gui          тонкие обёртки: парсинг аргументов и отображение
   │
core              оркестрация, краулер, отчёты (без Qt)
   │
modules           проверки, каждая независима
   │
infra             HTTP, TLS, URL, HTML, контракты Finding / Evidence / IModule
```

Инварианты, которые проверяет сборка и CI:

1. `core` не зависит от Qt. Проверяется скриптом `scripts/check-core-no-qt.sh` в CI.
2. `modules` знают только `infra` и контракты `IModule` / `ScanContext`.
3. `Finding` иммутабелен и передаётся по значению.
4. `testing` подключается только к тестам и никогда не попадает в продуктовые бинарники.

Подробнее: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Сборка

Требования: CMake 3.24+, компилятор с поддержкой C++20, vcpkg.

```bash
cmake --preset debug
cmake --build --preset debug
ctest --preset debug
```

Опции сборки (`cmake/options.cmake`): `ENABLE_GUI`, `ENABLE_TESTS`, `ENABLE_SANITIZERS`, `ENABLE_LLM`, `ENABLE_STORAGE`. По умолчанию GUI, LLM и SQLite выключены, MVP собирается без них.

## Security considerations

Verdict отправляет активные запросы, поэтому используйте его только против систем, на тестирование которых у вас есть письменное разрешение.

- Тестируйте только собственные приложения, учебные стенды (DVWA, Juice Shop) или цели, для которых есть явное разрешение владельца.
- Встроенный rate limit защищает от перегрузки цели. Не отключайте его без необходимости.
- Активные проверки (XSS, SQLi, Open Redirect) можно отключить флагом, оставив только пассивный анализ.
- Verdict не выполняет деструктивных payload'ов (например, `DROP TABLE`) и не выполняет JavaScript в браузере.
- LLM не используется для обнаружения уязвимостей. В будущем он может только фильтровать и объяснять уже доказанные находки.

Подробнее: [docs/SECURITY.md](docs/SECURITY.md) и [docs/THREAT_MODEL.md](docs/THREAT_MODEL.md).

## Бенчмарк

Метрики будут опубликованы после реализации MVP: findings, ложные срабатывания и время скана на DVWA и Juice Shop, в сравнении с ZAP baseline. Стенд описан в [bench/](bench/).

| Цель | Находки | FP | Время | ZAP baseline |
|---|---|---|---|---|
| DVWA | — | — | — | — |
| Juice Shop | — | — | — | — |

## Дорожная карта

- **MVP:** verification-ядро, 8 модулей, краулер, CLI с JSON, SARIF и `--fail-on`, GitHub Action.
- **Phase 2:** YAML-шаблоны проверок, история сканов в SQLite, сравнение по fingerprint, вход по форме и cookie jar.
- **Phase 3:** LLM-триаж как отключаемый слой над `Likely`-находками, импорт OpenAPI.
- **Phase 4:** прокси-режим как отдельный продукт.

## Лицензия

Указана в файле [LICENSE](LICENSE).
