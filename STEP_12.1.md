# STEP_12.1.md — Image SEO (Оптимизация изображений)

> **Модуль 3 · Спецификация шага** · Загружается вместе с `CORE.md` + `PROJECT.md`.
> Бюджет токенов: ~4K. Версия: `step12_1_v1.0`.

---

## ██ 1. КАРТОЧКА ШАГА (STEP CARD)

| **Параметр** | **Значение** |
| --- | --- |
| ID | `12.1` |
| Название | Image SEO — аудит и оптимизация изображений |
| Блок | 3: Контент и оптимизация (09–13) |
| Основная роль | **B: Инженер автоматизации** |
| Консультативные роли | A: SEO-стратег (приоритеты, engine-split), D: Аналитик конкурентной разведки (бенчмарки), F: Промпт-инженер (fallback-генерация alt), G: Инженер контрактов данных (валидация JSON) |
| Тип | **Скрипт** (Opus используется только в fallback для генерации alt) |
| HitL | Нет (результат автоматически → Шаг 13) |
| Приоритет ROI | ВЫСОКИЙ — влияет на CWV (LCP/CLS), ранжирование в Яндекс.Картинках и Google Images, доступность |
| Зависит от | Шаг 04 (Структура страниц) — карта URL; Шаг 12 (SEO-текст) — keywords/ключи по страницам; `PROJECT.md` |
| Передаёт в | Шаг 13 (Schema.org + OG — `ImageObject`), Шаг 14 (Sitemap — image-sitemap.xml), Шаг 16 (Скорость — рекомендации по WebP/lazy), Шаг 17 (Финальный аудит) |
| Режим Opus | **Не основной.** При необходимости fallback: JSON-вывод, `temperature=0.15`, `top_p=0.9`, `max_tokens=4096` |
| JSON Template | **Шаблон 4: Script/Config** (с расширением `engine_tag` на каждый action) |
| Prompt Version | `step12_1_v1.0` |

---

## ██ 2. ЦЕЛЬ (GOAL)

**Одно предложение:** Провести технический аудит всех изображений сайта и сгенерировать детерминированный набор действий по оптимизации (alt, filename, формат, размер, lazy loading, `width`/`height`, srcset) для роста ранжирования в Яндекс.Картинках/Google Images, улучшения Core Web Vitals и доступности — без обращения к LLM на горячем пути.

---

## ██ 3. КОНТРАКТ ВХОДНЫХ ДАННЫХ (INPUT CONTRACT)

Вход — объединение выхода Шага 04 (структура страниц) и Шага 12 (SEO-тексты с ключами). Плюс сырые данные с сайта (собираются pre-скриптом).

```json
{
  "project_id": "string",
  "step_id": "12.1",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "priority_engines": "yandex|google|both",
  "niche_cluster": "string",
  "geo": "string",
  "language": "string",
  "pages": [
    {
      "page_url": "string (абсолютный URL)",
      "page_type": "landing|category|article|faq|product",
      "primary_keyword": "string",
      "secondary_keywords": ["string"],
      "persona_id": "string (P001…)",
      "hunt_level": "number 1-5"
    }
  ],
  "crawl_config": {
    "max_images_per_page": "number (default 50)",
    "user_agent": "string",
    "respect_robots_txt": "boolean (default true)",
    "timeout_sec": "number (default 15)"
  },
  "thresholds": {
    "max_file_size_kb": "number (default 200)",
    "max_dimension_px": "number (default 1920)",
    "min_alt_length": "number (default 5)",
    "max_alt_length": "number (default 125)"
  }
}
```

**Источники полей:**
- `pages[]` ← Шаг 04 (`structure.pages`) + Шаг 12 (`page_url`, `keywords`)
- `priority_engines`, `niche_cluster`, `geo`, `language` ← `PROJECT.md`
- `thresholds` — дефолтные, можно переопределить

---

## ██ 4. КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (OUTPUT CONTRACT)

Наследует **Шаблон 4 (Script/Config)** с расширением `engine_tag` на каждый action (требование CORE — маркировка при `priority_engines=both`). Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "12.1",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "target_system": "cms|frontend|cdn|storage",
  "summary": {
    "pages_scanned": "number",
    "images_total": "number",
    "images_with_issues": "number",
    "critical_issues": "number",
    "avg_page_weight_before_kb": "number",
    "avg_page_weight_after_kb": "number (прогноз)",
    "estimated_lcp_improvement_ms": "number"
  },
  "actions": [
    {
      "action_id": "string (IMG_001…)",
      "type": "create|update|delete|configure",
      "target": "string (URL страницы + CSS-селектор изображения)",
      "image_src": "string (абсолютный URL изображения)",
      "issue_category": "alt_missing|alt_low_quality|filename_non_seo|oversized|wrong_format|no_lazy|no_dimensions|no_srcset|duplicate|decorative_missing_role",
      "severity": "critical|high|medium|low",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "confidence": "✓ПОДТВЕРЖДЕНО|◆ОБОСНОВАНО|◇ГИПОТЕЗА|⚠РИСК",
      "current_value": {
        "alt": "string|null",
        "filename": "string",
        "format": "jpg|jpeg|png|gif|webp|avif|svg",
        "file_size_kb": "number",
        "width_px": "number|null",
        "height_px": "number|null",
        "has_lazy": "boolean",
        "has_dimensions_attr": "boolean",
        "has_srcset": "boolean"
      },
      "recommendation": {
        "alt": "string|null",
        "filename": "string",
        "format": "webp|avif|jpeg|png",
        "compression_quality": "number 60-90",
        "target_dimensions": {"width": "number", "height": "number"},
        "loading": "lazy|eager",
        "fetchpriority": "high|low|auto",
        "srcset": ["string"],
        "sizes_attr": "string"
      },
      "rationale": "string (почему это важно, со ссылкой на правило/движок)",
      "estimated_impact": "number 1-10",
      "effort": "number 1-10",
      "validation_result": "pass|fail|skip"
    }
  ],
  "execution_log": ["string"],
  "image_sitemap_ready": "boolean",
  "qc_score": "number 0-100"
}
```

**Ключевое отличие от чистого Шаблона 4:** добавлены `summary`, `engine_tag` + `confidence` на каждое действие, вложенный `current_value`/`recommendation` вместо плоского `config`. Это не нарушает шаблон, а расширяет (обратная совместимость сохранена — поля-основа Шаблона 4 присутствуют).

---

## ██ 5. ПРОМПТ ДЛЯ CLAUDE OPUS (минимальный — только fallback)

> Для шага типа **Скрипт** Opus на горячем пути **не вызывается**. Промпт используется только в fallback-режиме для генерации alt-текста, когда скриптовые эвристики не смогли построить качественное описание (например, у изображения нет окружающего текста, нет `title`, нет имени файла с ключами).

### 5.1. Системный промпт (fallback)

```
Ты — Роль F: Senior Промпт-инженер в связке с Ролью A (SEO-стратег).
Задача: Сгенерировать alt-текст для изображения, у которого скриптовые эвристики
не смогли собрать качественное описание.

ПРАВИЛА ALT:
1. Длина: 5–125 символов (оптимум 40–90).
2. Естественный русский язык, описание того, ЧТО на изображении.
3. Мягкая интеграция primary_keyword, если это органично. Запрещён keyword stuffing.
4. Без фраз-паразитов: «изображение», «картинка», «фото» (кроме случаев, когда тип изображения семантически важен).
5. Учитывай niche_cluster:
   - medical: строго фактологично, без оценочных суждений (YMYL)
   - ecom: название товара + ключевая характеристика
   - commercial_local: упоминание гео, если изображение его содержит
   - info: описательно, с контекстом раздела
6. Декоративные изображения → alt="" (пустая строка), role="presentation".
7. Помечай engine-split в поле meta:
   - [Я] если alt оптимизирован под Яндекс.Картинки (транслит в backup filename)
   - [G] если под Google Images (семантика + окружение)
   - [ОБА] если универсально (предпочтительный режим)

ВЫВОД: Валидный JSON без markdown-обёртки, без комментариев.
Язык содержания: РУССКИЙ.
Формат:
{
  "alt": "string",
  "alt_length": number,
  "engine_tag": "[Я]|[G]|[ОБА]",
  "keyword_included": boolean,
  "confidence": "◆ОБОСНОВАНО|◇ГИПОТЕЗА"
}
```

### 5.2. Пользовательский промпт (шаблон)

```
Сгенерируй alt-текст для изображения.

Страница: {{page_url}}
Тип страницы: {{page_type}}
Основной ключ: {{primary_keyword}}
Второстепенные ключи: {{secondary_keywords | join(', ')}}
Ниша: {{niche}}
Кластер: {{niche_cluster}}
Гео: {{geo}}

Данные об изображении:
- URL: {{image_src}}
- Имя файла: {{filename}}
- Размеры: {{width}}×{{height}}
- Окружающий текст (±200 символов): {{surrounding_text | default('нет')}}
- Заголовок ближайшего блока: {{nearest_heading | default('нет')}}
- Подпись (figcaption): {{figcaption | default('нет')}}

{{#if is_decorative_hint}}
Внимание: изображение похоже на декоративное (иконка, разделитель, фон). Рассмотри alt="".
{{/if}}

{{#if rag_context}}
Контекст конкурентов (как они называют похожие изображения):
{{rag_context}}
{{/if}}
```

**Бюджет fallback:** максимум 1 вызов Opus на изображение; в среднем ≤10% изображений проекта попадают в fallback после эвристик.

---

## ██ 6. RAG-ОБОГАЩЕНИЕ

Запросы к Qdrant делаются **только в fallback-ветке** генерации alt.

| **Коллекция** | **Запрос** | **Что достаём** | **Лимит** |
| --- | --- | --- | --- |
| `competitors` | ниша + `page_type` | Паттерны alt/filename топ-3 конкурентов | 500 токенов |
| `niches` | `niche_cluster` + `language` | Шаблоны alt для похожих проектов (ecom товары, medical симптомы и т.д.) | 500 токенов |
| `entities` | `primary_keyword` | Сущности Knowledge Graph, связанные с ключом — для семантического обогащения | 500 токенов |
| `image_patterns` (новая) | `niche_cluster` + `geo` | Накопленные эталонные alt/filename пары (растёт по мере работы системы) | 500 токенов |

**Общий лимит RAG-контекста:** 2000 токенов. При старте проекта `image_patterns` может быть пуста — это нормально, коллекция наполняется через `post_save`.

**Инжекция:** RAG-контекст подставляется в `{{rag_context}}` пользовательского промпта в виде структурированного блока:

```
### Паттерны конкурентов:
- [конкурент1.ru] товары на категорийных: "{category} {brand} купить в {geo}"
- [конкурент2.ru] article-страницы: краткое описание смысла блока, без ключа в начале

### Эталоны из ниши:
- medical, симптомы: "Локализация боли при {condition}, схема"
```

---

## ██ 7. СКРИПТЫ CLAUDE CODE

### 7.1. Pre-step скрипты

#### `step_12_1_pre_crawl_images.py`

```python
# Назначение: Краулинг страниц и извлечение всех <img> + <picture> + CSS background-image
# Вход: pages[] из input contract
# Выход: raw_images.json — { page_url: [ { src, alt, title, filename, css_selector,
#        surrounding_text, nearest_heading, figcaption, is_css_bg } ] }
# Действия:
#   1. Для каждой страницы: GET с user_agent, timeout 15с
#   2. Парсинг HTML через lxml/BeautifulSoup
#   3. Сбор <img>, <picture><source>, <input type="image">, inline background-image
#   4. Для каждого изображения извлечь:
#      - src (абсолютный URL через urljoin)
#      - alt, title, loading, fetchpriority, width/height атрибуты, srcset, sizes
#      - CSS-селектор (nth-child путь)
#      - Окружающий текст: 200 символов до + 200 после (для Opus-fallback)
#      - Ближайший H1-H6 выше в DOM
#      - <figcaption> если внутри <figure>
#   5. Респект robots.txt (robotparser)
#   6. Retry 3x с бэкоффом 2с→5с→15с при 5xx/429
#   7. Сохранение в NocoDB таблицу step12_1_raw_crawl
# Зависимости: requests, lxml, beautifulsoup4, urllib.robotparser
# Таймбюджет: 0.5с на страницу при параллелизме 10
```

#### `step_12_1_pre_analyze_media.py`

```python
# Назначение: Получение метаданных по каждому изображению (размер, формат, габариты)
# Вход: raw_images.json
# Выход: analyzed_images.json — добавлены file_size_kb, real_width, real_height,
#        real_format, has_exif, is_animated, dominant_colors_hash
# Действия:
#   1. Дедупликация по src (один URL — один запрос)
#   2. HEAD-запрос → Content-Length, Content-Type (быстро, без скачивания)
#   3. Если HEAD не дал габаритов → частичная загрузка первых 8KB (обычно хватает
#      для сигнатуры PNG/JPEG/WebP + IHDR/SOF/VP8)
#   4. Определение формата по magic bytes (не по расширению)
#   5. Проверка на анимацию (GIF, WebP animated, APNG)
#   6. Хэш доминирующих цветов (для детекции дубликатов и декоративных иконок)
#   7. Флаг is_decorative_candidate: маленький размер (<64px) + отсутствие alt + повторяется >3 раз
# Зависимости: httpx (async), Pillow (для сигнатур), imagehash
# Таймбюджет: 50ms на уникальное изображение при параллелизме 20
# Fallback: при недоступности изображения — метка status=unreachable, действие отложено
```

#### `step_12_1_pre_competitor_benchmark.py`

```python
# Назначение: Бенчмарк image SEO топ-3 конкурентов (опционально, только при первом запуске)
# Вход: project.main_competitors, niche_cluster
# Выход: competitor_image_benchmark.json — медианы по alt, filename, формату
# Действия:
#   1. Для каждого конкурента: краулинг топ-5 страниц через ту же логику
#   2. Агрегация: % изображений с alt, медиана alt_length, доля webp/avif,
#      средний вес страницы в KB, паттерны именования
#   3. Запись в Qdrant коллекцию competitors с тегами image_seo
#   4. Роль D помечает gap-зоны (где мы хуже медианы конкурентов)
# Запуск: раз в 14 дней (cron в N8N)
```

### 7.2. Основной скрипт (rule-based движок)

#### `step_12_1_main_audit.py`

```python
# Назначение: Детерминированная генерация списка actions по правилам Image SEO
# Вход: analyzed_images.json, PROJECT.md, thresholds, competitor_benchmark.json (опц.)
# Выход: actions.json по контракту Template 4
# Логика (правила применяются последовательно, каждое даёт 0-N действий):
#
# ПРАВИЛО 1 [ОБА]: alt_missing
#   - Если alt отсутствует И изображение не декоративное → severity=critical
#   - Recommendation.alt: попытка сборки из (primary_keyword + nearest_heading + figcaption)
#   - Если эвристика даёт строку длиной <5 или >125 → пометка fallback_to_opus=true
#
# ПРАВИЛО 2 [ОБА]: alt_low_quality
#   - Если alt содержит: "image123.jpg", "DSC_0001", "photo", "картинка", GUID, только цифры
#     → severity=high, fallback_to_opus=true
#   - Если alt длиннее 125 символов → severity=medium, обрезка + правка
#
# ПРАВИЛО 3 [ОБА]: filename_non_seo
#   - Регэксп на SEO-friendly: ^[a-z0-9]+(-[a-z0-9]+)*\.(webp|avif|jpg|jpeg|png|svg)$
#   - Если не соответствует → severity=medium
#   - Рекомендация: slugify(primary_keyword) + "-" + index + "." + target_format
#   - [Я]: Яндекс распознаёт транслит кириллицы, но ASCII-латиница предпочтительна (безопаснее для CDN/URL)
#   - [G]: Google считывает имя файла как слабый сигнал, ASCII обязательна
#
# ПРАВИЛО 4 [ОБА]: oversized
#   - Если file_size_kb > max_file_size_kb (200) → severity=high (critical если >500)
#   - Рекомендация: target_format=webp, compression_quality=82 (80 для фото, 90 для UI)
#   - Оценка экономии: (current - estimated_webp_size) в KB
#
# ПРАВИЛО 5 [G]: wrong_format (CWV-сигнал)
#   - jpg/png >100KB без webp-альтернативы → severity=high
#   - <picture><source type="image/avif"><source type="image/webp"><img ...></picture>
#   - Google: формат прямо влияет на LCP и score PageSpeed Insights
#
# ПРАВИЛО 6 [G]: no_lazy
#   - Если не hero-изображение (не в first viewport) И loading != "lazy" → severity=medium
#   - Критически важно для CLS и Interaction-to-Next-Paint
#
# ПРАВИЛО 7 [G]: no_dimensions (CLS-сигнал)
#   - Отсутствуют атрибуты width/height → severity=high (высокий вклад в CLS)
#   - Рекомендация: подставить real_width/real_height как HTML-атрибуты + CSS max-width:100%
#
# ПРАВИЛО 8 [ОБА]: no_srcset (responsive)
#   - Изображение шире 480px и без srcset → severity=medium
#   - Генерация: 320w, 768w, 1280w, 1920w (±2x для retina)
#
# ПРАВИЛО 9 [Я]: decorative_missing_role
#   - Маленькие иконки/разделители с alt → предложить alt="" + role="presentation"
#   - Яндекс лучше трактует декор при явной пометке, меньше "мусорных" alt в индексе картинок
#
# ПРАВИЛО 10 [Я]: missing_og_image_for_share
#   - Если на странице нет og:image → добавить действие для Шага 13
#   - Яндекс подтягивает og:image в ПФ-сниппеты, в Дзен-share и Turbo
#
# ПРАВИЛО 11 [G]: hero_without_fetchpriority
#   - LCP-кандидат (первое большое изображение в viewport) без fetchpriority="high"
#     → severity=medium
#
# ПРАВИЛО 12 [ОБА]: duplicate_image
#   - Одинаковый imagehash на ≥3 страницах с разным alt → пометка для ревью (кросс-шаговый QC)
#
# HitL: false. Все действия применяются автоматически на следующих шагах.
# Выход: actions[] отсортированы по severity desc, затем estimated_impact desc.
# Лимит: не более 500 actions за один запуск (иначе — пометка pagination_required).
```

### 7.3. Post-step скрипты

#### `step_12_1_post_validate.py`

```python
# Назначение: Структурная + содержательная валидация выхода
# Вход: actions.json
# Проверки (Роль G):
#   1. JSON Schema (ajv): соответствие расширенному Template 4
#   2. Каждый action имеет engine_tag ∈ {[Я], [G], [ОБА]}
#   3. При priority_engines=both: доля [ОБА] + ([Я] ∧ [G]) должна быть ≥95% (нет "голых" рекомендаций)
#   4. Уникальность action_id
#   5. Для type=update target существует в raw_crawl
#   6. recommendation.alt: длина в [min_alt_length, max_alt_length] ИЛИ явно alt=""
#   7. recommendation.filename соответствует регэкспу SEO-friendly
#   8. estimated_impact/effort ∈ [1,10]
#   9. Сумма severity-веса по actions коррелирует с summary.critical_issues (±5%)
#  10. Если есть действия по hero-изображениям — проверка fetchpriority="high"
# Выход: qc_score (0-100), фейл-лог при провале
```

#### `step_12_1_post_save.py`

```python
# Назначение: Персистентность результатов
# Действия:
#   1. NocoDB: upsert в таблицу step_outputs с prompt_version=step12_1_v1.0
#   2. NocoDB: детальные действия в step12_1_actions (для дашборда Grafana)
#   3. Qdrant collection image_patterns: upsert эмбеддингов пар (alt, surrounding_text)
#      для будущего RAG (роль F помечает как эталонные при qc_score ≥90)
#   4. Генерация image-sitemap.xml (draft → передача в Шаг 14)
#   5. Grafana: метрики images_total, critical_issues, avg_page_weight_before/after
#   6. Отправка события step_12_1.completed в N8N → триггер Шага 13
```

---

## ██ 8. КРИТЕРИИ QC (QC CRITERIA)

Rule-based (ML отключён до ≥200 размеченных образцов). Сумма весов = 100.

| **Проверка** | **Порог** | **Вес** | **При провале** |
| --- | --- | --- | --- |
| JSON-схема Template 4 (структурный QC) | 100% | 15 | Retry (строгий вывод) |
| Engine-маркировка на каждом action при `priority_engines=both` | ≥95% покрытия | 15 | Retry 1 (добавить теги) |
| alt-рекомендации: длина 5–125 символов (кроме пустых для декор.) | 100% валидных | 12 | Retry по отдельным action |
| filename-рекомендации: SEO-friendly регэксп | 100% | 10 | Авто-коррекция через slugify |
| Отсутствие keyword stuffing в alt (ключ ≤1 раз, alt не начинается с ключа в 100% случаев) | ≥95% | 10 | Retry 2 (fallback Opus) |
| Покрытие критических правил (alt_missing, oversized для hero) | 100% | 10 | Retry |
| Дубли action_id | 0 | 5 | Фатал, полный retry |
| Соответствие target_format правилам формата | ≥98% | 5 | Авто-коррекция |
| Согласованность `summary.critical_issues` с фактическим количеством | ±5% | 5 | Пересчёт |
| Наличие rationale для каждого action | 100% | 5 | Retry |
| Cross-step: каждый `image_src` из выхода присутствует в raw_crawl | 100% | 5 | Фатал, полный retry |
| Семантический QC: дубли рекомендаций alt на разных страницах | <5% дублей при cosine >0.9 | 3 | Ревью |

**Скоринг:** ≥80 → готово, передача в Шаг 13. 60–79 → ревью через Grafana-алерт (не блокирует пайплайн). <60 → retry по протоколу CORE v2.

---

## ██ 9. СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (событие step_12.completed ИЛИ cron 1 раз/неделю)
Нода 2:  Загрузка PROJECT.md из NocoDB (роль: контекст)
Нода 3:  Загрузка pages[] из Шага 04 + keywords из Шага 12
Нода 4:  [Code] step_12_1_pre_crawl_images.py (параллелизм 10)
Нода 5:  [Code] step_12_1_pre_analyze_media.py (параллелизм 20)
Нода 6:  [IF] первый запуск ИЛИ прошло 14 дней с последнего бенчмарка?
         ├─ Да → Нода 6a: step_12_1_pre_competitor_benchmark.py
         └─ Нет → пропуск
Нода 7:  [Code] step_12_1_main_audit.py (rule-based движок)
Нода 8:  [IF] есть действия с fallback_to_opus=true?
         ├─ Да → Нода 8a: Qdrant RAG (competitors, niches, entities, image_patterns)
         │       → Нода 8b: Сборка промптов (батчи по 10 изображений)
         │       → Нода 8c: Claude Opus API (JSON-режим, temp 0.15)
         │       → Нода 8d: Парсинг JSON, мердж с actions[]
         └─ Нет → пропуск
Нода 9:  [Code] step_12_1_post_validate.py
         └─ provalen → Нода 7 (retry по протоколу CORE v2, max 3)
Нода 10: [Code] step_12_1_post_save.py (NocoDB + Qdrant + image-sitemap draft)
Нода 11: Grafana metrics push
Нода 12: Trigger Шаг 13 (Schema.org + OG) ИЛИ stop при qc<60

Обработка ошибок:
  • Opus 429/500 (только в fallback): бэкофф 10с→30с→90с, макс 3
  • Qdrant недоступен: fallback_to_opus без RAG, лог WARN
  • Сайт недоступен (>10% страниц не закраулено): пауза пайплайна, алерт Telegram
  • QC-провал 3x подряд: алерт человеку (роль A ревьюит вручную через NocoDB UI)
  • Невалидный JSON из Opus: удаление markdown → повторный парсинг → retry
```

---

## ██ 10. FEW-SHOT ПРИМЕР ВЫХОДА (grounding, сокращённо)

Пример для ниши `medical` (детская стоматология, Москва), `priority_engines=both`:

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "12.1",
  "prompt_version": "step12_1_v1.0",
  "timestamp": "2025-04-16T10:23:14Z",
  "target_system": "cms",
  "summary": {
    "pages_scanned": 47,
    "images_total": 312,
    "images_with_issues": 198,
    "critical_issues": 34,
    "avg_page_weight_before_kb": 2840,
    "avg_page_weight_after_kb": 1120,
    "estimated_lcp_improvement_ms": 1100
  },
  "actions": [
    {
      "action_id": "IMG_001",
      "type": "update",
      "target": "https://example-dental.ru/lechenie-kariesa-u-detey > main > article > figure:nth-child(2) > img",
      "image_src": "https://example-dental.ru/wp-content/uploads/2024/09/DSC_00123.jpg",
      "issue_category": "alt_missing",
      "severity": "critical",
      "engine_tag": "[ОБА]",
      "confidence": "✓ПОДТВЕРЖДЕНО",
      "current_value": {
        "alt": null,
        "filename": "DSC_00123.jpg",
        "format": "jpg",
        "file_size_kb": 487,
        "width_px": 3024,
        "height_px": 4032,
        "has_lazy": false,
        "has_dimensions_attr": false,
        "has_srcset": false
      },
      "recommendation": {
        "alt": "Ребёнок на приёме у детского стоматолога в клинике в Москве",
        "filename": "lechenie-kariesa-u-detey-moskva.webp",
        "format": "webp",
        "compression_quality": 82,
        "target_dimensions": {"width": 1280, "height": 853},
        "loading": "lazy",
        "fetchpriority": "auto",
        "srcset": [
          "lechenie-kariesa-u-detey-moskva-320w.webp 320w",
          "lechenie-kariesa-u-detey-moskva-768w.webp 768w",
          "lechenie-kariesa-u-detey-moskva-1280w.webp 1280w"
        ],
        "sizes_attr": "(max-width: 768px) 100vw, 768px"
      },
      "rationale": "Отсутствует alt на content-изображении внутри article. Критический пробел для a11y и ранжирования в Яндекс.Картинках/Google Images. Оригинал 487KB JPEG — WebP q82 даст ~110KB (–77%), вес страницы снизится на ~370KB, LCP улучшится на ~800ms на 4G. Filename с primary_keyword + гео повышает релевантность в Яндекс.Картинках по коммерческому локальному интенту.",
      "estimated_impact": 9,
      "effort": 3,
      "validation_result": "pass"
    },
    {
      "action_id": "IMG_002",
      "type": "update",
      "target": "https://example-dental.ru/ > header > img.logo",
      "image_src": "https://example-dental.ru/wp-content/themes/dental/images/logo.png",
      "issue_category": "hero_without_fetchpriority",
      "severity": "medium",
      "engine_tag": "[G]",
      "confidence": "◆ОБОСНОВАНО",
      "current_value": {
        "alt": "Логотип",
        "filename": "logo.png",
        "format": "png",
        "file_size_kb": 42,
        "width_px": 240,
        "height_px": 60,
        "has_lazy": false,
        "has_dimensions_attr": true,
        "has_srcset": false
      },
      "recommendation": {
        "alt": "Детская стоматология «Пример», Москва — логотип",
        "filename": "logo-detskaya-stomatologiya-moskva.svg",
        "format": "svg",
        "compression_quality": null,
        "target_dimensions": {"width": 240, "height": 60},
        "loading": "eager",
        "fetchpriority": "high",
        "srcset": [],
        "sizes_attr": ""
      },
      "rationale": "Логотип — LCP-кандидат в хедере. SVG устранит ретина-проблему, fetchpriority='high' снизит LCP на ~150ms (Google). Alt расширен до брендовой формулы для CTR в Яндекс.Картинках на брендовые запросы.",
      "estimated_impact": 6,
      "effort": 2,
      "validation_result": "pass"
    },
    {
      "action_id": "IMG_003",
      "type": "configure",
      "target": "https://example-dental.ru/klinika-foto > section.gallery > img.divider",
      "image_src": "https://example-dental.ru/wp-content/themes/dental/images/wave-divider.png",
      "issue_category": "decorative_missing_role",
      "severity": "low",
      "engine_tag": "[Я]",
      "confidence": "◆ОБОСНОВАНО",
      "current_value": {
        "alt": "волна",
        "filename": "wave-divider.png",
        "format": "png",
        "file_size_kb": 8,
        "width_px": 1920,
        "height_px": 40,
        "has_lazy": true,
        "has_dimensions_attr": true,
        "has_srcset": false
      },
      "recommendation": {
        "alt": "",
        "filename": "wave-divider.svg",
        "format": "svg",
        "compression_quality": null,
        "target_dimensions": {"width": 1920, "height": 40},
        "loading": "lazy",
        "fetchpriority": "low",
        "srcset": [],
        "sizes_attr": ""
      },
      "rationale": "Декоративный разделитель. Яндекс.Картинки индексируют все alt — 'волна' создаёт мусор в индексе. Пустой alt + role='presentation' + перевод в SVG (инлайн ≤2KB).",
      "estimated_impact": 2,
      "effort": 1,
      "validation_result": "pass"
    }
  ],
  "execution_log": [
    "2025-04-16T10:19:02Z crawl started, 47 pages",
    "2025-04-16T10:20:44Z crawl finished, 312 images found, 4 unreachable",
    "2025-04-16T10:21:30Z analyze finished, 298 unique images metadata fetched",
    "2025-04-16T10:22:12Z rule engine applied, 198 actions generated",
    "2025-04-16T10:22:18Z 23 actions flagged fallback_to_opus → batch sent",
    "2025-04-16T10:23:01Z Opus fallback completed, 23 alt texts merged",
    "2025-04-16T10:23:14Z validation passed, qc_score=88"
  ],
  "image_sitemap_ready": true,
  "qc_score": 88
}
```

---

## ██ 11. СТРАТЕГИЯ ОТКАТА (FALLBACK STRATEGY)

При 3-кратном провале QC применяется следующая эскалация.

**Retry 1 (CORE protocol):** Инжекция списка ошибок QC в промпт fallback-ветки Opus. Тот же `temperature=0.15`. Применяется только к изображениям, попавшим в Opus-ветку.

**Retry 2:** Декомпозиция — разделить actions[] на 2 подзадачи:
- Подзадача A: только alt-генерация (изображения с `fallback_to_opus=true`)
- Подзадача B: структурные правки (formats, dimensions, loading) — полностью детерминированно, без Opus
Объединение результатов после валидации каждой подзадачи отдельно.

**Retry 3 (Fallback-промпт):** Строгий промпт с явным few-shot примером ожидаемого alt (из коллекции `image_patterns` с `qc_score≥90`). `temperature=0.1`, `max_tokens=1024`.

**Откат А — Деградация до базовых правил:** Если Opus-ветка нестабильна — для всех изображений без alt применяется шаблонный эвристический alt: `{primary_keyword} — {page_type} — изображение {index}`. Помечается `confidence=⚠РИСК`, передаётся в HitL-очередь роли A для ручного ревью.

**Откат B — Партицирование:** Если `images_total > 500` — разбить на батчи по 100 страниц и запускать последовательно с паузой 5 минут (снижение нагрузки на сайт-источник и CDN-источники).

**Откат C — Skip-режим (emergency):** При недоступности сайта >30% — шаг завершается со статусом `partial`, обрабатываются только успешно закраеннные страницы. Алерт в Telegram роли B.

**Откат D — Заморозка правил:** При обнаружении массовых false-positive (>20% actions отвергаются на HitL-ревью) — автоматическая блокировка соответствующих правил до разбора. Алерт роли F.

---

## ██ 12. АДАПТАЦИИ ПОД NICHE CLUSTER

| **Кластер** | **Специфика Image SEO** |
| --- | --- |
| **commercial_local** | Alt с гео-маркером: `{действие} {в/на} {город}`. [Я] ПФ-сигнал: изображения витрины/фасада для Яндекс.Бизнес. Filename включает гео-токен. Высокий приоритет og:image для Яндекс.Дзен и Turbo-шаринга. |
| **commercial_national** | Фокус на брендовую фотографию, широкий srcset для разных устройств. [G] приоритет fetchpriority='high' на hero (Core Web Vitals критичны для national-конкуренции). AVIF где возможно. |
| **ecom** | Alt: `{товар} {ключевая характеристика} {бренд}`. Обязательная Schema.org ImageObject (передача в Шаг 13). Галереи товаров: первое изображение — `loading=eager`, остальные — `lazy`. srcset обязателен (разные плотности экрана). [Я] Яндекс.Маркет тянет og:image и product image. |
| **saas** | Скриншоты UI с текстовым alt для поиска по фичам: `{feature name} в {product}, интерфейс`. [G] SGE/AI Mode охотно цитирует размеченные ImageObject. Английские дубли alt при `international=true`. |
| **info** | Инфографика, схемы — alt с полным объяснением смысла (до 125 символов). `<figcaption>` обязателен (Шаг 13 учитывает). Низкий эффект ccompression (не сжимать ниже q85 для читаемости мелкого текста на схемах). |
| **medical** | **YMYL-режим:** alt строго фактологичен, без оценок. Запрещены продающие формулировки. Сохранение EXIF недопустимо (приватность). Схемы/снимки — пометка в alt: `схема`, `рентгенограмма`, `илл.`. Подписи (figcaption) проверяются на соответствие регуляторным требованиям (роль A в HitL-очереди). |
| **legal** | Фото адвокатов/офиса — alt с именем и статусом: `{ФИО}, {должность/статус}`. E-E-A-T сигнал. Запрет на стоковые изображения людей (детектор через imagehash против известных стоковых баз). |
| **education** | Alt для учебных материалов с явным уровнем: `{предмет}, {уровень: школа/вуз}, {тема}`. Сезонность: в август-сентябрь ставить приоритет на обновление изображений основных страниц. |

---

## ██ 13. МЕТРИКИ И КРИТЕРИИ УСПЕХА

**Опережающие (сразу после шага):**
- `qc_score ≥ 80`
- `images_with_issues / images_total` до шага → количество issues
- Все `priority_engines=both` страницы имеют ≥95% action'ов с engine-маркировкой
- `estimated_lcp_improvement_ms` > 0 для ≥70% страниц с hero-изображениями

**Запаздывающие (через 14–30 дней после применения action'ов):**
- [G] PageSpeed Insights: LCP улучшение ≥15%, CLS → <0.1
- [G] Google Search Console → Images: impressions +≥20%
- [Я] Яндекс.Вебмастер → Индексирование/Картинки: количество проиндексированных картинок +≥30%
- [Я] Яндекс.Метрика → Просмотры из Картинок: +≥10%
- [ОБА] Общий вес страницы: –50% в среднем
- [ОБА] Доля страниц с FCP<1.8s: +≥20 п.п.

**Контрольные (через 3 месяца):**
- ROI: рост органического трафика с картинок × средний LTV / затраты на storage и CDN
- AEO-метрика: `citation_rate` изображений в AI-ответах (Шаг 40.3) для AI Mode
- Снижение bounce rate на страницах-получателях трафика из Картинок

**Анти-метрики (не должно ухудшиться):**
- Качество визуала: отсутствие жалоб от клиента на «плохо выглядит» (ручной QA на топ-20 страницах после применения)
- Количество 404 на изображениях (при переименовании filename должен быть настроен 301 или rewrite — проверка в Шаге 14.1)

---

## ██ 14. CHANGELOG

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2026-04-16 | Начальная версия. Rule-based движок (12 правил), fallback Opus для alt, расширение Template 4 полями `engine_tag`/`confidence`/`summary`, 8 адаптаций под niche_cluster. |
| v1.1 (план) | Q3 2026 | Интеграция AVIF с детекцией поддержки по User-Agent через CDN-правила. Правило 13: проверка og:image размеров (1200×630) для соц-шаринга. |
| v2.0 (план, после ≥200 образцов) | 2027 | Включение ML-скоринга качества alt (fine-tune на собранных `image_patterns`). Семантический QC через cosine similarity alt-текстов между страницами. |

---

## ██ ПРИЛОЖЕНИЕ: КРОСС-ШАГОВЫЕ ЗАВИСИМОСТИ

**Получает данные от:**
- `STEP_04.md` — `structure.pages[]` (URL и тип каждой страницы)
- `STEP_12.md` — `content_blocks[].keywords[]`, `meta` (контекст для alt)
- `PROJECT.md` — `priority_engines`, `niche_cluster`, `geo`, `language`, `main_competitors`

**Передаёт данные в:**
- `STEP_13.md` (Schema.org + OG) — список изображений для `ImageObject`, флаг `image_sitemap_ready`
- `STEP_14.md` (Sitemap) — черновик `image-sitemap.xml`
- `STEP_16.md` (Скорость) — `summary.estimated_lcp_improvement_ms` для верификации после применения
- `STEP_17.md` (Финальный аудит) — `summary.critical_issues` как один из входов gate

**Общие антипаттерны этого шага (роль A запрещает):**
- ✗ Generic alt вида `"изображение товара"` без конкретики
- ✗ Keyword stuffing: alt = список ключей через запятую
- ✗ Пропуск engine-маркировки при `priority_engines=both`
- ✗ Автоматическое применение без Шага 14.1 (редиректы) при переименовании filename
- ✗ Использование Opus на горячем пути для всех изображений (противоречит типу «Скрипт»)
- ✗ Агрессивное сжатие (q<75) на инфографике/medical-схемах

---

**Конец STEP_12.1.md · step12_1_v1.0**
