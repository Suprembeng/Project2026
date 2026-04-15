# STEP_06.md — E-E-A-T аудит (v1.0)

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4.5K токенов.

---

## ██ КАРТОЧКА ШАГА

| **Параметр** | **Значение** |
| --- | --- |
| ID | 06 |
| Название | Аудит E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) |
| Блок | 2: Структура и контент |
| Основная роль | A: SEO-стратег |
| Консультативные | E: CRO-аналитик, D: Аналитик конкур. разведки, F: Промпт-инженер |
| Тип | Промпт |
| HitL | Нет (результат → Шаги 08, 12, 17 автоматически) |
| Приоритет ROI | КРИТИЧЕСКИЙ для YMYL-ниш (medical, legal, finance); ВЫСОКИЙ для остальных |
| Зависит от | Шаг 05 (КФ-аудит — eeat_signals, category_scores, passed_checks), Шаг 04 (Структура — типы страниц), PROJECT.md |
| Передаёт в | Шаг 08 (ТЗ — требования к E-E-A-T блокам), Шаг 12 (SEO-текст — авторский стиль, экспертность), Шаг 17 (Финальный аудит — E-E-A-T скоринг), Шаг 27.2 (Карта экспертности) |
| Режим Opus | Аналитика/Стратегия: temperature 0.2, top_p 0.9, max_tokens 8192 |
| Prompt version | step06_v1.0 |

---

## ██ ЦЕЛЬ

Провести комплексный аудит четырёх компонентов E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) по всем типам страниц сайта с учётом специфики Яндекса ([Я]: ИКС, КФ, авторитетность через ПФ и ссылки) и Google ([G]: Quality Rater Guidelines, Schema Person/Organization, HCU, YMYL), оценить каждый компонент по балльной шкале, сопоставить с конкурентами, и сформировать приоритизированный план усиления E-E-A-T — для интеграции в ТЗ (Шаг 08), контент-стратегию (Шаг 12) и финальный аудит (Шаг 17).

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход формируется из output Шага 05 (КФ-аудит) + Шаг 04 (структура) + PROJECT.md + pre-script данные.

```json
{
  "project_id": "string",
  "domain": "string",
  "niche": "string",
  "niche_cluster": "string",
  "geo": "string",
  "priority_engines": "yandex|google|both",
  "cms": "string",
  "domain_age": "number|null",
  "main_competitors": ["string"],
  "step_05_output": {
    "overall_score": "number",
    "eeat_signals": {
      "expertise": ["string"],
      "authority": ["string"],
      "trust": ["string"],
      "experience": ["string"]
    },
    "category_scores": {
      "trust": "number",
      "legal": "number",
      "content": "number"
    },
    "issues": [
      {
        "id": "string",
        "category": "string",
        "severity": "string",
        "engine_tag": "string",
        "cross_step_hooks": { "to_step_06_eeat": "boolean" }
      }
    ]
  },
  "step_04_output": {
    "pages": [
      { "url": "string", "page_type": "string", "title": "string", "h1": "string" }
    ]
  },
  "eeat_crawl_data": {
    "author_pages": [
      {
        "url": "string",
        "author_name": "string|null",
        "has_bio": "boolean",
        "has_photo": "boolean",
        "has_credentials": "boolean",
        "has_social_links": "boolean",
        "schema_person": "boolean",
        "articles_count": "number"
      }
    ],
    "about_page": {
      "exists": "boolean",
      "url": "string|null",
      "has_history": "boolean",
      "has_team": "boolean",
      "has_mission": "boolean",
      "has_awards": "boolean",
      "schema_organization": "boolean"
    },
    "trust_signals": {
      "ssl": "boolean",
      "privacy_policy": "boolean",
      "terms_of_service": "boolean",
      "inn_ogrn_present": "boolean",
      "editorial_policy": "boolean",
      "fact_check_policy": "boolean",
      "correction_policy": "boolean",
      "contact_page_complete": "boolean"
    },
    "content_signals": {
      "avg_article_wordcount": "number|null",
      "articles_with_author": "number",
      "articles_total": "number",
      "articles_with_date": "number",
      "articles_with_sources": "number",
      "avg_update_frequency_days": "number|null"
    },
    "external_signals": {
      "backlink_domains_count": "number|null",
      "brand_mentions_count": "number|null",
      "google_knowledge_panel": "boolean|null",
      "wikipedia_page": "boolean|null",
      "yandex_iks": "number|null"
    },
    "competitor_eeat": [
      {
        "domain": "string",
        "has_author_pages": "boolean",
        "has_about_page": "boolean",
        "schema_person": "boolean",
        "schema_organization": "boolean",
        "editorial_policy": "boolean",
        "team_page": "boolean",
        "estimated_eeat_level": "low|medium|high"
      }
    ]
  }
}
```

**Обязательные:** `project_id`, `domain`, `niche_cluster`, `priority_engines`, `step_05_output.eeat_signals`.

**Опциональные:** `eeat_crawl_data` (при отсутствии — Opus оценивает на основе step_05 + RAG), `competitor_eeat`.

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 2: Аудит со скорингом**. Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "06",
  "prompt_version": "step06_v1.0",
  "timestamp": "ISO 8601",
  "overall_score": "0-100",
  "issues": [
    {
      "id": "string (EEAT001, EEAT002...)",
      "category": "experience|expertise|authoritativeness|trustworthiness",
      "severity": "critical|high|medium|low",
      "current_value": "string",
      "target_value": "string",
      "fix_description": "string (≤200 симв)",
      "estimated_impact": "1-10",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "affected_page_types": ["string"],
      "effort": "1-10",
      "effort_unit": "hours|days|weeks",
      "confidence_tag": "[✓]|[◆]|[◇]|[⚠]",
      "ymyl_critical": "boolean",
      "cross_step_hooks": {
        "to_step_08_tz": "boolean",
        "to_step_12_content": "boolean",
        "to_step_17_audit": "boolean",
        "to_step_27_2_expertise_map": "boolean"
      },
      "dependencies": ["EEAT_id"]
    }
  ],
  "passed_checks": [
    { "check": "string", "category": "string", "evidence": "string" }
  ],
  "component_scores": {
    "experience": "0-100",
    "expertise": "0-100",
    "authoritativeness": "0-100",
    "trustworthiness": "0-100"
  },
  "overall_score_breakdown": {
    "weights_used": { "experience": "num", "expertise": "num", "authoritativeness": "num", "trustworthiness": "num" },
    "weighted_sum": "num",
    "verification": "pass|fail"
  },
  "ymyl_assessment": {
    "is_ymyl": "boolean",
    "ymyl_topics": ["string"],
    "ymyl_risk_level": "critical|high|medium|low|none",
    "special_requirements": ["string"]
  },
  "author_strategy": {
    "recommended_author_pages": "number",
    "author_schema_required": "boolean",
    "author_bio_template": "string",
    "credentials_to_highlight": ["string"]
  },
  "org_authority_plan": {
    "schema_organization_fields": ["string"],
    "awards_certifications_to_add": ["string"],
    "media_mentions_strategy": "string|null",
    "knowledge_panel_strategy": "string|null"
  },
  "content_trust_plan": {
    "editorial_policy_required": "boolean",
    "fact_check_process": "string|null",
    "sources_citation_standard": "string",
    "update_policy": "string"
  },
  "benchmark_summary": {
    "benchmark_available": "boolean",
    "our_score": "0-100",
    "avg_competitor_score": "number|null",
    "leader": "string|null",
    "gap_components": ["string"]
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
- `overall_score` = взвешенная сумма `component_scores` (веса по `niche_cluster`, YMYL-ниши: Trust доминирует).
- Каждый `issues[].engine_tag` обязателен.
- `ymyl_assessment` — обязательно заполнять для ЛЮБОГО кластера (даже если is_ymyl=false).
- `author_strategy`, `org_authority_plan`, `content_trust_plan` — структурированные рекомендации для Шагов 08, 12, 27.2.
- `fix_description` ≤200 символов, без реальных ИНН/телефонов — только плейсхолдеры.

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

**Системный промпт**

```
Ты — Роль A: Senior SEO-стратег (10+ лет, Яндекс и Google), эксперт по E-E-A-T.
Консультанты: Роль E (CRO/поведение), Роль D (конкур. разведка), Роль F (промпт-инженер).
Prompt version: step06_v1.0

CoT ОБЯЗАТЕЛЕН для каждого issue:
[АНАЛИЗ]   Какой компонент E-E-A-T? Для какого ПС критичен? YMYL?
[РОЛИ]     A принимает, E/D консультируют.
[КОНТЕКСТ] niche_cluster, YMYL-статус, priority_engines, domain_age.
[КОНТРАКТ] JSON output по схеме.
[БЕНЧМАРК] Что у конкурентов? Quality Rater Guidelines стандарт.
[РЕШЕНИЕ]  current→target; severity; impact; effort.
[РИСКИ]    Последствия: HCU, Core Update, ИКС-падение.
[HitL]     Не требуется.

ПРАВИЛА:
1. Оцени 4 компонента: Experience, Expertise, Authoritativeness, Trustworthiness.
2. engine_tag ∈ {[Я],[G],[ОБА]}; при priority_engines=both → оба обязательно.
3. Маркировка: [✓]=verified данные; [◆]=обосновано; [◇]=гипотеза; [⚠]=риск.
4. Специфика [Я] (Яндекс E-E-A-T-аналоги):
   - ИКС (Индекс Качества Сайта) — зависит от экспертности + КФ + ПФ
   - Авторитетность через ссылочный профиль + возраст домена + КФ
   - Доверие: ИНН/ОГРН, лицензии, отзывы, Яндекс.Бизнес
   - ПФ (поведенческие факторы): вовлечённость = косвенный сигнал качества
   - Региональность: экспертность в конкретном регионе
5. Специфика [G] (Google E-E-A-T):
   - Quality Rater Guidelines (QRG) → Page Quality rating
   - Experience: первичный опыт автора (фото, кейсы, личные примеры)
   - Expertise: квалификация, образование, Schema Person + sameAs
   - Authoritativeness: упоминания в СМИ, Knowledge Panel, Wikipedia, цитирования
   - Trustworthiness: SSL, контакты, policies, accuracy, editorial process
   - HCU (Helpful Content Update): контент для людей, не для ботов
   - YMYL: Your Money or Your Life — повышенные требования к E-E-A-T
   - Schema Organization + Person + Article (author, datePublished, dateModified)
6. YMYL-ниши (medical, legal, finance): Trust ДОМИНИРУЕТ → weight 0.40+.
7. overall_score = Σ(component_scores[i] × weights[i]) — математически строго.
8. Веса по кластеру (единственный источник): {{cluster_eeat_weights}}
9. fix_description ≤200 символов. Без реальных ИНН/телефонов — плейсхолдеры [ИНН_КЛИЕНТА].
10. author_strategy: сколько авторских страниц нужно, шаблон био, credentials.
11. org_authority_plan: Schema Organization, упоминания, Knowledge Panel стратегия.
12. content_trust_plan: editorial policy, fact-checking, цитирование источников.
13. ymyl_assessment заполняется ВСЕГДА (даже для non-YMYL: is_ymyl=false).
14. Конкуренты с высоким E-E-A-T → фиксировать gap_components.
15. passed_checks.category НЕ пересекается с issues.category для того же аспекта.

АНТИПАТТЕРНЫ:
✗ Общие рекомендации без конкретики («улучшить E-E-A-T»)
✗ Без engine_tag
✗ Пропуск хоть одного из 4 компонентов
✗ Реальные ИНН/ОГРН/телефоны
✗ overall_score ≠ Σ(weights × component_scores)
✗ YMYL-ниша без ymyl_assessment.is_ymyl=true
✗ Дубли issues (одна проблема — один id)

ВЫВОД: Валидный JSON по контракту. Без markdown. На русском.
```

**Пользовательский промпт (шаблон)**

```
Проведи аудит E-E-A-T.

Проект: {{project.project_id}} | Домен: {{project.domain}}
Ниша: {{project.niche}} | Кластер: {{project.niche_cluster}}
Гео: {{project.geo}} | ПС: {{project.priority_engines}} | Возраст домена: {{project.domain_age}}

ВЕСА E-E-A-T (единственный источник):
{{cluster_eeat_weights | json}}

СИГНАЛЫ E-E-A-T ИЗ ШАГА 05 (КФ-аудит):
{{step_05_output.eeat_signals | json}}

КФ-СКОРИНГ (trust, legal, content):
trust={{step_05_output.category_scores.trust}}, legal={{step_05_output.category_scores.legal}}, content={{step_05_output.category_scores.content}}

ISSUES ИЗ ШАГА 05 С ХУКОМ to_step_06_eeat=true:
{{step_05_eeat_issues | json}}

ТИПЫ СТРАНИЦ (Шаг 04):
{{step_04_output.pages | json}}

{{#if eeat_crawl_data}}
ДАННЫЕ КРАУЛИНГА E-E-A-T:
Авторские страницы: {{eeat_crawl_data.author_pages | json}}
О компании: {{eeat_crawl_data.about_page | json}}
Сигналы доверия: {{eeat_crawl_data.trust_signals | json}}
Контентные сигналы: {{eeat_crawl_data.content_signals | json}}
Внешние сигналы: {{eeat_crawl_data.external_signals | json}}
{{/if}}

{{#if eeat_crawl_data.competitor_eeat}}
КОНКУРЕНТЫ E-E-A-T:
{{eeat_crawl_data.competitor_eeat | json}}
{{/if}}

{{#if rag_context}}
RAG КОНТЕКСТ:
{{rag_context}}
{{/if}}

Prompt version: step06_v1.0
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| **Коллекция Qdrant** | **Запрос** | **Назначение** |
| --- | --- | --- |
| `competitors` | `niche_cluster` + `geo` + «E-E-A-T» | Бенчмарк E-E-A-T конкурентов |
| `niches` | `niche_cluster` + «Quality Rater Guidelines E-E-A-T» | Стандарты QRG для кластера |
| `personas` | `project_id` | Ожидания ЦА по экспертности и доверию |
| `cf_audit` | `project_id` + step_id=05 | Результаты КФ-аудита (trust/legal/content) |
| `site_structure` | `project_id` | Текущие страницы для маппинга рекомендаций |

Лимит RAG-контекста: 2000 токенов.

---

## ██ СКРИПТЫ CLAUDE CODE

> **Тип шага — Промпт.** Основная логика — в промпте Opus. Скрипты: только pre-step сбор данных и post-step валидация.

### Пред-шаг: Краулер E-E-A-T сигналов

```
# step_06_pre_eeat_crawler.py
# Назначение: Сбор E-E-A-T сигналов со страниц сайта
# Вход: project.domain, step_04_output.pages[]
# Выход: eeat_crawl_data.json
# Действия:
# 1. АВТОРСКИЕ СТРАНИЦЫ:
#    - Поиск URL-паттернов: /author/, /avtor/, /team/, /vrachi/, /specialists/
#    - Для каждой: has_bio (>100 слов), has_photo (<img> в блоке автора),
#      has_credentials (образование/опыт/диплом), has_social_links (ссылки на соцсети),
#      schema_person (JSON-LD Person), articles_count (связанные статьи)
# 2. СТРАНИЦА «О КОМПАНИИ»:
#    - /about/, /o-kompanii/, /o-nas/ → has_history, has_team, has_mission,
#      has_awards (грамоты/награды), schema_organization
# 3. СИГНАЛЫ ДОВЕРИЯ:
#    - SSL, /privacy/, /terms/, /oferta/, ИНН/ОГРН (с валидацией КС),
#      /editorial-policy/, /fact-check/, /corrections/, contact_page_complete
# 4. КОНТЕНТНЫЕ СИГНАЛЫ:
#    - Выборка 10 статей: средняя длина, наличие автора (byline),
#      дата публикации/обновления, ссылки на источники
# 5. ВНЕШНИЕ СИГНАЛЫ (API):
#    - Ahrefs/Majestic API: backlink_domains_count
#    - Brand mention check (web_search)
#    - Google Knowledge Panel: web_search "{brand}"
#    - Wikipedia: API check
#    - ИКС: Яндекс.Вебмастер API (если доступен)
# 6. КОНКУРЕНТЫ:
#    - Для каждого конкурента (макс. 3): та же проверка (сокращённо)
#    - estimated_eeat_level: low (<30%), medium (30-60%), high (>60%) по факторам
#
# Ограничения: robots.txt, delay 300мс, timeout 30с/URL
# Fallback: API недоступен → null, продолжить
```

### Пост-шаг: Валидатор

```
# step_06_post_validate.py
# Назначение: Валидация выхода по JSON-схеме и QC
# Проверки:
# 1. JSON Schema — Template 2 + расширения E-E-A-T [hard]
# 2. Все issues.id уникальны, формат EEAT### [hard]
# 3. engine_tag ∈ {[Я],[G],[ОБА]} 100% [hard]
# 4. Все 4 компонента в component_scores [hard]
# 5. component_scores[i] ∈ [0,100] [hard]
# 6. overall_score ≈ Σ(component_scores[i] × weights[i]) ±5 [hard]
# 7. YMYL-ниша → ymyl_assessment.is_ymyl=true [hard]
# 8. issues ≥ 3 (минимум) [hard]
# 9. fix_description ≤200 символов [hard]
# 10. Нет реальных ИНН/ОГРН/телефонов (regex blacklist) [hard]
# 11. priority_engines=both → [Я] И [G] есть [hard]
# 12. passed_checks.category ∩ issues.category = ∅ (для аспекта) [hard]
# 13. Нет дублей id [hard]
# 14. author_strategy заполнен при expertise<60 [soft]
# 15. org_authority_plan заполнен при authoritativeness<60 [soft]
# 16. content_trust_plan заполнен при trustworthiness<60 [soft]
# 17. cross_step_hooks: to_step_08_tz=true у critical/high [soft]
# 18. ymyl_assessment заполнен для ВСЕХ кластеров [hard]
# 19. benchmark_summary согласован [soft]
# 20. confidence_tag=[✓] запрещено при benchmark_available=false [soft]
```

### Пост-шаг: NocoDB + Qdrant

```
# step_06_post_store.py
# 1. Idempotency: sha256(project_id + step_id + timestamp)
# 2. NocoDB INSERT eeat_audit (project_id, step_id, overall_score, component_scores, output_json)
# 3. NocoDB UPDATE step_results (step_id=06, status=done, qc_score, prompt_version)
# 4. NocoDB INSERT audit_log (prompt_version, model_version)
# 5. Qdrant upsert 'eeat_audit': embedding(category + current + target + fix)
# 6. Event emitter → Шаги 08, 12, 17, 27.2 по cross_step_hooks
# 7. Grafana push: component_scores, overall_score, issues_count
```

---

## ██ КРИТЕРИИ QC

| **Проверка** | **Порог** | **Вес** | **Тип** | **При провале** |
| --- | --- | --- | --- | --- |
| JSON Schema валидна | Pass | 10 | hard | Retry |
| Все 4 компонента в component_scores | 4/4 | 10 | hard | Retry |
| Engine теги 100% | Pass | 10 | hard | Retry |
| overall_score = Σ(comp×weight) ±5 | Pass | 15 | hard | Retry |
| YMYL-ниша → is_ymyl=true | Pass | 10 | hard | Retry |
| ymyl_assessment заполнен | Pass | 5 | hard | Retry |
| Issues ≥ 3 | Pass | 5 | hard | Retry |
| fix_description ≤200 | 100% | 3 | hard | Retry |
| Нет реальных реквизитов | Pass | 10 | hard | Retry |
| both → [Я] И [G] | Pass | 5 | hard | Retry |
| passed_checks ∩ issues = ∅ | Pass | 5 | hard | Retry |
| Нет дублей id | Pass | 2 | hard | Retry |
| author_strategy при expertise<60 | Pass | 3 | soft | Ревью |
| content_trust_plan при trust<60 | Pass | 3 | soft | Ревью |
| cross_step_hooks согласованы | Pass | 2 | soft | Ревью |
| confidence_tag согласован | Pass | 2 | soft | Ревью |

QC = сумма весов. ≥80 + все hard → Шаг 08; 60-79 → ревью; <60 → retry.

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (после Шага 05 или ручной запуск)
Нода 2:  PROJECT.md + output Шагов 04, 05 из NocoDB
Нода 3:  step_06_pre_eeat_crawler.py → eeat_crawl_data.json
Нода 4:  Qdrant RAG (5 коллекций: competitors, niches, personas, cf_audit, site_structure)
Нода 5:  Инжект cluster_eeat_weights + prompt_version
Нода 6:  Сборка промпта (system + user с переменными + RAG + step_05 eeat_signals)
Нода 7:  Claude Opus API (temperature 0.2, top_p 0.9, max_tokens 8192)
Нода 8:  Парсинг JSON (strip markdown-обёртки)
Нода 9:  step_06_post_validate.py
         ├─ hard-fail → Нода 6 (retry, макс. 3, инъекция ошибок QC)
         └─ hard-fail 2x → декомпозиция на 2 вызова (Experience+Expertise | Authority+Trust)
Нода 10: step_06_post_store.py → NocoDB + Qdrant (idempotent)
Нода 11: Grafana metrics push (component_scores, overall_score, ymyl_risk)
Нода 12: Event emitter → Шаги 08, 12, 17, 27.2 по cross_step_hooks
Нода 13: Передача → Шаг 08 (ТЗ) или стоп
```

**Обработка ошибок:**

```
• Краулер E-E-A-T сигналов таймаут: partial data, confidence ≤ [◆]
• API внешних сигналов (Ahrefs/Majestic) недоступен: external_signals = null, fallback на web_search
• Opus 429/500: бэкофф 10с→30с→90с, макс. 3
• QC hard-fail 2x: декомпозиция (Experience+Expertise | Authority+Trust) → merge
• QC hard-fail 3x: алерт Telegram с логом
• Qdrant недоступен: без RAG, лог предупреждения
• Невалидный JSON от Opus: strip ```json```, повторный парсинг → retry
```

---

## ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "06",
  "prompt_version": "step06_v1.0",
  "timestamp": "2025-12-22T11:00:00Z",
  "overall_score": 38,
  "issues": [
    {
      "id": "EEAT001",
      "category": "expertise",
      "severity": "critical",
      "current_value": "Нет авторских страниц врачей (0 из 8 врачей имеют профили на сайте)",
      "target_value": "Профиль каждого врача: фото, bio ≥200 слов, образование, стаж, специализация, Schema Person",
      "fix_description": "Создать /vrachi/{name}/ для каждого врача. Шаблон: фото, bio, образование, стаж, отзывы, Schema Person.",
      "estimated_impact": 10,
      "engine_tag": "[ОБА]",
      "affected_page_types": ["service", "article", "about"],
      "effort": 5,
      "effort_unit": "days",
      "confidence_tag": "[✓]",
      "ymyl_critical": true,
      "cross_step_hooks": {
        "to_step_08_tz": true,
        "to_step_12_content": true,
        "to_step_17_audit": true,
        "to_step_27_2_expertise_map": true
      },
      "dependencies": []
    },
    {
      "id": "EEAT002",
      "category": "trustworthiness",
      "severity": "critical",
      "current_value": "Отсутствуют лицензии и сертификаты на сайте (YMYL-ниша medical)",
      "target_value": "Страница /licenzii/ со сканами лицензий, Schema hasCredential, ссылки из footer",
      "fix_description": "Создать /licenzii/ со сканами. Добавить Schema MedicalOrganization.hasCredential. Ссылку в footer.",
      "estimated_impact": 10,
      "engine_tag": "[ОБА]",
      "affected_page_types": ["main", "about"],
      "effort": 3,
      "effort_unit": "days",
      "confidence_tag": "[✓]",
      "ymyl_critical": true,
      "cross_step_hooks": {
        "to_step_08_tz": true,
        "to_step_12_content": false,
        "to_step_17_audit": true,
        "to_step_27_2_expertise_map": false
      },
      "dependencies": []
    },
    {
      "id": "EEAT003",
      "category": "authoritativeness",
      "severity": "high",
      "current_value": "Нет Schema Organization + Person на сайте",
      "target_value": "JSON-LD Organization (главная) + Person (врачи) + sameAs (соцсети, профреестры)",
      "fix_description": "Внедрить JSON-LD: Organization на главной, Person на /vrachi/*, sameAs на профили врачей в NMO.",
      "estimated_impact": 7,
      "engine_tag": "[G]",
      "affected_page_types": ["main", "about", "doc"],
      "effort": 3,
      "effort_unit": "days",
      "confidence_tag": "[◆]",
      "ymyl_critical": false,
      "cross_step_hooks": {
        "to_step_08_tz": true,
        "to_step_12_content": false,
        "to_step_17_audit": true,
        "to_step_27_2_expertise_map": true
      },
      "dependencies": ["EEAT001"]
    },
    {
      "id": "EEAT004",
      "category": "experience",
      "severity": "high",
      "current_value": "Статьи блога без авторства (0/15 с byline), без дат обновления",
      "target_value": "Каждая статья: автор-врач (byline + Schema author), dateModified, 'опыт из практики'",
      "fix_description": "Добавить byline врача в каждую статью. Добавить dateModified. Включить секцию 'Из практики'.",
      "estimated_impact": 8,
      "engine_tag": "[G]",
      "affected_page_types": ["article"],
      "effort": 4,
      "effort_unit": "days",
      "confidence_tag": "[◆]",
      "ymyl_critical": true,
      "cross_step_hooks": {
        "to_step_08_tz": true,
        "to_step_12_content": true,
        "to_step_17_audit": true,
        "to_step_27_2_expertise_map": true
      },
      "dependencies": ["EEAT001"]
    },
    {
      "id": "EEAT005",
      "category": "trustworthiness",
      "severity": "medium",
      "current_value": "Нет editorial policy и correction policy",
      "target_value": "Страницы /editorial-policy/ и /corrections/ с описанием процесса рецензирования",
      "fix_description": "Создать /editorial-policy/ с описанием: кто пишет, рецензирует, обновляет. Ссылку в footer.",
      "estimated_impact": 5,
      "engine_tag": "[G]",
      "affected_page_types": ["article", "main"],
      "effort": 2,
      "effort_unit": "days",
      "confidence_tag": "[◆]",
      "ymyl_critical": true,
      "cross_step_hooks": {
        "to_step_08_tz": true,
        "to_step_12_content": true,
        "to_step_17_audit": true,
        "to_step_27_2_expertise_map": false
      },
      "dependencies": []
    },
    {
      "id": "EEAT006",
      "category": "authoritativeness",
      "severity": "medium",
      "current_value": "ИКС = 20 (низкий для медицинской ниши, конкуренты 50-200)",
      "target_value": "ИКС ≥ 80 через улучшение КФ + E-E-A-T + ПФ",
      "fix_description": "Комплекс: лицензии, авторы, КФ (Шаг 05), ПФ (Шаг 25). ИКС — следствие, не причина.",
      "estimated_impact": 6,
      "engine_tag": "[Я]",
      "affected_page_types": ["all"],
      "effort": 8,
      "effort_unit": "weeks",
      "confidence_tag": "[◇]",
      "ymyl_critical": false,
      "cross_step_hooks": {
        "to_step_08_tz": false,
        "to_step_12_content": false,
        "to_step_17_audit": true,
        "to_step_27_2_expertise_map": false
      },
      "dependencies": ["EEAT001", "EEAT002", "EEAT003"]
    }
  ],
  "passed_checks": [
    { "check": "SSL-сертификат активен (HTTPS)", "category": "trustworthiness", "evidence": "https://dental-moscow.ru" },
    { "check": "Страница контактов полная (адрес + карта + телефон)", "category": "trustworthiness", "evidence": "/contacts/" },
    { "check": "Политика конфиденциальности существует", "category": "trustworthiness", "evidence": "/privacy-policy/" }
  ],
  "component_scores": {
    "experience": 20,
    "expertise": 25,
    "authoritativeness": 30,
    "trustworthiness": 45
  },
  "overall_score_breakdown": {
    "weights_used": { "experience": 0.15, "expertise": 0.25, "authoritativeness": 0.20, "trustworthiness": 0.40 },
    "weighted_sum": 32.25,
    "verification": "pass"
  },
  "ymyl_assessment": {
    "is_ymyl": true,
    "ymyl_topics": ["health", "medical_treatment", "medication"],
    "ymyl_risk_level": "critical",
    "special_requirements": [
      "Все медицинские статьи должны быть написаны/рецензированы врачом",
      "Лицензии обязательны в footer и на отдельной странице",
      "Запрет на медицинские советы без дисклеймера",
      "Schema MedicalOrganization + Physician обязательны"
    ]
  },
  "author_strategy": {
    "recommended_author_pages": 8,
    "author_schema_required": true,
    "author_bio_template": "Имя, фото, специализация, образование (ВУЗ+год), стаж (лет), сертификаты, публикации, отзывы пациентов",
    "credentials_to_highlight": ["Медицинское образование", "Действующая лицензия", "Стаж ≥5 лет", "Членство в профессиональных ассоциациях"]
  },
  "org_authority_plan": {
    "schema_organization_fields": ["name", "legalName", "taxID", "address", "telephone", "url", "logo", "sameAs", "hasCredential", "numberOfEmployees", "foundingDate"],
    "awards_certifications_to_add": ["Лицензия Минздрава", "Сертификат ISO", "Награды профессиональных конкурсов"],
    "media_mentions_strategy": "PR-публикации в med-порталах (prodoctorov.ru, doctorpiter.ru) + экспертные комментарии в СМИ",
    "knowledge_panel_strategy": "Верификация Google Business Profile + Schema Organization + Wikipedia (при достаточной notability)"
  },
  "content_trust_plan": {
    "editorial_policy_required": true,
    "fact_check_process": "Каждая статья рецензируется профильным врачом (byline рецензента). Источники — PubMed, клинические рекомендации МЗ РФ.",
    "sources_citation_standard": "Минимум 3 источника на статью. Формат: [Автор, Название, Журнал, Год]. Ссылки на PubMed/eLibrary.",
    "update_policy": "Статьи обновляются не реже 1 раза в 12 месяцев. dateModified обязателен. Устаревшие → пометка + обновление."
  },
  "benchmark_summary": {
    "benchmark_available": true,
    "our_score": 38,
    "avg_competitor_score": 65,
    "leader": "smile-clinic.ru",
    "gap_components": ["expertise", "experience", "authoritativeness"]
  },
  "priority_engines_split": {
    "yandex_specific_issues": 1,
    "google_specific_issues": 3,
    "both_issues": 2
  },
  "qc_score": 91
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

```
Откат A: Краулер E-E-A-T не завершился.
  → Opus оценивает на основе step_05_output.eeat_signals + RAG.
  → eeat_crawl_data = null, confidence ≤ [◆].

Откат B: Конкуренты недоступны.
  → benchmark_available = false, competitor_benchmark = null.
  → Opus оценивает по абсолютным стандартам QRG (из RAG).
  → [✓] запрещено.

Откат C: Opus hard-QC fail 2x.
  → Декомпозиция на 2 вызова:
    В1: Experience + Expertise (компоненты, связанные с авторами/контентом)
    В2: Authoritativeness + Trustworthiness (компоненты, связанные с организацией/доверием)
  → Merge + пересчёт overall_score.

Откат D: Qdrant недоступен.
  → Без RAG. Confidence ≤ [◆].
  → Opus оценивает на основе crawl_data + step_05_output.

Откат E: API внешних сигналов (Ahrefs/Majestic) недоступен.
  → external_signals заполнены частично/null.
  → authoritativeness score основан на on-site сигналах.
  → Пометка: partial_external_data = true.
```

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

```
Веса E-E-A-T по кластерам (единственный источник):

cluster_eeat_weights = {
  commercial_local: {experience:0.15, expertise:0.20, authoritativeness:0.25, trustworthiness:0.40},
  commercial_national: {experience:0.15, expertise:0.25, authoritativeness:0.30, trustworthiness:0.30},
  ecom: {experience:0.20, expertise:0.15, authoritativeness:0.25, trustworthiness:0.40},
  saas: {experience:0.20, expertise:0.30, authoritativeness:0.25, trustworthiness:0.25},
  info: {experience:0.25, expertise:0.30, authoritativeness:0.25, trustworthiness:0.20},
  medical: {experience:0.15, expertise:0.25, authoritativeness:0.20, trustworthiness:0.40},
  legal: {experience:0.15, expertise:0.25, authoritativeness:0.20, trustworthiness:0.40},
  education: {experience:0.25, expertise:0.25, authoritativeness:0.20, trustworthiness:0.30}
}

Специфика:

• commercial_local:
  — Trust доминирует: ИНН/ОГРН, лицензии, Яндекс.Бизнес (NAP)
  — Expertise: профили сотрудников/мастеров с фото и стажем
  — Experience: отзывы с фото, кейсы «до/после»
  — Authority: упоминания в локальных СМИ, рейтинги на картах

• commercial_national:
  — Authority доминирует: ссылки, упоминания, лидерство в нише
  — Expertise: кейсы, портфолио, цифры (N клиентов, M лет)
  — Trust: реквизиты, сертификаты ISO, партнёры

• ecom:
  — Trust критичен: SSL, безопасные оплаты, return policy, Schema Product
  — Experience: отзывы покупателей с фото, видеообзоры товаров
  — Authority: бренд на маркетплейсах, упоминания блогерами

• saas:
  — Expertise доминирует: техническая документация, API, кейсы
  — Experience: demo/trial, changelog, community отзывы
  — Authority: интеграции, партнёры, tech-blog ссылки

• info:
  — Expertise+Experience: авторская экспертиза, byline, bio, первичный опыт
  — Authority: цитирования, гостевые посты, конференции
  — Trust: editorial policy, fact-checking, sources

• medical (YMYL):
  — Trust КРИТИЧЕН (0.40): лицензии Минздрава, профили врачей с NMO
  — Expertise: специализация, образование, публикации в PubMed
  — Experience: кейсы лечения (с согласия), фото «до/после»
  — Schema: MedicalOrganization, Physician, MedicalCondition
  — ОБЯЗАТЕЛЬНО: disclamer «Не является медицинской консультацией»

• legal (YMYL):
  — Trust КРИТИЧЕН (0.40): статус адвоката, реестр, лицензия
  — Expertise: специализации, кейсы (анонимизированные), публикации
  — Experience: стаж, количество дел, результаты
  — Schema: LegalService, Attorney (custom)

• education:
  — Experience: отзывы выпускников, фото с обучения, результаты
  — Expertise: преподаватели с credential, программы аккредитованы
  — Trust: лицензия на образовательную деятельность, диплом гос. образца
```

---

## ██ МЕТРИКИ УСПЕХА

```
Опережающий:
  — overall_score рассчитан, verification=pass
  — Все 4 компонента покрыты
  — YMYL-assessment заполнен корректно
  — author_strategy, org_authority_plan, content_trust_plan заполнены при низких scores
  — Issues ≥ 3 с конкретными fix_description
  — Engine split: [Я] И [G] при priority_engines=both

Запаздывающий:
  — После Шага 08: ≥80% critical/high issues включены в ТЗ
  — После Шага 12: авторские byline в ≥80% новых статей
  — После Шага 17: component_scores +15 пунктов каждый
  — Через 90 дней: HCU не затронул сайт (проверка GSC impressions)
  — Через 90 дней: рост ИКС ≥ 20% (для Яндекса)
  — Через 120 дней: появление Knowledge Panel (для Google)

Контроль:
  — Ревью: author_bio_template реалистичен для ниши (≥80% заполняемость)
  — Бенчмарк: 3 issues проверены через web_search — ≥80% верны
  — Стабильность: 2 запуска дают ±5 пунктов overall_score
  — Token budget: ≤9K/вызов
```

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2025-12 | Базовая версия: 4 компонента E-E-A-T, Template 2, кластерные веса, YMYL-assessment, author_strategy, org_authority_plan, content_trust_plan, Schema Person/Organization, QRG-стандарты, cross_step_hooks для 08/12/17/27.2 |
