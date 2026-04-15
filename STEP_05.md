# STEP_05.md — Коммерческие факторы (v1.0)

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4.5K токенов.

---

## ██ КАРТОЧКА ШАГА

| **Параметр** | **Значение** |
| --- | --- |
| ID | 05 |
| Название | Аудит коммерческих факторов |
| Блок | 2: Структура и контент |
| Основная роль | A: SEO-стратег |
| Консультативные | E: CRO-аналитик, D: Аналитик конкур. разведки, G: Инженер контрактов данных |
| Тип | Промпт+Скрипт |
| HitL | Нет (результат → Шаги 06, 08, 17 автоматически) |
| Приоритет ROI | КРИТИЧЕСКИЙ — КФ напрямую влияют на ранжирование в Яндексе и конверсию в обоих ПС |
| Зависит от | Шаг 04 (Структура страниц — список URL и типы), Шаг 04.2 (Сиротские страницы — для исключения orphan из аудита), PROJECT.md |
| Передаёт в | Шаг 06 (E-E-A-T аудит — пересечение trust-сигналов), Шаг 08 (ТЗ на страницы — требования к КФ-блокам), Шаг 11 (Метатеги — КФ-ключи в description), Шаг 17 (Финальный аудит — КФ-скоринг) |
| Режим Opus | JSON-вывод: temperature 0.2, top_p 0.9, max_tokens 8192 |
| Prompt version | step05_v1.0 |

---

## ██ ЦЕЛЬ

Провести комплексный аудит коммерческих факторов ранжирования по всем ключевым типам страниц сайта, оценить каждый фактор по балльной шкале с учётом специфики Яндекса ([Я]: ИКС, КФ, ПФ, региональность) и Google ([G]: E-E-A-T, CWV, доверие), сопоставить с конкурентами, и сформировать приоритизированный список issues с конкретными рекомендациями по устранению — для последующей интеграции в ТЗ (Шаг 08) и финальный аудит (Шаг 17).

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход формируется из output Шага 04 (Структура страниц) + PROJECT.md + результаты pre-scripts.

```json
{
  "project_id": "string",
  "domain": "string",
  "niche": "string",
  "niche_cluster": "string",
  "geo": "string",
  "priority_engines": "yandex|google|both",
  "cms": "string",
  "main_competitors": ["string"],
  "monthly_traffic": "number|null",
  "step_04_output": {
    "pages": [
      {
        "url": "string",
        "page_type": "main|category|product|service|article|contacts|about|delivery|faq|reviews|branch|other",
        "title": "string",
        "h1": "string"
      }
    ]
  },
  "crawl_snapshot": {
    "pages_data": [
      {
        "url": "string",
        "http_status": "number",
        "has_phone": "boolean",
        "has_email": "boolean",
        "has_address": "boolean",
        "has_price": "boolean",
        "has_cart": "boolean",
        "has_reviews": "boolean",
        "has_schema_org": "boolean",
        "has_ssl": "boolean",
        "has_privacy_policy": "boolean",
        "has_inn_ogrn": "boolean",
        "has_map": "boolean",
        "has_social_links": "boolean",
        "has_callback": "boolean",
        "has_whatsapp_telegram": "boolean",
        "has_working_hours": "boolean",
        "has_delivery_info": "boolean",
        "has_payment_info": "boolean",
        "has_return_policy": "boolean",
        "has_warranty": "boolean",
        "has_certificates": "boolean",
        "has_team_page": "boolean",
        "has_breadcrumbs": "boolean",
        "has_cta": "boolean",
        "images_count": "number",
        "images_with_alt": "number",
        "forms_count": "number"
      }
    ]
  },
  "competitor_cf_data": [
    {
      "domain": "string",
      "factors": {
        "phone_visible": "boolean",
        "price_on_pages": "boolean",
        "delivery_page": "boolean",
        "payment_page": "boolean",
        "reviews_section": "boolean",
        "certificates": "boolean",
        "inn_ogrn": "boolean",
        "map_on_contacts": "boolean",
        "chat_widget": "boolean",
        "social_links": "boolean",
        "working_hours": "boolean",
        "callback_form": "boolean"
      }
    }
  ]
}
```

**Обязательные поля:** `project_id`, `domain`, `niche_cluster`, `priority_engines`, `crawl_snapshot`.

**Опциональные поля:** `competitor_cf_data` (при отсутствии — аудит без бенчмарка, пометка `benchmark_available: false`), `step_04_output` (при отсутствии — аудит всех URL из crawl_snapshot).

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 2: Аудит со скорингом**. Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "05",
  "prompt_version": "step05_v1.0",
  "timestamp": "ISO 8601",
  "overall_score": "0-100",
  "issues": [
    {
      "id": "string (CF001, CF002...)",
      "category": "trust|contact|price|delivery|payment|legal|social|ux|conversion|content|local|technical",
      "severity": "critical|high|medium|low",
      "current_value": "string",
      "target_value": "string",
      "fix_description": "string",
      "estimated_impact": "1-10",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "affected_pages": ["string (URL или page_type)"],
      "effort": "1-10",
      "competitor_benchmark": "string|null",
      "confidence_tag": "[✓]|[◆]|[◇]|[⚠]",
      "cross_step_hooks": {
        "to_step_06_eeat": "boolean",
        "to_step_08_tz": "boolean",
        "to_step_11_meta": "boolean",
        "to_step_17_audit": "boolean"
      }
    }
  ],
  "passed_checks": ["string"],
  "category_scores": {
    "trust": "0-100",
    "contact": "0-100",
    "price": "0-100",
    "delivery": "0-100",
    "payment": "0-100",
    "legal": "0-100",
    "social": "0-100",
    "ux": "0-100",
    "conversion": "0-100",
    "content": "0-100",
    "local": "0-100"
  },
  "benchmark_summary": {
    "benchmark_available": "boolean",
    "our_score": "0-100",
    "avg_competitor_score": "0-100|null",
    "leader": "string|null",
    "gap_factors": ["string"]
  },
  "priority_engines_split": {
    "yandex_specific_issues": "number",
    "google_specific_issues": "number",
    "both_issues": "number"
  },
  "qc_score": "0-100"
}
```

**Правила:**
- Каждый `issues[].engine_tag` обязателен: `[Я]`, `[G]` или `[ОБА]`.
- `category` охватывает 11 направлений КФ (trust, contact, price, delivery, payment, legal, social, ux, conversion, content, local).
- `overall_score` = взвешенная сумма `category_scores` (веса зависят от `niche_cluster`).
- `cross_step_hooks` — явная маршрутизация для Шагов 06, 08, 11, 17.
- `passed_checks` — факторы, которые уже в норме (для положительного отчёта).

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

**Системный промпт**

```
Ты — Роль A: Senior SEO-стратег с 10+ летним опытом в Яндексе и Google.
Консультант: Роль E: CRO-аналитик (конверсия, поведение пользователей).

Задача: Провести аудит коммерческих факторов ранжирования сайта.

CoT (обязателен для каждого фактора):
[АНАЛИЗ] Какой КФ проверяем? Для какого ПС критичен?
[БЕНЧМАРК] Что у конкурентов? Стандарт ниши?
[РЕШЕНИЕ] Текущее состояние → целевое. Severity, impact.
[РИСКИ] Что будет если не исправить?

ПРАВИЛА:
1. Проверь ВСЕ 11 категорий КФ: trust, contact, price, delivery, payment, legal, social, ux, conversion, content, local
2. Каждая issue помечена engine_tag: [Я]|[G]|[ОБА]
3. Маркировка уверенности: [✓] данные подтверждены, [◆] логически обосновано, [◇] гипотеза, [⚠] риск
4. Специфика Яндекса [Я]:
   - ИКС напрямую зависит от КФ
   - Региональность: телефон с кодом города, адрес, карта
   - Яндекс.Бизнес / 2GIS обязательны для commercial_local
   - Полнота карточки товара/услуги (Яндекс особенно строг)
   - Цены обязательны на коммерческих страницах
   - ИНН/ОГРН в подвале — сильный сигнал
5. Специфика Google [G]:
   - E-E-A-T: компетентность, авторитет, доверие
   - Schema.org (LocalBusiness, Product, AggregateRating)
   - Google Business Profile — связь с сайтом
   - Reviews и рейтинги для rich snippets
   - Core Web Vitals косвенно связаны с КФ (user trust)
6. overall_score: взвешенная сумма категорий (веса по кластеру)
7. Учитывай niche_cluster:
   - commercial_local: контакты (30%), trust (25%), local (20%), price (15%), ux (10%)
   - ecom: price (25%), delivery (20%), payment (15%), trust (15%), conversion (15%), ux (10%)
   - medical: trust (30%), legal (25%), contact (20%), content (15%), ux (10%)
   - saas: conversion (25%), trust (20%), content (20%), ux (20%), price (15%)
   - legal: trust (30%), legal (25%), contact (20%), content (15%), ux (10%)
8. fix_description — конкретное действие (НЕ «улучшить», а «добавить ИНН/ОГРН в footer»)
9. estimated_impact: 1-10 (влияние на ранжирование/конверсию)
10. effort: 1-10 (1=5 минут, 10=несколько недель разработки)

АНТИПАТТЕРНЫ:
✗ Общие рекомендации без конкретики
✗ Рекомендации без engine_tag
✗ Пропуск хоть одной из 11 категорий
✗ Оценка без сравнения с конкурентами (если данные есть)
✗ fix_description длиннее 200 символов

ВЫВОД: Валидный JSON по контракту. Без markdown. Без комментариев. Весь текст — на РУССКОМ языке.
```

**Пользовательский промпт (шаблон)**

```
Проведи аудит коммерческих факторов:

Проект: {{project.project_id}}
Домен: {{project.domain}}
Ниша: {{project.niche}}
Кластер: {{project.niche_cluster}}
Гео: {{project.geo}}
Приоритет ПС: {{project.priority_engines}}
CMS: {{project.cms}}

Данные краулинга (КФ-сигналы по страницам):
{{crawl_snapshot.pages_data | json}}

Типы страниц (из Шага 04):
{{step_04_output.pages | json}}

{{#if competitor_cf_data}}
Данные конкурентов:
{{competitor_cf_data | json}}
{{/if}}

{{#if rag_context}}
Контекст из RAG:
{{rag_context}}
{{/if}}

Prompt version: step05_v1.0
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| **Коллекция Qdrant** | **Запрос** | **Назначение** |
| --- | --- | --- |
| `competitors` | `niche_cluster` + `geo` + «коммерческие факторы» | Бенчмарк КФ конкурентов, стандарты ниши |
| `niches` | `niche_cluster` + «чеклист КФ» | Шаблоны КФ для конкретного кластера (ecom vs medical vs legal) |
| `personas` | `project_id` | Персоны из Шага 00 — ожидания ЦА по КФ (доверие, контакты, цены) |
| `site_structure` | `project_id` + page_types | Текущая структура сайта для маппинга issues на конкретные URL |

Лимит RAG-контекста: 2000 токенов.

---

## ██ СКРИПТЫ CLAUDE CODE

### Пред-шаг 1: Краулер КФ-сигналов

```
# step_05_pre_cf_crawler.py
# Назначение: Извлечение КФ-сигналов со страниц сайта
# Вход: project.domain, step_04_output.pages[]
# Выход: crawl_snapshot.pages_data[]
# Действия:
# 1. Для каждого типа страниц (main, category, product, service, contacts, about, delivery):
#    взять до 5 представителей
# 2. Для каждой страницы извлечь КФ-сигналы:
#    a) Контакты: телефон (regex +7/8-xxx), email, адрес, карта (iframe яндекс.карты/google maps)
#    b) Цены: наличие цифр с ₽/руб/рублей, "от X руб", "цена по запросу"
#    c) Корзина/заказ: <form>, кнопки "В корзину", "Заказать", "Купить"
#    d) Отзывы: блоки reviews, Schema AggregateRating
#    e) Юридическое: ИНН/ОГРН (regex), ссылки на /privacy/, /oferta/, /agreement/
#    f) Социальные: ссылки на vk.com, t.me, ok.ru, youtube.com, instagram (для [G])
#    g) Доверие: SSL (https), сертификаты (img с "сертификат"/"certificate"), лицензии
#    h) UX: breadcrumbs (<nav> или Schema BreadcrumbList), CTA-кнопки, формы
#    i) Доставка/Оплата: ссылки /delivery/, /payment/, /shipping/, /oplata/
#    j) Гарантия/Возврат: /warranty/, /vozvrat/, /return-policy/
#    k) Рабочие часы: "Пн-Пт", "ежедневно", "график работы"
#    l) Команда: /team/, /about/, /o-kompanii/ + фото сотрудников
#    m) WhatsApp/Telegram: wa.me, t.me виджеты и ссылки
#    n) Callback: формы обратного звонка, виджеты
# 3. Для изображений: подсчёт total + с alt
# 4. HTTP-статус каждой страницы
# 5. Нормализация результатов → crawl_snapshot.json
#
# Ограничения: robots.txt, delay 300мс, User-Agent: SEOBot/1.0
# Таймаут: 30с на URL
# Fallback: при блокировке → пометить url, продолжить
```

### Пред-шаг 2: Парсинг КФ конкурентов

```
# step_05_pre_competitor_cf.py
# Назначение: Извлечение КФ-сигналов с конкурентных сайтов
# Вход: project.main_competitors[], project.niche_cluster
# Выход: competitor_cf_data[]
# Действия:
# 1. Для каждого конкурента: главная + /contacts/ + /delivery/ + /about/
# 2. Извлечь те же КФ-сигналы что и для основного сайта (сокращённо):
#    - phone_visible, price_on_pages, delivery_page, payment_page
#    - reviews_section, certificates, inn_ogrn, map_on_contacts
#    - chat_widget, social_links, working_hours, callback_form
# 3. Сохранить competitor_cf_data.json
# 4. Рассчитать avg_competitor_score (% факторов present у каждого конкурента)
#
# Ограничения: 4 страницы на конкурента макс, delay 500мс
# Fallback: конкурент недоступен → пометить, продолжить без него
```

### Пред-шаг 3: Проверка Яндекс.Бизнес / GBP

```
# step_05_pre_business_profiles.py
# Назначение: Проверка наличия и полноты бизнес-профилей
# Вход: project.domain, project.geo, project.priority_engines
# Выход: business_profiles_data.json
# Действия:
# 1. Яндекс.Бизнес: проверить привязку домена (поиск "site:domain" + Яндекс.Карты API)
# 2. 2GIS: проверить наличие организации
# 3. GBP (Google Business Profile): проверить через Google Places API
# 4. Для каждого найденного профиля:
#    - Заполненность (название, адрес, телефон, часы, фото, отзывы)
#    - Рейтинг и количество отзывов
#    - Связь с сайтом (URL в профиле)
# 5. Fallback: API недоступен → пометка, продолжить
```

### Пост-шаг: Валидатор

```
# step_05_post_validate.py
# Назначение: Валидация выхода по JSON-схеме и QC-критериям
# Проверки:
# 1. JSON Schema — структура соответствует Template 2
# 2. Все issues имеют id (формат CF###)
# 3. Все issues имеют engine_tag ∈ {[Я], [G], [ОБА]}
# 4. overall_score ∈ [0, 100]
# 5. Все 11 категорий представлены в category_scores
# 6. Каждая category_scores ∈ [0, 100]
# 7. severity ∈ {critical, high, medium, low}
# 8. issues ≥ 5 (мин. 5 проблем для любого сайта)
# 9. passed_checks ≥ 1 (хотя бы что-то хорошо)
# 10. fix_description ≤ 200 символов
# 11. При priority_engines=both → есть [Я] И [G] issues
# 12. confidence_tag присутствует у каждого issue
# 13. cross_step_hooks заполнены (хотя бы to_step_08_tz=true у critical/high)
# 14. estimated_impact и effort ∈ [1, 10]
# 15. benchmark_summary.benchmark_available корректен (true если данные конкурентов есть)
#
# При провале: вернуть список ошибок → retry
```

### Пост-шаг: NocoDB + Qdrant

```
# step_05_post_store.py
# Назначение: Сохранение результатов
# Действия:
# 1. NocoDB: INSERT в таблицу cf_audit (project_id, step_id, timestamp, output JSON)
# 2. NocoDB: UPDATE step_results (step_id=05, status=done, qc_score)
# 3. NocoDB: INSERT audit_log (prompt_version, model_version, token_usage)
# 4. Qdrant: upsert в коллекцию 'cf_audit' — embedding(category + fix_description)
#    для использования в Шаге 08 (ТЗ) и Шаге 17 (Финальный аудит)
# 5. Тег project_id, prompt_version
# 6. Idempotency: sha256(project_id + step_id + crawl_date) для защиты от дублей
```

---

## ██ КРИТЕРИИ QC

| **Проверка** | **Порог** | **Вес** | **Тип** | **При провале** |
| --- | --- | --- | --- | --- |
| JSON Schema валидна | Pass | 15 | hard | Retry |
| Все 11 категорий в category_scores | 11/11 | 15 | hard | Retry |
| Engine теги | 100% issues имеют tag | 15 | hard | Retry |
| overall_score в [0,100] | Pass | 5 | hard | Retry |
| Issues ≥ 5 | Pass | 10 | hard | Retry |
| fix_description ≤ 200 символов | 100% | 5 | hard | Retry |
| passed_checks ≥ 1 | Pass | 5 | hard | Retry |
| При both → есть [Я] И [G] | Pass | 10 | hard | Retry |
| Конкурентный бенчмарк согласован | benchmark_available корректен | 5 | soft | Ревью |
| confidence_tag у всех issues | 100% | 5 | soft | Ревью |
| cross_step_hooks у critical/high | ≥ to_step_08=true | 5 | soft | Ревью |
| Нет дублей id | Pass | 5 | hard | Retry |
| Категория scores ∈ [0,100] | 100% | 5 | hard | Retry |

QC = сумма весов пройденных. ≥80 + все hard → Шаг 06; 60-79 → ревью; <60 → retry.

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (после завершения Шага 04.2 или ручной запуск)
Нода 2:  PROJECT.md + output Шага 04 из NocoDB
Нода 3:  [Параллельный запуск]:
         ├─ 3a: step_05_pre_cf_crawler.py → crawl_snapshot.json
         ├─ 3b: step_05_pre_competitor_cf.py → competitor_cf_data.json
         └─ 3c: step_05_pre_business_profiles.py → business_profiles.json
Нода 4:  Merge — объединение всех pre-step данных
Нода 5:  Qdrant RAG — запрос 4 коллекций (competitors, niches, personas, site_structure)
Нода 6:  Сборка промпта (system + user с переменными + RAG-контекст)
Нода 7:  Claude Opus API (temperature 0.2, top_p 0.9, max_tokens 8192)
Нода 8:  Парсинг JSON (strip markdown-обёртки если есть)
Нода 9:  step_05_post_validate.py
         └─ провал hard → Нода 6 (retry, макс. 3, инъекция ошибок QC)
Нода 10: step_05_post_store.py → NocoDB + Qdrant
Нода 11: Grafana metrics push (overall_score, issues_count, category_scores)
Нода 12: Передача → Шаг 06 (E-E-A-T) или стоп
```

**Обработка ошибок:**

```
• Краулер таймаут/блокировка: уменьшить выборку до 3 страниц/тип, retry
• Конкурент недоступен: пропустить, benchmark_available = false если все конкуренты недоступны
• Opus 429/500: бэкофф 10с→30с→90с, макс. 3
• Qdrant недоступен: без RAG, лог предупреждения
• QC провал 3x: алерт Telegram с логом 3 попыток
• Невалидный JSON от Opus: strip ```json```, повторный парсинг → retry
• Яндекс.Бизнес/GBP API недоступен: пропустить проверку, пометка в output
```

---

## ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "05",
  "prompt_version": "step05_v1.0",
  "timestamp": "2025-12-20T10:00:00Z",
  "overall_score": 52,
  "issues": [
    {
      "id": "CF001",
      "category": "contact",
      "severity": "critical",
      "current_value": "Телефон только на главной, нет на внутренних",
      "target_value": "Телефон в шапке на всех страницах + кнопка звонка на мобильных",
      "fix_description": "Добавить телефон +7(495)XXX-XX-XX в header сайта. Для мобильных — кнопку tel: с иконкой.",
      "estimated_impact": 9,
      "engine_tag": "[Я]",
      "affected_pages": ["category", "service", "product"],
      "effort": 2,
      "competitor_benchmark": "3/3 конкурента имеют телефон на всех страницах",
      "confidence_tag": "[✓]",
      "cross_step_hooks": {
        "to_step_06_eeat": false,
        "to_step_08_tz": true,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF002",
      "category": "price",
      "severity": "critical",
      "current_value": "Цены отсутствуют на 8 из 12 страниц услуг",
      "target_value": "Цены (или 'от X руб') на каждой странице услуги",
      "fix_description": "Добавить блок с ценами на все /uslugi/* страницы. Формат: таблица услуга — цена — 'Записаться'.",
      "estimated_impact": 10,
      "engine_tag": "[Я]",
      "affected_pages": [
        "https://dental-moscow.ru/uslugi/implantaciya",
        "https://dental-moscow.ru/uslugi/protezirovanie",
        "https://dental-moscow.ru/uslugi/ortodontiya"
      ],
      "effort": 4,
      "competitor_benchmark": "2/3 конкурента указывают цены на всех услугах",
      "confidence_tag": "[✓]",
      "cross_step_hooks": {
        "to_step_06_eeat": false,
        "to_step_08_tz": true,
        "to_step_11_meta": true,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF003",
      "category": "trust",
      "severity": "high",
      "current_value": "Нет ИНН/ОГРН на сайте",
      "target_value": "ИНН и ОГРН в футере всех страниц",
      "fix_description": "Добавить в footer: ООО «Дентал», ИНН 7701234567, ОГРН 1177700001234.",
      "estimated_impact": 7,
      "engine_tag": "[Я]",
      "affected_pages": ["all"],
      "effort": 1,
      "competitor_benchmark": "2/3 конкурента указывают ИНН/ОГРН",
      "confidence_tag": "[✓]",
      "cross_step_hooks": {
        "to_step_06_eeat": true,
        "to_step_08_tz": true,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF004",
      "category": "legal",
      "severity": "high",
      "current_value": "Нет лицензий на сайте (медицинская ниша)",
      "target_value": "Сканы лицензий + страница /licenzii/ со Schema MedicalOrganization",
      "fix_description": "Создать /licenzii/ с фото лицензий, добавить Schema MedicalOrganization с licence-полем.",
      "estimated_impact": 9,
      "engine_tag": "[ОБА]",
      "affected_pages": ["main", "about", "new:/licenzii/"],
      "effort": 3,
      "competitor_benchmark": "3/3 конкурента публикуют лицензии",
      "confidence_tag": "[✓]",
      "cross_step_hooks": {
        "to_step_06_eeat": true,
        "to_step_08_tz": true,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF005",
      "category": "local",
      "severity": "high",
      "current_value": "Нет Яндекс.Бизнес профиля",
      "target_value": "Верифицированный профиль Яндекс.Бизнес с рейтингом ≥4.5",
      "fix_description": "Создать и верифицировать организацию в Яндекс.Бизнес. Заполнить: фото, часы, услуги, цены.",
      "estimated_impact": 8,
      "engine_tag": "[Я]",
      "affected_pages": ["contacts"],
      "effort": 3,
      "competitor_benchmark": "3/3 конкурента зарегистрированы в Яндекс.Бизнес",
      "confidence_tag": "[✓]",
      "cross_step_hooks": {
        "to_step_06_eeat": false,
        "to_step_08_tz": false,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF006",
      "category": "ux",
      "severity": "medium",
      "current_value": "Schema.org отсутствует полностью",
      "target_value": "LocalBusiness на главной, MedicalOrganization, AggregateRating",
      "fix_description": "Внедрить JSON-LD: LocalBusiness (главная), Service (услуги), AggregateRating (отзывы).",
      "estimated_impact": 6,
      "engine_tag": "[G]",
      "affected_pages": ["main", "service", "reviews"],
      "effort": 4,
      "competitor_benchmark": "2/3 конкурента используют Schema.org",
      "confidence_tag": "[◆]",
      "cross_step_hooks": {
        "to_step_06_eeat": true,
        "to_step_08_tz": true,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF007",
      "category": "social",
      "severity": "medium",
      "current_value": "Нет ссылок на соцсети",
      "target_value": "VK, Telegram, YouTube в футере + Schema sameAs",
      "fix_description": "Добавить в footer иконки VK, Telegram, YouTube. В Schema LocalBusiness — sameAs с URL профилей.",
      "estimated_impact": 4,
      "engine_tag": "[ОБА]",
      "affected_pages": ["all"],
      "effort": 2,
      "competitor_benchmark": "2/3 конкурента имеют соцсети в footer",
      "confidence_tag": "[◆]",
      "cross_step_hooks": {
        "to_step_06_eeat": true,
        "to_step_08_tz": true,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    },
    {
      "id": "CF008",
      "category": "conversion",
      "severity": "medium",
      "current_value": "Форма записи только на странице контактов",
      "target_value": "CTA 'Записаться' на каждой странице услуги + попап",
      "fix_description": "Добавить кнопку 'Записаться на приём' + попап-форму (имя, телефон) на все /uslugi/* страницы.",
      "estimated_impact": 7,
      "engine_tag": "[ОБА]",
      "affected_pages": ["service", "category"],
      "effort": 4,
      "competitor_benchmark": "3/3 конкурента имеют формы на услугах",
      "confidence_tag": "[✓]",
      "cross_step_hooks": {
        "to_step_06_eeat": false,
        "to_step_08_tz": true,
        "to_step_11_meta": false,
        "to_step_17_audit": true
      }
    }
  ],
  "passed_checks": [
    "SSL-сертификат активен (https)",
    "Страница /contacts/ существует с адресом и картой",
    "Хлебные крошки на внутренних страницах",
    "Страница /delivery/ существует"
  ],
  "category_scores": {
    "trust": 35,
    "contact": 40,
    "price": 25,
    "delivery": 70,
    "payment": 60,
    "legal": 20,
    "social": 15,
    "ux": 45,
    "conversion": 35,
    "content": 55,
    "local": 30
  },
  "benchmark_summary": {
    "benchmark_available": true,
    "our_score": 52,
    "avg_competitor_score": 74,
    "leader": "smile-clinic.ru",
    "gap_factors": ["price", "legal", "trust", "local", "social"]
  },
  "priority_engines_split": {
    "yandex_specific_issues": 3,
    "google_specific_issues": 1,
    "both_issues": 4
  },
  "qc_score": 89
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

```
Откат A: Краулер не завершился (таймаут/блокировка).
  → Уменьшить выборку до 3 страниц на тип (вместо 5).
  → Если всё равно fail → использовать только главную + contacts + delivery.
  → Пометка: partial_crawl = true, confidence всех issues *= 0.7.

Откат B: Конкуренты недоступны.
  → benchmark_available = false.
  → competitor_benchmark = null у всех issues.
  → Opus оценивает по абсолютным стандартам ниши (из RAG-контекста).

Откат C: Opus не может сгенерировать валидный JSON (3x fail).
  → Декомпозиция: разбить на 3 вызова по 4 категории.
  → Вызов 1: trust, contact, price, delivery.
  → Вызов 2: payment, legal, social, ux.
  → Вызов 3: conversion, content, local + merge + overall_score.
  → Объединить результаты.

Откат D: Qdrant недоступен (нет RAG).
  → Opus оценивает без конкурентного контекста.
  → Все confidence_tag ≤ [◆] (без подтверждённого бенчмарка).
  → Пометка: rag_unavailable = true.

Откат E: API бизнес-профилей недоступны.
  → local-категория оценивается по наличию /contacts/ + карты на странице.
  → business_profiles_data = null.
  → confidence для local issues: [◇].
```

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

```
• commercial_local:
  — Веса: contact (30%), trust (25%), local (20%), price (15%), ux (10%)
  — Обязательные КФ: телефон с кодом города, адрес, карта, Яндекс.Бизнес, 2GIS
  — Микрогеография: проверка NAP-consistency (Name, Address, Phone)
  — Рабочие часы: обязательны, в формате Яндекса
  — Фото фасада/интерьера: проверка наличия на сайте и в бизнес-профиле

• commercial_national:
  — Веса: trust (25%), conversion (20%), price (20%), delivery (15%), ux (10%), content (10%)
  — Фокус: прайс-листы, калькуляторы, формы заявки
  — 8-800 номер (бесплатный для клиентов)
  — Портфолио / кейсы — доверие на национальном уровне

• ecom:
  — Веса: price (25%), delivery (20%), payment (15%), trust (15%), conversion (15%), ux (10%)
  — Полнота карточки товара: фото (≥3), описание (≥300 слов), характеристики, цена, наличие
  — Корзина + оформление заказа: работоспособность, шаги, способы оплаты
  — Доставка: сроки, стоимость, калькулятор, регионы
  — Возврат/обмен: обязателен по закону, проверка наличия
  — Schema Product + Offer + AggregateRating

• saas:
  — Веса: conversion (25%), trust (20%), content (20%), ux (20%), price (15%)
  — Pricing page: тарифы, сравнение, CTA
  — Trial/Demo: наличие бесплатной версии
  — Интеграции: список партнёров, API-документация
  — Case studies / отзывы клиентов
  — Security / compliance страница

• info:
  — Веса: content (30%), trust (25%), ux (20%), social (15%), contact (10%)
  — Минимальные КФ: об авторе, контакты, подписка
  — Фокус на E-E-A-T (передача в Шаг 06)
  — Реклама не перегружает контент

• medical:
  — Веса: trust (30%), legal (25%), contact (20%), content (15%), ux (10%)
  — YMYL: лицензии ОБЯЗАТЕЛЬНЫ, профили врачей ОБЯЗАТЕЛЬНЫ
  — Запись на приём: онлайн-форма на каждой услуге
  — Прайс-лист: обязателен по закону (частная медицина)
  — ОГРН, ИНН, лицензии: footer + /licenzii/
  — Schema MedicalOrganization, Physician

• legal:
  — Веса: trust (30%), legal (25%), contact (20%), content (15%), ux (10%)
  — Статус адвоката / лицензия: обязательно
  — Кейсы / результаты: доверие
  — Бесплатная консультация: CTA
  — Конфиденциальность: политика обработки данных

• education:
  — Веса: content (25%), trust (20%), conversion (20%), price (20%), ux (15%)
  — Программы: детальное описание, сроки, стоимость
  — Лицензия на образовательную деятельность
  — Отзывы студентов / выпускников
  — Диплом / сертификат по окончании
  — Сезонность: приём, даты начала
```

---

## ██ МЕТРИКИ УСПЕХА

```
Опережающий:
  — overall_score рассчитан (не null)
  — Все 11 категорий покрыты в category_scores
  — Issues ≥ 5 с конкретными fix_description
  — Конкурентный бенчмарк доступен (benchmark_available = true)
  — Engine split: есть [Я] и [G] issues при priority_engines=both

Запаздывающий:
  — После Шага 08: ≥80% critical/high issues включены в ТЗ
  — После Шага 17: overall_score повторного аудита ≥ +20 пунктов
  — Через 60 дней: рост ИКС (Яндекс) ≥ 10%
  — Через 90 дней: рост конверсии ≥ 5% (формы, звонки)

Контроль:
  — Ревью после Шага 08: fix_description достаточно конкретны для разработки?
  — Бенчмарк верен: ручная проверка 3 issues vs конкурент (≥80% совпадение)
  — False positive rate ≤ 10% (issues действительно нерелевантны)
  — Стабильность: повторный запуск даёт ±5 пунктов overall_score
```

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2025-12 | Базовая версия: 11 категорий КФ, Template 2, Prompt+Script, бенчмарк конкурентов, кластерные веса, бизнес-профили, cross_step_hooks |
