# STEP_01.md — Сбор семантического ядра (СЯ)

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4K токенов.

---

# ██ КАРТОЧКА ШАГА

| **Параметр**        | **Значение**                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| ID                  | 01                                                                          |
| Название            | Сбор семантического ядра (СЯ)                                              |
| Блок                | 1: Подготовительный                                                         |
| Основная роль       | A: SEO-стратег                                                              |
| Консультативные     | B: Инженер автоматизации, D: Аналитик конкурентной разведки, F: Промпт-инженер |
| Тип                 | Промпт+Скрипт                                                               |
| JSON-шаблон         | Шаблон 1: Список с приоритетами                                             |
| HitL                | Нет (результат → Шаг 02 автоматически)                                      |
| Приоритет ROI       | КРИТИЧЕСКИЙ — семантика = фундамент всей структуры и контента               |
| Зависит от          | Шаг 00 (Анализ ЦА: персоны, seed_queries), Шаг 00.2 (Аудит позиций: enhanced_seed_queries, baseline) |
| Передаёт в          | Шаг 02 (Структура сайта), Шаг 04 (Структура страниц), Шаг 08 (ТЗ на страницы) |
| Режим Opus          | JSON-вывод: temperature 0.15, top_p 0.9, max_tokens 8192                   |

---

# ██ ЦЕЛЬ

Собрать, кластеризовать и приоритизировать полное семантическое ядро проекта на основе персон (Шаг 00), текущих позиций (Шаг 00.2) и данных внешних API (Wordstat, KeySO, Serpstat). Каждый кластер = группа запросов с единым интентом, привязанная к персоне и помеченная engine-тегом. Результат — приоритизированный список кластеров, готовый для проектирования структуры сайта (Шаг 02).

---

# ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход формируется из output contract Шагов 00 и 00.2. Минимальные обязательные поля:

```json
{
  "project_id": "string",
  "niche": "string",
  "niche_cluster": "commercial_local|commercial_national|ecom|saas|info|medical|legal|education",
  "geo": "string",
  "language": "string",
  "priority_engines": "yandex|google|both",
  "domain": "string",
  "main_competitors": ["string"],
  "personas": [
    {
      "persona_id": "string (P001...)",
      "persona_name": "string",
      "segment": "B2C|B2B|B2B2C",
      "jtbd": {
        "core_job": "string",
        "functional_jobs": ["string"],
        "triggers": ["string"]
      },
      "hunt_ladder": {
        "level_1_unaware": { "sample_queries": [{"query":"string","engine":"string","intent":"string"}] },
        "level_2_problem_aware": { "sample_queries": [] },
        "level_3_solution_aware": { "sample_queries": [] },
        "level_4_product_aware": { "sample_queries": [] },
        "level_5_most_aware": { "sample_queries": [] }
      },
      "search_behavior": {
        "primary_engine": "string",
        "ai_search_usage": "none|sometimes|primary"
      },
      "priority_score": "number 1-100"
    }
  ],
  "seed_queries": [
    {"query":"string","engine":"string","persona_id":"string","hunt_level":"number"}
  ],
  "enhanced_seed_queries": [
    {"query":"string","source":"step00|step00+gsc|quick_win","position":"number|null","impressions":"number|null","engine":"string","persona_id":"string|null"}
  ],
  "baseline_snapshot": {
    "total_keywords_tracked": "number",
    "position_distribution": {"top3":"num","top10":"num","top20":"num","top50":"num"},
    "avg_position": "number",
    "branded_traffic_pct": "number"
  },
  "quick_wins": [
    {"query":"string","current_position":"number","estimated_traffic_gain":"number"}
  ]
}
```

---

# ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 1: Список с приоритетами**. Роль G валидирует. Расширен под специфику СЯ.

```json
{
  "project_id": "string",
  "step_id": "01",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "clusters": [
    {
      "cluster_id": "string (CL001...)",
      "cluster_name": "string — название кластера на русском",
      "intent": "informational|navigational|commercial|transactional|mixed",
      "hunt_level": "1|2|3|4|5",
      "mapped_personas": ["string (persona_id)"],
      "priority": "critical|high|medium|low",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "estimated_impact": "1-10",
      "effort": "1-10",
      "monthly_volume_total": "number",
      "competition_level": "low|medium|high",
      "queries": [
        {
          "query": "string",
          "volume_yandex": "number|null",
          "volume_google": "number|null",
          "engine": "yandex|google|both",
          "intent": "informational|navigational|commercial|transactional",
          "is_long_tail": "boolean",
          "current_position": "number|null",
          "source": "seed|wordstat|keyso|serpstat|suggest|gsc|manual",
          "ai_search_relevant": "boolean"
        }
      ],
      "page_type_suggestion": "landing|category|article|faq|product|hub",
      "niche_cluster_specific": "string|null — адаптация под кластер"
    }
  ],
  "items": [
    {
      "id": "string (= cluster_id)",
      "title": "string (= cluster_name)",
      "priority": "critical|high|medium|low",
      "action_required": "string — что делать с кластером на Шаге 02",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "estimated_impact": "1-10",
      "effort": "1-10"
    }
  ],
  "summary": {
    "total_clusters": "number",
    "total_queries": "number",
    "critical": "number",
    "high": "number",
    "medium": "number",
    "low": "number",
    "by_intent": {
      "informational": "number",
      "commercial": "number",
      "transactional": "number",
      "navigational": "number"
    },
    "by_engine": {
      "yandex_only": "number",
      "google_only": "number",
      "both": "number"
    },
    "coverage_by_persona": [
      {"persona_id": "string", "clusters_count": "number", "queries_count": "number"}
    ]
  },
  "unclustered_queries": [
    {"query": "string", "volume": "number", "engine": "string", "reason": "string"}
  ],
  "data_sources_used": ["string — wordstat|keyso|serpstat|gsc|suggest|manual"],
  "qc_score": "0-100"
}
```

---

# ██ ПРОМПТ ДЛЯ CLAUDE OPUS

## Системный промпт

```
Ты — Роль A: Senior SEO-стратег с 10+ летним опытом кластеризации семантики для русскоязычного рынка.
Консультативные роли: D (конкурентная разведка), F (промпт-инженер).

Задача: Кластеризовать и приоритизировать семантическое ядро на основе собранных запросов, персон и данных позиций.

ПРАВИЛА:
1. Каждый кластер — группа запросов с ЕДИНЫМ поисковым интентом. Один кластер = одна будущая страница.
2. Минимум 15 кластеров, максимум 100 (зависит от ниши).
3. Каждый кластер ОБЯЗАТЕЛЬНО с engine_tag: [Я], [G] или [ОБА].
4. Каждый кластер привязан к ≥1 persona_id из Шага 00.
5. Каждый кластер имеет hunt_level (1-5) по Лестнице Ханта.
6. Приоритизация по формуле:
   priority_score = (monthly_volume_total × 0.3) + (estimated_impact × 0.3) + (persona_priority × 0.2) + (competition_inverse × 0.2)
   critical: score ≥ 80, high: 60-79, medium: 40-59, low: < 40
7. page_type_suggestion — обязателен для каждого кластера:
   - transactional high-volume → landing
   - informational → article
   - вопросы (как, почему, что) → faq
   - товарные → product/category
8. Запросы из quick_wins (Шаг 00.2) автоматически получают приоритет ≥ high.
9. Запросы с ai_search_relevant: true — помечай для AEO/GEO-оптимизации (Шаги 29-30).
10. Учитывай niche_cluster проекта:
    - commercial_local: гео-модификаторы (район, метро, «рядом»), NAP-запросы
    - ecom: товарные фильтры (цена, размер, бренд), «купить», «заказать»
    - medical: симптомы (уровень 1-2), YMYL, «врач», «клиника»
    - saas: «альтернативы X», интеграции, pricing, vs-запросы
    - info: long-tail вопросы, how-to, гайды
11. НЕ включай branded-запросы конкурентов. Собственные branded — отдельный кластер с priority: low.
12. Не угадывай volume — если данных нет, ставь null.
13. unclustered_queries — запросы, которые не подошли ни к одному кластеру (с reason).

ВЫВОД: Валидный JSON по контракту. Без markdown-обёртки. Без комментариев. Весь текст — на РУССКОМ языке.
```

## Пользовательский промпт (шаблон)

```
Кластеризуй семантическое ядро:
Проект: {{project.project_id}}
Ниша: {{project.niche}}
Кластер: {{project.niche_cluster}}
Гео: {{project.geo}}
Язык: {{project.language}}
Домен: {{project.domain}}
Приоритет ПС: {{project.priority_engines}}
Конкуренты: {{project.main_competitors | join(', ')}}

=== ПЕРСОНЫ (Шаг 00) ===
{{personas_summary}}

=== SEED QUERIES (Шаг 00 + Шаг 00.2, обогащённые) ===
{{enhanced_seed_queries_json}}

=== ДАННЫЕ WORDSTAT API ===
{{wordstat_data_json}}

=== ДАННЫЕ KEYSO/SERPSTAT ===
{{keyso_serpstat_data_json}}

=== SUGGEST (Яндекс + Google) ===
{{suggest_data_json}}

=== ТЕКУЩИЕ ПОЗИЦИИ (baseline из Шага 00.2) ===
{{baseline_summary_json}}

=== QUICK WINS (Шаг 00.2) ===
{{quick_wins_json}}

{{#if rag_context}}
=== КОНТЕКСТ ИЗ RAG ===
{{rag_context}}
{{/if}}
```

---

# ██ RAG-ОБОГАЩЕНИЕ

| Коллекция Qdrant   | Запрос                              | Что извлекаем                                         | Лимит токенов |
|---------------------|-------------------------------------|-------------------------------------------------------|--------------|
| `personas`          | project_id                          | Полные персоны с JTBD для маппинга кластеров          | 800          |
| `competitors`       | niche + geo                         | Семантические ядра конкурентов (если сохранены ранее)  | 600          |
| `niches`            | niche_cluster                       | Шаблоны кластеризации похожих проектов                | 400          |
| `audits`            | project_id + step_id=00.2           | Baseline позиций для маппинга текущих ключей          | 200          |

Общий лимит RAG-контекста: 2000 токенов.

---

# ██ СКРИПТЫ CLAUDE CODE

## Пред-шаг 1: Wordstat API (step_01_pre_wordstat.py)

```
# Назначение: Сбор частотности запросов из Яндекс.Wordstat
# Вход: seed_queries + enhanced_seed_queries из Шагов 00/00.2
# Выход: wordstat_data.json
# Действия:
# 1. Объединить и дедуплицировать seed_queries + enhanced_seed_queries
# 2. Для каждого seed-запроса: Wordstat API → базовая + фразовая частотность
# 3. Расширение: для топ-30 по частотности → запросить «похожие запросы»
# 4. Гео-фильтр: только регион project.geo (если commercial_local)
# 5. Лимит: до 1000 уникальных запросов
# 6. Rate limiting: 1 запрос/сек (лимит API Wordstat)
# 7. При ошибке API: retry 3x с бэкоффом, при неудаче → warning, продолжить
# 8. Сохранить в NocoDB таблицу raw_keywords (project_id, query, volume_wordstat, geo)
```

## Пред-шаг 2: KeySO / Serpstat расширение (step_01_pre_keyso.py)

```
# Назначение: Расширение СЯ через KeySO/Serpstat — ключи конкурентов
# Вход: project.main_competitors, project.domain, project.priority_engines
# Выход: keyso_serpstat_data.json
# Действия:
# 1. Для каждого конкурента: получить топ-500 ключей (se_id по priority_engines)
# 2. Для project.domain: получить ключи для пересечений
# 3. Вычислить: unique_to_competitor (ключи, которых нет у нас) — content gap
# 4. Пометить source: keyso|serpstat
# 5. Adaptive rate limiting: проверять X-RateLimit-Remaining header
# 6. При priority_engines=both: раздельные запросы Google RU + Yandex
# 7. Лимит: 500 ключей на конкурента (ecom/national), 200 для остальных
# 8. Дедупликация с Wordstat-данными
```

## Пред-шаг 3: Suggest API (step_01_pre_suggest.py)

```
# Назначение: Сбор подсказок из Яндекс Suggest и Google Suggest
# Вход: seed_queries (топ-50 по volume), project.geo, project.language
# Выход: suggest_data.json
# Действия:
# 1. Для каждого seed-запроса: Яндекс Suggest API + Google Suggest API
# 2. Пометить engine: yandex|google|both (если совпадает)
# 3. Пометить is_long_tail: true если слов ≥ 4
# 4. Дедупликация с Wordstat и KeySO данными
# 5. Лимит: до 500 уникальных suggest-запросов
# 6. Rate limiting: 0.5 сек между запросами
```

## Пред-шаг 4: Merge данных (step_01_pre_merge.py)

```
# Назначение: Объединение всех источников в единый массив для Opus
# Вход: wordstat_data.json, keyso_serpstat_data.json, suggest_data.json,
#        enhanced_seed_queries, quick_wins
# Выход: merged_queries.json (≤ 6000 токенов для Opus)
# Действия:
# 1. Объединить все источники, дедупликация по normalized query
# 2. Нормализация: lowercase, убрать лишние пробелы, ё→е для сравнения
# 3. Для каждого запроса: объединить данные из разных источников
#    (volume_yandex из Wordstat, volume_google из KeySO, position из GSC)
# 4. Приоритизация для truncation (бюджет 6000 токенов):
#    a) quick_wins (все, без обрезки)
#    b) seed_queries с position (все, без обрезки)
#    c) Запросы с volume ≥ 100/мес (до 300 штук)
#    d) Long-tail из suggest (до 200 штук)
#    e) Content gap из KeySO (до 100 штук)
# 5. Пометить source для каждого запроса
# 6. Вычислить статистику: total_raw, total_after_dedup, total_truncated
```

## Пост-шаг: Валидатор (step_01_post_validate.py)

```
# Назначение: Валидация output по JSON-схеме и бизнес-правилам
# Проверки (см. секцию QC CRITERIA):
# 1. JSON Schema валидация
# 2. clusters ≥ 15
# 3. Каждый кластер: engine_tag заполнен
# 4. Каждый кластер: mapped_personas не пуст
# 5. Каждый кластер: queries ≥ 2
# 6. items массив соответствует clusters (1:1)
# 7. total_queries ≥ 50
# 8. Все 4 интента представлены (informational, commercial, transactional, navigational)
# 9. coverage_by_persona: каждая persona_id из входа имеет ≥ 1 кластер
# 10. quick_wins из входа присутствуют в кластерах (≥ 80%)
# 11. Нет дублей cluster_id
# 12. При priority_engines=both: есть кластеры с [Я] И с [G] И с [ОБА]
# 13. Branded-кластер: priority ≤ medium
# 14. page_type_suggestion заполнен у всех кластеров
# 15. unclustered_queries имеет reason для каждого
```

## Пост-шаг: Сохранение (step_01_post_save.py)

```
# Назначение: Сохранение результатов в NocoDB + Qdrant
# Действия:
# 1. NocoDB: INSERT в step_results (project_id, step_id=01, result_json, qc_score, prompt_version)
#    При повторном прогоне: run_number++ (не перезаписывать)
# 2. Qdrant коллекция 'clusters':
#    Для каждого кластера: embedding = "{cluster_name} {intent} {top-3 queries}"
#    Payload: cluster_id, project_id, priority, persona_ids, volume
#    Нужен для Шагов 02, 04, 04.1 (каннибализация)
# 3. NocoDB таблица 'keywords':
#    Batch INSERT всех запросов (project_id, cluster_id, query, volume, engine, intent)
```

---

# ██ КРИТЕРИИ QC

| **Проверка**                                 | **Порог**                          | **Вес** | **При провале**    |
|----------------------------------------------|------------------------------------|---------|--------------------|
| Количество кластеров                         | ≥ 15                               | 10      | Retry              |
| Запросов всего                               | ≥ 50                               | 10      | Retry              |
| engine_tag заполнен                          | 100% кластеров                     | 10      | Retry              |
| mapped_personas не пуст                      | 100% кластеров                     | 10      | Retry              |
| Запросов в кластере                          | ≥ 2 у каждого                      | 8       | Retry              |
| 4 интента представлены                       | informational + commercial + transactional + ≥1 ещё | 8 | Retry |
| Покрытие персон                              | Каждая персона → ≥ 1 кластер       | 10      | Retry              |
| Quick wins покрытие                          | ≥ 80% quick_wins из входа в кластерах | 8    | Retry              |
| Уникальность cluster_id                     | Нет дублей                         | 5       | Retry              |
| [Я]/[G]/[ОБА] при both                      | Есть все 3 типа engine_tag         | 5       | Ревью              |
| Branded priority ≤ medium                    | Branded-кластер не critical/high   | 3       | Ревью              |
| page_type_suggestion заполнен               | 100% кластеров                     | 5       | Retry              |
| items ↔ clusters соответствие               | 1:1                                | 5       | Retry              |
| unclustered_queries с reason                | 100% имеют reason                  | 3       | Ревью              |

QC = сумма весов. ≥80 → Шаг 02, 60-79 → ревью, <60 → retry.

---

# ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1: Триггер (из Шага 00/00.2 или Manual)
  ↓
┌─────┴──────┬──────────────┐
Нода 2a      Нода 2b        Нода 2c
(Wordstat)   (KeySO/Serpst) (Suggest)
  ↓            ↓              ↓
└─────┬──────┴──────────────┘
  ↓
Нода 3: Merge + дедупликация (Code-нода)
  ↓
┌─────┴──────┐
Нода 4       Нода 4r
(Qdrant RAG) (Fetch Step00 + 00.2 из NocoDB)
  ↓            ↓
└─────┬──────┘
  ↓
Нода 5: Сборка промпта (Code-нода)
  ↓
Нода 6: Claude Opus API (HTTP Request)
  ↓
Нода 7: Парсинг JSON (Code-нода)
  ↓
Нода 8: Валидатор QC (Code-нода)
  ↓
Нода 9: IF → QC ≥ 80?
  ├── ДА → Нода 10a (NocoDB save) + Нода 10b (Qdrant save) [параллельно]
  │         ↓
  │         Нода 11: Grafana Metrics
  │         ↓
  │         Нода 12: Build Response → Шаг 02
  │
  └── НЕТ → Нода 9f: IF retry < 3?
              ├── ДА → Нода 9r: Prepare Retry → Нода 5 (loop)
              └── НЕТ → Нода 9t: Telegram Alert
```

## Обработка ошибок

- **Wordstat API 429/500**: бэкофф 5с→15с→45с, макс. 3 попытки. При неудаче → продолжить без Wordstat, лог warning.
- **KeySO/Serpstat недоступен**: продолжить без данных конкурентов, лог warning.
- **Suggest API**: при ошибке одного источника → продолжить с другим.
- **Opus 429/500**: бэкофф 10с→30с→90с, макс. 3 попытки.
- **QC провал 3x**: алерт Telegram с логом 3 попыток.
- **Qdrant недоступен**: сохранить только в NocoDB, лог предупреждения.
- **NocoDB save падает**: retry 2x, при провале → JSON в файл, алерт для ручного импорта.

---

# ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "01",
  "prompt_version": "step01_v1.0",
  "timestamp": "2025-01-15T10:00:00Z",
  "clusters": [
    {
      "cluster_id": "CL001",
      "cluster_name": "Детская стоматология — запись и выбор",
      "intent": "transactional",
      "hunt_level": 4,
      "mapped_personas": ["P001"],
      "priority": "critical",
      "engine_tag": "[ОБА]",
      "estimated_impact": 9,
      "effort": 3,
      "monthly_volume_total": 8200,
      "competition_level": "high",
      "queries": [
        {"query": "детская стоматология москва", "volume_yandex": 4500, "volume_google": 2100, "engine": "both", "intent": "transactional", "is_long_tail": false, "current_position": 18, "source": "seed", "ai_search_relevant": false},
        {"query": "детский стоматолог рядом со мной", "volume_yandex": 900, "volume_google": 400, "engine": "both", "intent": "transactional", "is_long_tail": true, "current_position": null, "source": "suggest", "ai_search_relevant": true},
        {"query": "записать ребёнка к стоматологу москва", "volume_yandex": 300, "volume_google": null, "engine": "yandex", "intent": "transactional", "is_long_tail": true, "current_position": null, "source": "wordstat", "ai_search_relevant": false}
      ],
      "page_type_suggestion": "landing",
      "niche_cluster_specific": "commercial_local: гео-модификаторы, карта, NAP-блок"
    },
    {
      "cluster_id": "CL002",
      "cluster_name": "Лечение кариеса у детей — информационный",
      "intent": "informational",
      "hunt_level": 2,
      "mapped_personas": ["P001"],
      "priority": "high",
      "engine_tag": "[ОБА]",
      "estimated_impact": 7,
      "effort": 5,
      "monthly_volume_total": 5400,
      "competition_level": "medium",
      "queries": [
        {"query": "кариес у ребёнка что делать", "volume_yandex": 2200, "volume_google": 1500, "engine": "both", "intent": "informational", "is_long_tail": true, "current_position": 35, "source": "wordstat", "ai_search_relevant": true},
        {"query": "молочные зубы лечить или удалять", "volume_yandex": 1100, "volume_google": 600, "engine": "both", "intent": "informational", "is_long_tail": true, "current_position": null, "source": "suggest", "ai_search_relevant": true}
      ],
      "page_type_suggestion": "article",
      "niche_cluster_specific": "medical: YMYL, указать квалификацию врачей"
    },
    {
      "cluster_id": "CL003",
      "cluster_name": "Цены на детскую стоматологию",
      "intent": "commercial",
      "hunt_level": 4,
      "mapped_personas": ["P001", "P002"],
      "priority": "high",
      "engine_tag": "[Я]",
      "estimated_impact": 8,
      "effort": 2,
      "monthly_volume_total": 3100,
      "competition_level": "medium",
      "queries": [
        {"query": "детская стоматология москва цены", "volume_yandex": 1800, "volume_google": null, "engine": "yandex", "intent": "commercial", "is_long_tail": false, "current_position": 12, "source": "gsc", "ai_search_relevant": false},
        {"query": "сколько стоит вылечить зуб ребёнку", "volume_yandex": 800, "volume_google": 500, "engine": "both", "intent": "commercial", "is_long_tail": true, "current_position": null, "source": "wordstat", "ai_search_relevant": true}
      ],
      "page_type_suggestion": "landing",
      "niche_cluster_specific": "commercial_local: прайс-лист, калькулятор стоимости"
    }
  ],
  "items": [
    {"id": "CL001", "title": "Детская стоматология — запись и выбор", "priority": "critical", "action_required": "Создать главный landing с геолокацией и формой записи", "engine_tag": "[ОБА]", "estimated_impact": 9, "effort": 3},
    {"id": "CL002", "title": "Лечение кариеса у детей — информационный", "priority": "high", "action_required": "Создать экспертную статью с E-E-A-T сигналами", "engine_tag": "[ОБА]", "estimated_impact": 7, "effort": 5},
    {"id": "CL003", "title": "Цены на детскую стоматологию", "priority": "high", "action_required": "Создать страницу прайса с микроразметкой", "engine_tag": "[Я]", "estimated_impact": 8, "effort": 2}
  ],
  "summary": {
    "total_clusters": 3,
    "total_queries": 7,
    "critical": 1,
    "high": 2,
    "medium": 0,
    "low": 0,
    "by_intent": {"informational": 1, "commercial": 1, "transactional": 1, "navigational": 0},
    "by_engine": {"yandex_only": 1, "google_only": 0, "both": 2},
    "coverage_by_persona": [
      {"persona_id": "P001", "clusters_count": 3, "queries_count": 7},
      {"persona_id": "P002", "clusters_count": 1, "queries_count": 2}
    ]
  },
  "unclustered_queries": [
    {"query": "стоматология для взрослых москва", "volume": 2000, "engine": "both", "reason": "Не соответствует ЦА проекта (детская стоматология)"}
  ],
  "data_sources_used": ["wordstat", "serpstat", "suggest", "gsc"],
  "qc_score": 88
}
```

---

# ██ СТРАТЕГИЯ ОТКАТА

| Попытка | Стратегия | Действия |
|---------|-----------|----------|
| Retry 1 | Инъекция ошибок QC | Вставить конкретные ошибки QC в промпт. Та же температура и стратегия. |
| Retry 2 | Декомпозиция | Разбить на 2 вызова: (1) кластеризация только transactional+commercial запросов, (2) informational+navigational. Объединить. |
| Retry 3 | Fallback с few-shot | Строгий промпт с полным few-shot примером. Сократить до 15-20 кластеров. Температура 0.1. |
| Retry 4 | Алерт человеку | Telegram/Email с логом 3 попыток. Ручная кластеризация или правка. |

**Откат B**: Если API Wordstat/KeySO недоступны — кластеризовать только на основе seed_queries + enhanced_seed_queries + suggest. Пометить в data_sources_used. QC снижает порог total_queries до ≥ 30.

**Откат C**: Если запросов < 50 после всех API — Opus генерирует запросы на основе персон + JTBD (помечает source: "manual"). Пометка в NocoDB для ручной верификации.

---

# ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер проекта       | Специфика кластеризации СЯ                                                                                     |
|------------------------|----------------------------------------------------------------------------------------------------------------|
| commercial_local       | Гео-модификаторы обязательны: «город», «район», «метро», «рядом». Отдельные кластеры для каждой услуги × гео. Карточные запросы (Яндекс.Карты, 2GIS, Google Maps). |
| commercial_national    | Федеральные запросы без гео. Кластеры по категориям товаров/услуг. SERP-фичи: featured snippets, PAA.         |
| ecom                   | Товарные кластеры: «купить X», «X цена», «X отзывы». Фильтровые: «X размер/цвет/бренд». Категорийные: «лучшие X 2025». |
| saas                   | vs-запросы: «X vs Y». Альтернативы: «альтернативы X». Интеграции: «X + Z интеграция». Pricing: «X pricing/тарифы». |
| info                   | Вопросные: «как», «почему», «что такое». Гайды: «пошаговая инструкция». Hub-кластеры для topical authority.   |
| medical                | Симптомные (уровень 1-2): «болит X», «причины Y». Лечебные (3-4): «лечение Z». YMYL: обязательно E-E-A-T.    |
| legal                  | Ситуативные: «что делать если», «как подать». Срочные: «адвокат срочно». Процедурные: «документы для».        |
| education              | Сезонные: «курсы X 2025», «подготовка к Y». Journey: «с чего начать Z».                                       |

---

# ██ МЕТРИКИ УСПЕХА

| Тип          | Метрика                                                | Как измерять                                                |
|--------------|--------------------------------------------------------|-------------------------------------------------------------|
| Опережающий  | Покрытие: каждая персона имеет ≥ 3 кластера?           | coverage_by_persona в summary                               |
| Опережающий  | Баланс интентов: все 4 типа представлены?              | by_intent в summary                                         |
| Опережающий  | Quick wins из 00.2 — вошли в кластеры?                 | Пересечение quick_wins → clusters.queries                   |
| Запаздывающий| После Шага 02: каждый кластер → URL в структуре сайта? | Ревью после Шага 02                                         |
| Запаздывающий| После Шага 04: нет каннибализации между кластерами?    | Cosine similarity в Qdrant (Шаг 04.1)                       |
| Контроль     | Ревью после Шага 02: кластеры отражают реальный спрос? | Ручная проверка sample кластеров vs Wordstat                |

---

# ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| Версия | Дата       | Изменения                                                                                              |
|--------|------------|--------------------------------------------------------------------------------------------------------|
| v1.0   | 2025-01    | Базовый сбор СЯ: seed → Wordstat → кластеризация                                                      |
| v2.0   | 2025-06    | + engine_tag, hunt_level, mapped_personas, page_type_suggestion, suggest API                            |
| v3.0   | 2025-12    | + RAG-обогащение, enhanced_seed_queries из 00.2, ai_search_relevant, адаптации под кластер, декомпозиция retry, unclustered_queries, coverage_by_persona |
