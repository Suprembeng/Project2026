# МОДУЛЬ 3: STEP_12.3.md — UGC-стратегия ★

**SEO AUTOMATION ENGINEERING · v6.0 · КОНФИДЕНЦИАЛЬНО**

Самодостаточный модуль. Загружается: `CORE.md` + `PROJECT.md` + этот файл. ~4K токенов.

Наследует: **Шаблон 3 (Генерация контента)**. Primary Role: **A + E**. Тип: **Промпт**.

Статус: **★ новый шаг v6.0** (одна из семи новых фич: IndexNow, Link Decay, AI Citation Tracking, Intent Validation, Core Update Readiness, JS Rendering Audit, **UGC Strategy**).

---

## ██ КАРТОЧКА ШАГА

| **Параметр**        | **Значение**                                                                                                                                                                                                             |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ID                  | 12.3                                                                                                                                                                                                                     |
| Название            | UGC-стратегия (User-Generated Content)                                                                                                                                                                                   |
| Блок                | 3: Контент и оптимизация                                                                                                                                                                                                 |
| Основная роль       | A: SEO-стратег + E: CRO-аналитик (со-ведущие)                                                                                                                                                                            |
| Консультативные     | F: Промпт-инженер, D: Аналитик конкур. разведки, G: Инженер контрактов данных                                                                                                                                            |
| Тип                 | **Промпт** — вся логика в Opus, скрипты только пост-валидация                                                                                                                                                            |
| HitL                | Нет (результат → Шаги 13, 19, 26, 27, 29 автоматически). Исключение: YMYL-кластер с ≥1 ⚠РИСК → ручное ревью Role A                                                                                                       |
| Приоритет ROI       | ВЫСОКИЙ — UGC даёт уникальный масштабируемый контент, E-E-A-T сигналы, долгий хвост Q&A-запросов, цитирование в Яндекс Нейро и Google AI Mode (человеческий голос против AI-SEO-контента конкурентов)                    |
| Зависит от          | `PROJECT.md` (`ugc_enabled`), Шаг 00 (персоны), Шаг 04 (структура страниц), Шаг 12 (SEO-текст), Шаг 12.1 (Image SEO — для UGC-фото от клиентов), Шаг 12.2 (Video SEO — для видео-отзывов), Шаг 05 (коммерч. факторы — существующие отзывы) |
| Передаёт в          | Шаг 13 (Schema.org: Review, FAQPage, Comment, DiscussionForumPosting), Шаг 19 (Бизнес-профили: Я.Бизнес/GBP-отзывы), Шаг 26 (ORM-стратегия), Шаг 27 (Блог/Контент-хаб: комментарии), Шаг 29 (AEO: UGC-цитаты в AI), Шаг 40.3 (AI-citation tracking), Шаг 05 (обратная связь по КФ) |
| Режим Opus          | Генерация контента: temperature 0.3, top_p 0.9, max_tokens 8192 (баланс креатива для incentive-формулировок и точности JSON-схемы)                                                                                       |
| Наследуемый шаблон  | Шаблон 3 (Генерация контента) — расширен полями `ugc_mechanics[]`, `ugc_strategy`, `business_profiles_sync`, `moderation_policy`, `handoff_to_13/19/26/29/40.3`, `summary`, `status`, `failed_pages`                     |
| Prompt Version      | step12_3_v1.0                                                                                                                                                                                                            |

---

## ██ ЦЕЛЬ

Для каждой страницы проекта сгенерировать полную UGC-стратегию: выбор механик (отзывы, Q&A, комментарии, фото-отзывы, видео-тестимониалы, рейтинги, форум, user stories) с учётом кластера ниши, `ugc_enabled` флага проекта, YMYL-ограничений, плотности трафика и персон; определить `placement`, `schema_markup` (Review/FAQPage/Comment/DiscussionForumPosting/AggregateRating), `moderation_policy` (auto-фильтры + SLA), `incentives`, `ui_prompts` для приглашения пользователей, стартовый `seeding_strategy` и синхронизацию с Я.Бизнес/GBP — всё для роста E-E-A-T, long-tail покрытия и цитирования в AI-поисковиках.

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход наследуется от **output Шага 12.2 (Video SEO, Шаблон 3 расширенный)** + подтягивает контекст Шага 12, Шага 05, Шага 00 и `PROJECT.md`.

```json
{
  "project_id": "string",
  "step_id": "12.3",
  "prompt_version": "step12_3_v1.0",
  "timestamp": "ISO 8601",
  "project_context": {
    "niche": "string",
    "niche_cluster": "commercial_local|commercial_national|ecom|saas|info|medical|legal|education",
    "geo": "string",
    "language": "string",
    "priority_engines": "yandex|google|both",
    "domain": "string",
    "ugc_enabled": "boolean",
    "international": "boolean",
    "project_phase": "setup|growth|maintenance|recovery",
    "monthly_traffic": "number|null",
    "main_competitors": ["string"],
    "existing_business_profiles": {
      "yandex_business": {"exists": "boolean", "rating": "number|null", "reviews_count": "number|null", "url": "string|null"},
      "google_business_profile": {"exists": "boolean", "rating": "number|null", "reviews_count": "number|null", "url": "string|null"},
      "2gis": {"exists": "boolean", "url": "string|null"}
    }
  },
  "pages_to_process": [
    {
      "page_url": "string",
      "content_type": "landing|category|article|faq|product",
      "total_word_count": "number",
      "target_queries": [{"query": "string", "engine": "yandex|google|both", "intent": "informational|commercial|navigational|transactional", "hunt_level": "number 1-5"}],
      "page_meta": {"title": "string", "description": "string"},
      "aeo_elements": {"has_how_to": "boolean", "faq_count": "number"},
      "persona_preferences": {
        "primary_persona_id": "string",
        "preferred_formats": ["string"],
        "ai_search_usage": "none|sometimes|primary"
      },
      "existing_ugc": {
        "reviews_on_page": "number",
        "avg_rating": "number|null",
        "qa_count": "number",
        "has_review_schema": "boolean",
        "source": "step_05|scraped|none"
      },
      "video_strategy_from_12_2": {
        "has_video": "boolean",
        "video_role": "string|null",
        "tracking_id": "string|null"
      }
    }
  ],
  "commercial_factors_context": {
    "orm_status_from_step_26": "not_started|in_progress|mature",
    "competitor_ugc_benchmarks": [{"domain": "string", "review_count": "number", "rating": "number", "has_qa": "boolean"}]
  }
}
```

**Примечание (согласно инструкции «тип шага = Промпт»):** pre-скрипты отсутствуют. Все данные поступают через input contract из предыдущих шагов и PROJECT.md. Скрипты Claude Code содержат **только post-step валидатор**.

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Роль G валидирует. Наследует **Шаблон 3** с расширением под UGC. Уроки из аудита 12.2 учтены с самого начала: `confidence`, `data_sources`, `handoff_to_*`, `summary`, `status`, `failed_pages`.

```json
{
  "project_id": "string",
  "step_id": "12.3",
  "prompt_version": "step12_3_v1.0",
  "timestamp": "ISO 8601",
  "status": "complete|partial|failed|disabled",
  "summary": {
    "pages_scanned": "number",
    "pages_with_mechanics": "number",
    "pages_ugc_disabled": "number",
    "mechanics_total": "number",
    "mechanic_types_distribution": {"review": "number", "qa": "number", "comment": "number", "photo": "number", "video_testimonial": "number", "rating": "number", "forum": "number", "user_story": "number"},
    "ymyl_pages_with_safe_ugc": "number",
    "ymyl_pages_with_disabled_qa": "number",
    "business_profiles_sync_needed": "boolean",
    "total_seeding_volume_target_monthly": "number"
  },
  "pages": [
    {
      "page_url": "string",
      "content_type": "landing|category|article|faq|product",
      "ugc_strategy": {
        "mechanics_count": "number (0-5)",
        "primary_mechanic": "review|qa|comment|photo|video_testimonial|rating|forum|user_story|none",
        "ymyl_restriction_level": "full_allowed|reviews_only|limited_qa|disabled",
        "rationale": "string",
        "confidence": "✓ПОДТВЕРЖДЕНО|◆ОБОСНОВАНО|◇ГИПОТЕЗА|⚠РИСК",
        "data_sources": ["project_context", "persona_analysis", "competitor_benchmark", "step_05_orm", "rag:ugc_patterns"]
      },
      "ugc_mechanics": [
        {
          "mechanic_id": "string (UGC001...)",
          "tracking_id": "string (UUID для Шага 40.3)",
          "type": "review|qa|comment|photo|video_testimonial|rating|forum|user_story",
          "placement": "mid_content|sidebar|above_footer|dedicated_section|product_card|faq_section",
          "engine_tag": "[Я]|[G]|[ОБА]",
          "target_queries": [{"query": "string", "engine": "yandex|google|both"}],
          "confidence": "✓ПОДТВЕРЖДЕНО|◆ОБОСНОВАНО|◇ГИПОТЕЗА|⚠РИСК",
          "schema_markup": {
            "@context": "https://schema.org",
            "@type": "Review|FAQPage|Comment|DiscussionForumPosting|AggregateRating|ImageObject",
            "fields_template": "object (шаблон с плейсхолдерами {{USER_NAME}}, {{REVIEW_DATE}}, {{RATING}}, {{REVIEW_BODY}})"
          },
          "moderation": {
            "type": "manual|auto|hybrid",
            "auto_filters": ["profanity", "spam_urls", "competitor_mentions", "ymyl_banned_phrases", "pii_leak"],
            "sla_hours": "number (4-48)",
            "escalation_rules": "string (когда передавать в ручную модерацию)"
          },
          "incentive": {
            "type": "discount|loyalty_points|gift|recognition|early_access|none",
            "value_placeholder": "string (напр. '{{DISCOUNT_PERCENT}}% на следующий заказ')",
            "legal_compliance_note": "string (ограничения по законодательству РФ: ст. 14.3 КоАП, ЗоЗПП, реклама)"
          },
          "ui_prompts": {
            "invite_text": "string (текст приглашения оставить UGC)",
            "form_fields": [{"field_name": "string", "type": "text|rating|photo_upload|select", "required": "boolean", "max_length": "number|null"}],
            "confirmation_text": "string",
            "anti_spam_hidden_field": "boolean (honeypot/CAPTCHA)"
          },
          "seeding_strategy": {
            "description": "string (как запустить UGC с нуля)",
            "initial_volume_target": "number (10-100 первых UGC для старта)",
            "channels": ["email_campaign", "loyalty_program", "post_purchase_form", "business_profile_import", "influencer_seeding", "none"],
            "timeline_days": "number"
          },
          "disclaimer_text": "string|null (ОБЯЗАТЕЛЬНО для medical/legal)",
          "collection_volume_target_monthly": "number",
          "engagement_lift_estimate": {
            "metric": "dwell_time_pct|bounce_rate_delta_pct|ctr_lift_pct",
            "estimated_value": "number",
            "confidence": "string"
          },
          "ai_citation_potential": "high|medium|low",
          "data_sources": ["string"]
        }
      ],
      "aeo_elements": {
        "enables_rich_results": ["Review", "FAQPage", "DiscussionForumPosting", "AggregateRating"],
        "ugc_citation_potential": "high|medium|low",
        "integrates_with_video_12_2": "boolean",
        "hook_for_ai_citation": "string (как UGC попадёт в AI-ответы)"
      },
      "engine_tags": {
        "yandex_specific": ["string (Я.Бизнес-привязка, Я.Дзен, Я.Услуги)"],
        "google_specific": ["string (GBP, Reviews schema, Reddit/Quora seeding)"]
      }
    }
  ],
  "business_profiles_sync": {
    "yandex_business": {
      "action_needed": "create|update|optimize|skip",
      "review_import_strategy": "string",
      "photo_sync_from_ugc": "boolean",
      "priority_score": "number 1-10"
    },
    "google_business_profile": {
      "action_needed": "create|update|optimize|skip",
      "review_import_strategy": "string",
      "qa_seeding_plan": "string",
      "priority_score": "number 1-10"
    },
    "2gis": {
      "action_needed": "create|update|skip",
      "priority_score": "number 1-10"
    }
  },
  "moderation_global_policy": {
    "banned_phrases_common": ["string"],
    "banned_phrases_ymyl": ["string (для medical/legal: 'лечит', 'гарантирует', 'излечение', '100%', 'без побочных', 'мгновенно')"],
    "pii_patterns_to_strip": ["phone", "email", "passport_number", "insurance_number"],
    "escalation_triggers": ["negative_sentiment_with_accusation", "legal_threat_mention", "competitor_explicit_mention"],
    "response_templates": [
      {"trigger": "negative_review", "template_placeholder": "string"},
      {"trigger": "question_unanswered", "template_placeholder": "string"}
    ]
  },
  "handoff_to_13": {
    "schema_graph_per_page": [
      {"page_url": "string", "graph_additions": ["Review", "FAQPage", "AggregateRating", "..."]}
    ]
  },
  "handoff_to_19": {
    "business_profile_actions": [
      {"platform": "yandex_business|gbp|2gis", "action": "string", "priority": "high|medium|low"}
    ]
  },
  "handoff_to_26": {
    "orm_triggers_to_monitor": [
      {"trigger_type": "string", "page_url": "string", "alert_threshold": "string"}
    ]
  },
  "handoff_to_29": {
    "ugc_ai_citation_targets": [
      {"page_url": "string", "expected_ai_citation_format": "string (quote|aggregate_rating|qa_answer)", "target_queries": ["string"]}
    ]
  },
  "handoff_to_40_3": [
    {"ugc_tracking_id": "string", "page_url": "string", "mechanic_type": "string", "expected_citation_queries": ["string"]}
  ],
  "failed_pages": [
    {"url": "string", "reason": "string", "retry_count": "number"}
  ],
  "qc_score": "number 0-100",
  "qc_breakdown": {"check_name": "score"}
}
```

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — со-ведущие Роли A (Senior SEO-стратег, 10+ лет) + E (CRO и поведенческий аналитик).
Консультанты: F (Промпт-инженер), D (Аналитик конкур. разведки), G (Инженер контрактов данных).

Задача: Сгенерировать UGC-стратегию для набора страниц проекта — типы механик, их placement,
schema_markup, moderation_policy, incentives, ui_prompts, seeding_strategy, синхронизацию
с бизнес-профилями Я.Бизнес/GBP/2GIS.

ЖЁСТКИЕ ПРАВИЛА:

1. Флаг ugc_enabled:
   - Если project_context.ugc_enabled=false → ВСЕ страницы получают ugc_mechanics=[],
     ugc_strategy.primary_mechanic="none", status="disabled", rationale объясняет причину.
     Возврат минимального output с summary.pages_ugc_disabled=pages_scanned.
   - Если true → генерируй механики по правилам ниже.

2. Количество механик на страницу (mechanics_count):
   - 0 для content_type='faq' с total_word_count<300 и без коммерч. интента.
   - 1 по умолчанию (primary_mechanic).
   - 2-3 для content_type ∈ {landing, category, product} с monthly_traffic ≥ 100 сессий/мес.
   - 4-5 ТОЛЬКО для hub-страниц с total_word_count > 3000 И niche_cluster ∈ {ecom, saas}.

3. Выбор типа механик по content_type:
   - landing/category: review + rating + qa
   - product: review + photo + rating + qa
   - article: comment + qa
   - faq: qa (расширяется через UGC-вопросы)

4. Engine tags ОБЯЗАТЕЛЬНЫ на каждой механике:
   - [Я]: Я.Бизнес-отзывы, Я.Услуги-отзывы, Я.Дзен-комментарии, Я.Карты, 2GIS-отзывы, Яндекс Нейро-цитаты
   - [G]: Google Business Profile Reviews, Google Maps, Reviews schema для SERP-звёзд, Reddit/Quora-seeding, AI Mode
   - [ОБА]: Review/FAQPage/Comment schema, AggregateRating, DiscussionForumPosting, модерация, incentive-механизмы

5. Schema.org markup ОБЯЗАТЕЛЬНО для каждой механики:
   - type='review' → Review + AggregateRating (если ≥3 отзывов)
   - type='qa' → FAQPage или QAPage
   - type='comment' → Comment (в Article) или DiscussionForumPosting
   - type='photo' → ImageObject с authorReview
   - type='video_testimonial' → VideoObject (интеграция с Шагом 12.2 через tracking_id)
   - type='rating' → AggregateRating
   - type='forum' → DiscussionForumPosting
   - type='user_story' → Article с author=Person (UGC-автор)
   ВСЕ поля schema_markup ДОЛЖНЫ быть с плейсхолдерами: {{USER_NAME}}, {{REVIEW_DATE}}, {{RATING}}, {{REVIEW_BODY}}, {{USER_PHOTO_URL}}, {{VIDEO_URL}}. НИКОГДА не выдумывай реальные имена или отзывы.

6. YMYL-ограничения (niche_cluster ∈ {medical, legal}) — КРИТИЧНО:
   - ymyl_restriction_level по умолчанию = "reviews_only"
   - ЗАПРЕТ на type='qa', type='comment', type='forum', type='user_story' где пользователи могут давать медицинские или юридические советы другим пользователям
   - Разрешено ТОЛЬКО: type='review' (о сервисе, не о лечении), type='rating', type='photo' (до/после только с согласием и дисклеймером, БЕЗ идентификации)
   - disclaimer_text ОБЯЗАТЕЛЬНО: "Отзыв отражает личный опыт. Не является медицинской/юридической рекомендацией."
   - moderation.type='manual' или 'hybrid' (НЕ 'auto')
   - moderation.auto_filters ОБЯЗАТЕЛЬНО содержат 'ymyl_banned_phrases'
   - incentive.type ∈ {'recognition', 'loyalty_points', 'none'} — ЗАПРЕЩЕНО 'discount' за отзывы (ст. 14.3 КоАП, реклама мед. услуг ФЗ-38)
   - Для medical: ЗАПРЕТ фото-отзывов с диагнозами, рентгенами, результатами анализов.

7. Incentive compliance (все кластеры):
   - ЗАПРЕТ денежных стимулов за положительные отзывы (вводит в заблуждение).
   - РАЗРЕШЕНО: стимул за ЛЮБОЙ отзыв (положительный или отрицательный) — должно быть явно указано.
   - Для medical/legal — см. правило 6.
   - Всё в value_placeholder — в формате плейсхолдера ({{DISCOUNT_PERCENT}}%, {{LOYALTY_POINTS}} баллов), без реальных цифр.
   - Обязательное поле legal_compliance_note для каждого incentive.

8. Moderation — СТРОГО:
   - auto_filters: минимум ['profanity', 'spam_urls', 'pii_leak'] для всех.
   - Для YMYL: + 'ymyl_banned_phrases'.
   - Для ecom: + 'competitor_mentions'.
   - sla_hours: 4-8 для ecom/commercial, 12-24 для info, 24-48 для YMYL (требует экспертной модерации).
   - escalation_rules: текстовое описание когда передавать в ручную (юр. угрозы, explicit negative, обвинения).

9. Seeding strategy:
   - Обязательно initial_volume_target (10-100 UGC) для старта.
   - channels: не может быть пустым, минимум 1 канал.
   - Для commercial_local: приоритет 'business_profile_import' + 'post_purchase_form'.
   - Для ecom: 'post_purchase_form' + 'email_campaign' + 'loyalty_program'.
   - Для saas: 'community_forum' (создание) + 'email_campaign'.
   - Для medical/legal: 'business_profile_import' (Я.Бизнес/GBP) + 'post_consultation_form'.

10. Бизнес-профили (business_profiles_sync):
    - Для niche_cluster ∈ {commercial_local, medical, legal} — action_needed НЕ 'skip' хотя бы для одного профиля.
    - priority_score 8-10 для commercial_local без существующих профилей.
    - Для commercial_national/ecom — обычно 'optimize' с priority 3-6.

11. AI-citation optimization:
    - ai_citation_potential='high' при ≥10 планируемых отзывов/мес И наличии Review schema.
    - hook_for_ai_citation — текстовое описание, как UGC попадёт в Яндекс Нейро и Google AI Mode (например: «цитаты из реальных отзывов с AggregateRating дают 2-3х вероятность цитирования в AI-ответе на запросы commercial intent»).

12. Cross-step integration:
    - Если video_strategy_from_12_2.has_video=true → ОБЯЗАТЕЛЬНО добавляй механику 'video_testimonial' с privacy_ref на Шаг 12.2 tracking_id.
    - Если existing_ugc.reviews_on_page>0 → наращивать, не дублировать механику type='review'.
    - Если commercial_factors_context.orm_status_from_step_26='mature' → moderation.type='hybrid', иначе 'manual' на старте.

ЗАПРЕЩЕНО (ВЫСОКИЙ РИСК):

- Выдумывать реальные имена пользователей, тексты отзывов, рейтинги. ВСЕГДА плейсхолдеры {{USER_NAME}}, {{REVIEW_BODY}}, {{RATING}}, {{REVIEW_DATE}}, {{VIDEO_URL}}, {{USER_PHOTO_URL}}.
- Выдумывать статистику конкурентов. Только из commercial_factors_context.competitor_ugc_benchmarks. Если данных нет → confidence="⚠РИСК", rationale_uncertain: true.
- Генерировать incentive с конкретными денежными суммами. Всегда плейсхолдер ({{DISCOUNT_PERCENT}}%, {{LOYALTY_POINTS}}).
- Для YMYL предлагать type='qa' или type='forum' где пользователи обсуждают диагнозы/лечение/юридические кейсы.
- Рекомендовать incentive 'discount' или 'gift' за отзыв в medical/legal (нарушает ФЗ-38 «О рекламе» и ст. 14.3 КоАП).
- Копировать текст приглашения (ui_prompts.invite_text) у конкурентов.

ВЫВОД: Валидный JSON строго по контракту. Без markdown-обёртки. Без комментариев.
Весь человекочитаемый текст — на РУССКОМ (или page_language при international=true).
JSON-ключи — на английском.

НАПОМИНАНИЕ: JSON без markdown-обёртки.
```

### Пользовательский промпт (шаблон)

```
Сгенерируй UGC-стратегию для проекта.

Проект:
  niche: {{project_context.niche}}
  niche_cluster: {{project_context.niche_cluster}}
  geo: {{project_context.geo}}
  language: {{project_context.language}}
  priority_engines: {{project_context.priority_engines}}
  ugc_enabled: {{project_context.ugc_enabled}}
  monthly_traffic: {{project_context.monthly_traffic}}
  international: {{project_context.international}}
  existing_business_profiles: {{project_context.existing_business_profiles | json}}

{{#if project_context.ugc_enabled == false}}
⚠️ UGC отключён на уровне проекта. Верни минимальный output: для каждой страницы ugc_mechanics=[], status="disabled", rationale объясняет что UGC отключён. summary.pages_ugc_disabled = общему числу страниц.
{{/if}}

ORM-статус (из Шага 26): {{commercial_factors_context.orm_status_from_step_26}}
Бенчмарки UGC конкурентов: {{commercial_factors_context.competitor_ugc_benchmarks | json}}

Страницы ({{pages_to_process.length}} шт.):
{{#each pages_to_process}}
---
URL: {{page_url}}
Type: {{content_type}}, total_word_count: {{total_word_count}}
Title: {{page_meta.title}}
Персона: {{persona_preferences.primary_persona_id}} (AI-search: {{persona_preferences.ai_search_usage}})
Целевые запросы: {{target_queries | json}}
has_how_to: {{aeo_elements.has_how_to}}, faq_count: {{aeo_elements.faq_count}}
Существующий UGC: {{existing_ugc | json}}
Видео из 12.2: has_video={{video_strategy_from_12_2.has_video}}, role={{video_strategy_from_12_2.video_role}}, tracking={{video_strategy_from_12_2.tracking_id}}
{{/each}}

{{#if rag_context}}
RAG context (паттерны UGC из похожих ниш, успешные seeding-стратегии):
{{rag_context}}
{{/if}}

НАПОМИНАНИЕ: JSON-объект корневого уровня со всеми обязательными полями (summary, pages[], business_profiles_sync, moderation_global_policy, handoff_to_13/19/26/29/40.3, failed_pages, qc_score, qc_breakdown). Без markdown.
```

---

## ██ RAG-ОБОГАЩЕНИЕ

Лимит RAG-контекста: **2000 токенов**. Запросы к Qdrant перед вызовом Opus.

| **Коллекция**         | **Запрос (эмбеддинг)**                                             | **Что достаём**                                                                            |
|-----------------------|--------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
| `ugc_patterns`        | `niche_cluster` + `geo` + `language`                               | Успешные UGC-механики в похожих проектах, медианные объёмы, working placements             |
| `competitors`         | `niche` + `main_competitors` → filter `has_ugc=true`               | Структуры UGC лидеров: типы, placements, volume, Review schema usage                       |
| `personas`            | `project_id` (из Шага 00)                                          | `content_preferences.writes_reviews`, `engagement_style` (lurker/commenter/creator)        |
| `legal_constraints`   | `niche_cluster` + `geo` (фильтр `ymyl=true` для medical/legal)     | Ограничения РФ: ФЗ-38, ст. 14.3 КоАП, ЗоЗПП, специфика мед./юр. рекламы                   |

Деградация: Qdrant недоступен → fallback на few-shot в system prompt, `qc_score -= 5`, WARN-лог.

---

## ██ СКРИПТЫ CLAUDE CODE

**Тип шага = Промпт.** Pre-скрипты отсутствуют. Только post-step валидатор (согласно инструкции архитектуры).

### Пост-скрипт: step_12_3_post_validate.py

```python
# Назначение: Валидация выхода Opus по JSON-схеме + бизнес-правилам UGC.
# Вход: opus_output.json + input contract (для cross-step проверок)
# Выход: {"qc_passed": bool, "qc_score": 0-100, "qc_breakdown": {...}, "issues": [...]}
#
# Проверки организованы по 7 слоям CORE (по образцу Шага 12.2 v1.1):
#
# L1 (Структурный):
#   1. JSON Schema валиден (ajv-compatible)
#   2. Уникальность mechanic_id в рамках страницы
#   3. Уникальность tracking_id глобально
#   4. Обязательные поля: summary, pages[], business_profiles_sync, moderation_global_policy,
#      handoff_to_13/19/26/29/40.3
#
# L2 (Содержательный):
#   5. ugc_enabled=false → ВСЕ pages с ugc_mechanics=[] И status="disabled"
#   6. mechanics_count соответствует content_type (faq<300w → 0; landing/product → 1-3; hub → 2-5)
#   7. Engine_tag на каждой механике ∈ {[Я], [G], [ОБА]}
#   8. Schema markup: @type соответствует type механики (review→Review; qa→FAQPage/QAPage; ...)
#   9. Все schema-поля содержат плейсхолдеры {{USER_NAME}}, {{REVIEW_BODY}}, {{RATING}} и т.д.
#      (НЕТ реальных имён/текстов — regex detect)
#   10. moderation.auto_filters содержит минимум ['profanity', 'spam_urls', 'pii_leak']
#   11. sla_hours в диапазоне 1-72
#   12. seeding_strategy.channels непусто, initial_volume_target в 10-100
#   13. incentive.value_placeholder — только плейсхолдеры (нет реальных сумм, regex:
#       не должно быть \d+\s*(?:руб|₽|%|баллов) вне {{...}})
#   14. Каждая механика имеет confidence ∈ {✓ПОДТВЕРЖДЕНО, ◆ОБОСНОВАНО, ◇ГИПОТЕЗА, ⚠РИСК}
#
# L3 (SEO):
#   15. При priority_engines=both: engine_tag = [Я] и [G] встречаются суммарно по механикам
#   16. Для commercial_local: business_profiles_sync.yandex_business.action_needed ≠ 'skip'
#       И google_business_profile.action_needed ≠ 'skip'
#   17. AggregateRating предполагает минимум 3 review-механики или collection_volume_target ≥ 30
#
# L4 (Семантический):
#   18. Cosine similarity ui_prompts.invite_text между механиками < 0.8 (не копия текстов)
#   19. Cosine similarity invite_text vs competitor_benchmarks < 0.85 (не скопировано)
#
# L5 (Интентный):
#   20. type mechanics ↔ content_type:
#       - faq → {qa} допустимо; {review, photo} не рекомендовано
#       - product → {review, rating, photo, qa}
#       - article → {comment, qa}
#       - landing → {review, rating, qa}
#
# L6 (Кросс-шаговый):
#   21. page_url ⊆ pages_to_process (из input)
#   22. Если video_strategy_from_12_2.has_video=true для страницы → ≥1 механика type='video_testimonial'
#       с tracking_id=ссылка на Шаг 12.2 tracking_id
#   23. Если existing_ugc.reviews_on_page>0 → механика type='review' учитывает это (seeding=наращивание)
#   24. handoff_to_13.schema_graph_per_page покрывает все pages с ugc_mechanics ≠ []
#   25. handoff_to_40_3 содержит по записи на каждый UGC tracking_id
#
# L7 (YMYL — особо строгий слой):
#   26. medical/legal → ВСЕ механики type ∈ {review, rating, photo} — ЗАПРЕТ на {qa, comment, forum, user_story}
#       при ymyl_restriction_level != 'full_allowed'
#   27. medical/legal → disclaimer_text непустой на каждой механике
#   28. medical/legal → moderation.type ∈ {'manual', 'hybrid'} (НЕ 'auto')
#   29. medical/legal → incentive.type ∈ {'recognition', 'loyalty_points', 'none'}
#       (ЗАПРЕТ 'discount' и 'gift' за отзывы — ФЗ-38, ст. 14.3 КоАП)
#   30. medical/legal → moderation.auto_filters содержит 'ymyl_banned_phrases'
#   31. moderation_global_policy.banned_phrases_ymyl содержит минимум:
#       ['лечит','гарантирует','излечение','100%','без побочных','мгновенно','самый эффективный']
#
# Веса и скоринг — см. следующую секцию QC CRITERIA.
# При провале критичных правил (фатальные: #1, #9, #13, #16, #26, #27, #29) → retry gate.
# При провале 3×: алерт Role A через Telegram + NocoDB review_queue.
```

---

## ██ КРИТЕРИИ QC

Rule-based. ML отключён до ≥200 размеченных образцов (политика CORE.md). Сумма весов = 100.

| **№** | **Проверка**                                                               | **Layer** | **Порог**    | **Вес** | **При провале** |
|-------|----------------------------------------------------------------------------|-----------|--------------|---------|-----------------|
| 1     | JSON Schema валиден                                                        | L1        | Pass         | 10      | Retry (gate)    |
| 2     | Уникальность mechanic_id / tracking_id                                     | L1        | 100%         | 3       | Retry           |
| 3     | Обязательные поля (summary, handoff_to_*, moderation_global_policy)        | L1        | 100%         | 4       | Retry           |
| 4     | ugc_enabled=false → все механики пусты + status="disabled"                 | L2        | 100%         | 3       | Auto-fix        |
| 5     | Соответствие mechanics_count content_type и правилам                       | L2        | ≥95%         | 3       | Retry           |
| 6     | Engine_tag на каждой механике                                              | L2        | 100%         | 5       | Retry           |
| 7     | Schema @type соответствует type механики                                   | L2        | 100%         | 6       | Retry           |
| 8     | Плейсхолдеры в schema-полях (НЕТ реальных имён/текстов)                    | L2        | 100%         | 6       | Фатал (юр./PII) |
| 9     | moderation.auto_filters содержит базовый набор                             | L2        | 100%         | 3       | Auto-fix        |
| 10    | seeding_strategy непуст + initial_volume_target в 10-100                   | L2        | 100%         | 3       | Retry           |
| 11    | Плейсхолдеры в incentive.value_placeholder (нет реальных сумм)             | L2        | 100%         | 4       | Фатал (реклама) |
| 12    | confidence на каждой механике                                              | L2        | 100%         | 2       | Retry           |
| 13    | При priority_engines=both: и [Я] и [G] встречаются                         | L3        | 100%         | 3       | Retry           |
| 14    | commercial_local/medical/legal → business_profiles_sync ≠ skip везде       | L3        | 100%         | 4       | Retry           |
| 15    | AggregateRating обоснован (≥3 review-механики или volume ≥ 30)             | L3        | 100%         | 2       | Auto-fix        |
| 16    | Cosine ui_prompts.invite_text между механиками < 0.8                       | L4        | ≥95%         | 2       | Review          |
| 17    | Cosine invite_text vs competitor_benchmarks < 0.85                         | L4        | 100%         | 2       | Retry           |
| 18    | type ↔ content_type mapping                                                | L5        | ≥90%         | 3       | Review          |
| 19    | page_url ⊆ pages_to_process                                                | L6        | 100%         | 2       | Фатал           |
| 20    | Если 12.2 has_video=true → механика video_testimonial с tracking_id        | L6        | 100%         | 3       | Retry           |
| 21    | handoff_to_13 покрывает все pages с mechanics≠∅                            | L6        | 100%         | 2       | Auto-fix        |
| 22    | handoff_to_40_3 содержит запись на каждый UGC tracking_id                  | L6        | 100%         | 2       | Auto-fix        |
| 23    | YMYL → механики ∈ {review, rating, photo} (запрет qa/comment/forum)        | L7        | 100%         | 6       | Фатал (юр.)     |
| 24    | YMYL → disclaimer_text непустой                                            | L7        | 100%         | 4       | Фатал           |
| 25    | YMYL → moderation.type ∈ {manual, hybrid}                                  | L7        | 100%         | 3       | Retry           |
| 26    | YMYL → incentive.type ∈ {recognition, loyalty_points, none}                | L7        | 100%         | 5       | Фатал (ФЗ-38)   |
| 27    | YMYL → moderation.auto_filters содержит 'ymyl_banned_phrases'              | L7        | 100%         | 3       | Retry           |
| 28    | moderation_global_policy.banned_phrases_ymyl заполнен (≥7 фраз)            | L7        | 100%         | 2       | Auto-fix        |
| **Итого** |                                                                        |           |              | **100** |                 |

**Скоринг:** ≥80 → передача в Шаги 13/19/26/29. 60–79 → `review_queue` в NocoDB + передача. <60 → Retry v2 по CORE (4 уровня).

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

| Нода | Действие                                                                                                      |
|------|---------------------------------------------------------------------------------------------------------------|
| 1    | Триггер: webhook от Шага 12.2 (событие `step_12_2.completed`)                                                 |
| 2a   | NocoDB: загрузка `PROJECT.md` (параллельно)                                                                   |
| 2b   | NocoDB: output Шага 12.2 (video_strategy + handoff_to_*) (параллельно)                                        |
| 2c   | NocoDB: output Шага 12 (content_blocks) + Шага 00 (personas) + Шага 05 (commercial_factors / orm) (параллельно) |
| 3    | Merge по `project_id` → сборка input contract для 12.3                                                        |
| 4    | Qdrant RAG: 4 коллекции (ugc_patterns, competitors, personas, legal_constraints)                              |
| 5    | Сборка промпта: CORE.md + PROJECT.md + STEP_12.3.md + input + RAG                                             |
| 6    | Claude Opus API (temperature 0.3, top_p 0.9, max_tokens 8192)                                                 |
| 7    | JSON парсер: strip markdown → `JSON.parse` → regex-extract fallback                                           |
| 8    | Execute `step_12_3_post_validate.py` (7 слоёв, 28 проверок)                                                   |
|      | ├─ qc ≥ 80 → Нода 9                                                                                           |
|      | ├─ 60 ≤ qc < 80 → Нода 9 + Telegram alert + review_queue insert                                               |
|      | └─ qc < 60 → Retry v2 (CORE): R1 инжекция ошибок → R2 декомпозиция per-page → R3 fallback-промпт → R4 алерт   |
| 9    | NocoDB: upsert в `ugc_strategies` (идемпотент по `(project_id, step_id, execution_id)`)                       |
| 10a  | Qdrant: upsert в `ugc_patterns` (только qc≥90) — параллельно с 10b                                            |
| 10b  | Grafana: push метрик (videos_generated, qc_score, ymyl_safe_ratio, mechanics_total) — non-blocking            |
| 11a  | Trigger Шаг 13 (Schema.org+OG), payload: `handoff_to_13` — параллельно с 11b/c/d                              |
| 11b  | Trigger Шаг 19 (Бизнес-профили), payload: `handoff_to_19` + `business_profiles_sync`                          |
| 11c  | Trigger Шаг 26 (ORM), payload: `handoff_to_26` + `moderation_global_policy`                                   |
| 11d  | Trigger Шаг 29 (AEO) + Шаг 40.3 (AI-цитирования), payload: `handoff_to_29` + `handoff_to_40_3`                |

**Обработка ошибок:**

- **Opus 429/500** → бэкофф 10с→30с→90с, max 3 попытки. Circuit Breaker: 3 подряд 5xx → halt + fallback на шаблоны из `ugc_patterns` RAG с `confidence="⚠РИСК"`.
- **Qdrant недоступен** → fallback на few-shot в system prompt, `qc_score -= 5`, WARN.
- **Невалидный JSON** → strip markdown → regex-extract → retry. Max 3 попытки.
- **Partial failures** (>30% страниц failed) → `status="partial"`, обработаны только успешные, Telegram alert.
- **Idempotency** — upsert-ключ `(project_id, step_id, execution_id)` на каждой NocoDB-операции.

---

## ██ ПРИМЕР FEW-SHOT

Сокращённый реалистичный пример для стоматологии (niche=dental, cluster=**medical** / YMYL, geo=Москва, engines=both, ugc_enabled=true). Консистентен с примерами Шагов 12.2 (та же ниша dental_moscow_001).

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "12.3",
  "prompt_version": "step12_3_v1.0",
  "timestamp": "2026-04-16T11:15:00Z",
  "status": "complete",
  "summary": {
    "pages_scanned": 1,
    "pages_with_mechanics": 1,
    "pages_ugc_disabled": 0,
    "mechanics_total": 3,
    "mechanic_types_distribution": {"review": 1, "rating": 1, "photo": 1, "video_testimonial": 0, "qa": 0, "comment": 0, "forum": 0, "user_story": 0},
    "ymyl_pages_with_safe_ugc": 1,
    "ymyl_pages_with_disabled_qa": 1,
    "business_profiles_sync_needed": true,
    "total_seeding_volume_target_monthly": 15
  },
  "pages": [
    {
      "page_url": "https://dentclinic.ru/services/implantaciya/",
      "content_type": "landing",
      "ugc_strategy": {
        "mechanics_count": 3,
        "primary_mechanic": "review",
        "ymyl_restriction_level": "reviews_only",
        "rationale": "Medical/YMYL landing. Разрешены только отзывы о сервисе, рейтинг, фото интерьера/результатов (с согласием). Запрещены Q&A и форум — риск советов между пациентами. Обязательны дисклеймер и ручная модерация.",
        "confidence": "◆ОБОСНОВАНО",
        "data_sources": ["project_context", "persona_analysis", "rag:legal_constraints", "competitor_benchmark"]
      },
      "ugc_mechanics": [
        {
          "mechanic_id": "UGC001",
          "tracking_id": "UGC-3a8b2c-dental-implant-review",
          "type": "review",
          "placement": "mid_content",
          "engine_tag": "[ОБА]",
          "target_queries": [
            {"query": "имплантация зубов москва отзывы", "engine": "both"},
            {"query": "отзывы о клинике имплантации", "engine": "yandex"}
          ],
          "confidence": "◆ОБОСНОВАНО",
          "schema_markup": {
            "@context": "https://schema.org",
            "@type": "Review",
            "fields_template": {
              "author": {"@type": "Person", "name": "{{USER_NAME}}"},
              "datePublished": "{{REVIEW_DATE}}",
              "reviewBody": "{{REVIEW_BODY}}",
              "reviewRating": {"@type": "Rating", "ratingValue": "{{RATING}}", "bestRating": 5},
              "itemReviewed": {"@type": "MedicalBusiness", "name": "{{CLINIC_NAME}}"}
            }
          },
          "moderation": {
            "type": "manual",
            "auto_filters": ["profanity", "spam_urls", "competitor_mentions", "ymyl_banned_phrases", "pii_leak"],
            "sla_hours": 24,
            "escalation_rules": "Любое упоминание осложнений, диагнозов, ФИО врача без согласия, юр. угроз — ручная модерация Role A + юрист клиники."
          },
          "incentive": {
            "type": "loyalty_points",
            "value_placeholder": "{{LOYALTY_POINTS}} баллов лояльности за любой честный отзыв (положительный или отрицательный)",
            "legal_compliance_note": "Стимул за ЛЮБОЙ отзыв (не только положительный) — соответствует ЗоЗПП и ФЗ-38. Запрещена привязка бонуса к оценке. Ст. 14.3 КоАП не нарушается."
          },
          "ui_prompts": {
            "invite_text": "Поделитесь вашим опытом имплантации в нашей клинике. Ваш отзыв поможет другим пациентам сделать информированный выбор.",
            "form_fields": [
              {"field_name": "rating", "type": "rating", "required": true, "max_length": null},
              {"field_name": "review_body", "type": "text", "required": true, "max_length": 2000},
              {"field_name": "consent_publication", "type": "select", "required": true, "max_length": null}
            ],
            "confirmation_text": "Спасибо! Ваш отзыв отправлен на модерацию. Публикация после проверки (в течение 24 часов).",
            "anti_spam_hidden_field": true
          },
          "seeding_strategy": {
            "description": "Импорт верифицированных отзывов из Я.Бизнес и GBP + приглашение после консультации (email/SMS через 7 дней)",
            "initial_volume_target": 15,
            "channels": ["business_profile_import", "post_consultation_form"],
            "timeline_days": 45
          },
          "disclaimer_text": "Отзыв отражает личный опыт пациента и не является медицинской рекомендацией. Требуется очная консультация врача.",
          "collection_volume_target_monthly": 8,
          "engagement_lift_estimate": {
            "metric": "dwell_time_pct",
            "estimated_value": 35,
            "confidence": "◆ОБОСНОВАНО"
          },
          "ai_citation_potential": "high",
          "data_sources": ["rag:ugc_patterns", "competitor_benchmark", "persona_analysis"]
        },
        {
          "mechanic_id": "UGC002",
          "tracking_id": "UGC-3a8b2c-dental-implant-rating",
          "type": "rating",
          "placement": "above_footer",
          "engine_tag": "[ОБА]",
          "target_queries": [{"query": "имплантация зубов москва рейтинг", "engine": "both"}],
          "confidence": "✓ПОДТВЕРЖДЕНО",
          "schema_markup": {
            "@context": "https://schema.org",
            "@type": "AggregateRating",
            "fields_template": {
              "ratingValue": "{{AVG_RATING}}",
              "reviewCount": "{{REVIEW_COUNT}}",
              "bestRating": 5,
              "itemReviewed": {"@type": "MedicalBusiness", "name": "{{CLINIC_NAME}}"}
            }
          },
          "moderation": {
            "type": "auto",
            "auto_filters": ["spam_urls", "pii_leak"],
            "sla_hours": 1,
            "escalation_rules": "Аномальные рейтинги (всплеск 5-звёзд за 1 день >20) — подозрение на накрутку, ручная проверка."
          },
          "incentive": {
            "type": "none",
            "value_placeholder": null,
            "legal_compliance_note": "Рейтинг рассчитывается агрегировано из механики UGC001. Отдельный стимул не предусмотрен."
          },
          "ui_prompts": {
            "invite_text": "Средняя оценка пациентов отображается агрегировано по отзывам.",
            "form_fields": [],
            "confirmation_text": "",
            "anti_spam_hidden_field": false
          },
          "seeding_strategy": {
            "description": "Агрегация из UGC001 + импорт рейтингов Я.Бизнес/GBP",
            "initial_volume_target": 15,
            "channels": ["business_profile_import"],
            "timeline_days": 45
          },
          "disclaimer_text": "Рейтинг отражает усреднённую оценку пациентами качества сервиса, не медицинских результатов.",
          "collection_volume_target_monthly": 8,
          "engagement_lift_estimate": {"metric": "ctr_lift_pct", "estimated_value": 12, "confidence": "◆ОБОСНОВАНО"},
          "ai_citation_potential": "high",
          "data_sources": ["rag:ugc_patterns", "aggregation_from_ugc001"]
        },
        {
          "mechanic_id": "UGC003",
          "tracking_id": "UGC-3a8b2c-dental-implant-photo",
          "type": "photo",
          "placement": "dedicated_section",
          "engine_tag": "[Я]",
          "target_queries": [{"query": "имплантация зубов до после", "engine": "yandex"}],
          "confidence": "◇ГИПОТЕЗА",
          "schema_markup": {
            "@context": "https://schema.org",
            "@type": "ImageObject",
            "fields_template": {
              "contentUrl": "{{USER_PHOTO_URL}}",
              "author": {"@type": "Person", "name": "{{USER_NAME_OR_ANONYMOUS}}"},
              "uploadDate": "{{UPLOAD_DATE}}",
              "caption": "{{CAPTION}}",
              "representativeOfPage": true
            }
          },
          "moderation": {
            "type": "manual",
            "auto_filters": ["profanity", "ymyl_banned_phrases", "pii_leak", "face_identification"],
            "sla_hours": 48,
            "escalation_rules": "Любое фото с идентифицируемым лицом без подтверждённого согласия — запрет публикации. Фото с признаками диагноза/патологии без сопровождающего дисклеймера — модерация юриста клиники."
          },
          "incentive": {
            "type": "recognition",
            "value_placeholder": "Благодарность в профиле клиента и публичная отметка (с согласия)",
            "legal_compliance_note": "Без денежных/материальных стимулов. Письменное согласие на публикацию обязательно (ФЗ-152 «О персональных данных»)."
          },
          "ui_prompts": {
            "invite_text": "С вашего согласия мы готовы разместить фото результата в галерее. Фото анонимизируется (без идентифицируемого лица) если не указано иное.",
            "form_fields": [
              {"field_name": "photo_upload", "type": "photo_upload", "required": true, "max_length": null},
              {"field_name": "consent_full", "type": "select", "required": true, "max_length": null},
              {"field_name": "caption", "type": "text", "required": false, "max_length": 200}
            ],
            "confirmation_text": "Спасибо. Фото отправлено на модерацию (до 48 часов). Публикация только после подтверждения согласия.",
            "anti_spam_hidden_field": true
          },
          "seeding_strategy": {
            "description": "Запрос у постоянных пациентов после завершения имплантации (через 3 мес) с оффлайн-подписанием согласия",
            "initial_volume_target": 10,
            "channels": ["post_consultation_form"],
            "timeline_days": 90
          },
          "disclaimer_text": "Фото публикуются с явного согласия пациента. Результаты индивидуальны и не гарантированы. Требуется очная консультация врача для оценки показаний.",
          "collection_volume_target_monthly": 3,
          "engagement_lift_estimate": {"metric": "dwell_time_pct", "estimated_value": 22, "confidence": "◇ГИПОТЕЗА"},
          "ai_citation_potential": "medium",
          "data_sources": ["rag:ugc_patterns", "persona_analysis"]
        }
      ],
      "aeo_elements": {
        "enables_rich_results": ["Review", "AggregateRating", "ImageObject"],
        "ugc_citation_potential": "high",
        "integrates_with_video_12_2": false,
        "hook_for_ai_citation": "AggregateRating + 15+ Review-цитат повышают вероятность цитирования клиники в Яндекс Нейро и Google AI Mode на запросы commercial intent типа «лучшая клиника имплантации Москва»."
      },
      "engine_tags": {
        "yandex_specific": ["Импорт отзывов из Я.Бизнес", "Привязка к Я.Карты", "Я.Услуги-отзывы", "Рейтинг в Яндекс Нейро через AggregateRating"],
        "google_specific": ["Импорт отзывов из Google Business Profile", "Review schema для SERP-звёзд", "Потенциал цитирования в AI Mode через Review snippets"]
      }
    }
  ],
  "business_profiles_sync": {
    "yandex_business": {
      "action_needed": "optimize",
      "review_import_strategy": "Ежедневный cron-импорт новых отзывов (Я.Бизнес API) в NocoDB + показ на сайте через Review schema",
      "photo_sync_from_ugc": true,
      "priority_score": 9
    },
    "google_business_profile": {
      "action_needed": "optimize",
      "review_import_strategy": "Еженедельный импорт через GBP API + дедупликация vs Я.Бизнес",
      "qa_seeding_plan": "ЗАПРЕЩЕНО для medical — только ответы клиники на общие вопросы о сервисе (режим работы, цены), БЕЗ медицинских советов",
      "priority_score": 8
    },
    "2gis": {"action_needed": "update", "priority_score": 6}
  },
  "moderation_global_policy": {
    "banned_phrases_common": ["лучший в мире", "100% успех", "не имеет аналогов"],
    "banned_phrases_ymyl": ["лечит", "гарантирует", "излечение", "100%", "без побочных", "мгновенно", "самый эффективный"],
    "pii_patterns_to_strip": ["phone", "email", "passport_number", "insurance_number", "medical_record_number"],
    "escalation_triggers": ["negative_sentiment_with_accusation", "legal_threat_mention", "malpractice_claim", "competitor_explicit_mention"],
    "response_templates": [
      {"trigger": "negative_review", "template_placeholder": "Спасибо за обратную связь, {{USER_NAME}}. Свяжитесь с нами для разбора ситуации: {{CONTACT_PHONE}}"},
      {"trigger": "question_unanswered", "template_placeholder": "Благодарим за вопрос. Для ответа по вашему клиническому случаю необходима очная консультация: {{BOOKING_URL}}"}
    ]
  },
  "handoff_to_13": {
    "schema_graph_per_page": [
      {"page_url": "https://dentclinic.ru/services/implantaciya/", "graph_additions": ["Review", "AggregateRating", "ImageObject"]}
    ]
  },
  "handoff_to_19": {
    "business_profile_actions": [
      {"platform": "yandex_business", "action": "Настроить регулярный импорт отзывов + загрузка фото-галереи из UGC003 (после модерации)", "priority": "high"},
      {"platform": "gbp", "action": "Включить Reviews API + настроить шаблонные ответы на отзывы", "priority": "high"},
      {"platform": "2gis", "action": "Синхронизировать рейтинг и 5 последних отзывов", "priority": "medium"}
    ]
  },
  "handoff_to_26": {
    "orm_triggers_to_monitor": [
      {"trigger_type": "negative_review_sentiment", "page_url": "https://dentclinic.ru/services/implantaciya/", "alert_threshold": "3 негативных отзыва за 7 дней"},
      {"trigger_type": "legal_threat_keyword", "page_url": "https://dentclinic.ru/services/implantaciya/", "alert_threshold": "1 упоминание"},
      {"trigger_type": "competitor_mention", "page_url": "https://dentclinic.ru/services/implantaciya/", "alert_threshold": "2 упоминания за 30 дней"}
    ]
  },
  "handoff_to_29": {
    "ugc_ai_citation_targets": [
      {"page_url": "https://dentclinic.ru/services/implantaciya/", "expected_ai_citation_format": "quote+aggregate_rating", "target_queries": ["имплантация зубов москва отзывы", "лучшая клиника имплантации"]}
    ]
  },
  "handoff_to_40_3": [
    {"ugc_tracking_id": "UGC-3a8b2c-dental-implant-review", "page_url": "https://dentclinic.ru/services/implantaciya/", "mechanic_type": "review", "expected_citation_queries": ["имплантация зубов москва отзывы", "отзывы о клинике имплантации"]},
    {"ugc_tracking_id": "UGC-3a8b2c-dental-implant-rating", "page_url": "https://dentclinic.ru/services/implantaciya/", "mechanic_type": "rating", "expected_citation_queries": ["имплантация зубов москва рейтинг"]},
    {"ugc_tracking_id": "UGC-3a8b2c-dental-implant-photo", "page_url": "https://dentclinic.ru/services/implantaciya/", "mechanic_type": "photo", "expected_citation_queries": ["имплантация зубов до после"]}
  ],
  "failed_pages": [],
  "qc_score": 100,
  "qc_breakdown": {"json_schema":10,"id_unique":3,"required_fields":4,"ugc_disabled_logic":3,"mechanics_count_match":3,"engine_tag":5,"schema_type_match":6,"placeholder_detect":6,"moderation_filters":3,"seeding_volume":3,"incentive_placeholders":4,"confidence":2,"both_coverage":3,"business_profiles":4,"aggregate_rating":2,"cosine_between":2,"cosine_competitors":2,"type_content_type":3,"url_subset":2,"video_testimonial_sync":3,"handoff_13_coverage":2,"handoff_40_3_records":2,"ymyl_allowed_types":6,"ymyl_disclaimer":4,"ymyl_moderation_type":3,"ymyl_incentive":5,"ymyl_banned_filter":3,"ymyl_banned_global":2}
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА (Retry v2 при 3× QC fail)

| **Откат**  | **Действие**                                                                                                                                              |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Retry 1** | Инжекция ошибок QC в промпт → только failing subset (per-page). Температура та же (0.3).                                                                 |
| **Retry 2** | Декомпозиция на 2 вызова: (1) стратегия + типы механик + placement; (2) schema_markup + moderation + ui_prompts + seeding. Merge на стороне N8N.         |
| **Retry 3** | Fallback-промпт строгий: few-shot из `ugc_patterns` (Qdrant, qc≥90). Temperature=0.15. Упрощённый каркас (1 механика на страницу).                       |
| **Retry 4** | Алерт Role A через Telegram + NocoDB review_queue. Пауза пайплайна. Лог 3 попыток. Переход в ручной режим (исключение 12.3 из цепочки).                   |
| **Откат A** — деградация       | Шаблон из `ugc_patterns` RAG, `confidence="⚠РИСК"`, review_queue. Для всех страниц.                                                              |
| **Откат B** — single-mechanic  | Все страницы → mechanics_count=1 (primary=review). Упрощённый output. Снимает 40% сложности.                                                     |
| **Откат C** — YMYL-safe-only   | Для YMYL-проекта при повторном провале: ВСЕ механики → только type='review' с ручной модерацией. `ymyl_restriction_level='reviews_only'`.        |
| **Откат D** — ugc_disabled     | Если входной флаг `ugc_enabled=false` → ВСЕ страницы с `ugc_mechanics=[]`, `status="disabled"`, прокидывается `summary.pages_ugc_disabled`.      |

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер                  | Приоритетные механики                              | Placement                  | Moderation      | Incentive                        | Особенности                                                                                             |
|--------------------------|----------------------------------------------------|----------------------------|-----------------|----------------------------------|---------------------------------------------------------------------------------------------------------|
| **commercial_local**     | review, rating, photo                              | above_footer, dedicated    | hybrid          | loyalty_points, recognition      | Я.Бизнес-импорт критичен; привязка к Я.Карты/2GIS; фото-отзывы от клиентов                              |
| **commercial_national**  | review, rating, user_story                         | mid_content, dedicated     | hybrid          | loyalty_points, early_access     | Case studies как user_story; интеграция с Я.Дзен/медиа                                                  |
| **ecom**                 | review, rating, photo, qa                          | product_card, mid_content  | auto+manual     | discount, loyalty_points         | Review schema в Product; Q&A для товаров; фото от клиентов повышают конверсию на 15-30%                 |
| **saas**                 | review, user_story, forum, qa                      | dedicated, sidebar         | hybrid          | early_access, recognition        | Community-форум как отдельный sub-domain; G2/Capterra-интеграция; international → многоязычные UGC      |
| **info**                 | comment, qa                                        | mid_content, end_of_article | auto           | recognition                      | Комментарии к статьям; Q&A расширяет long-tail; интеграция с Я.Дзен                                     |
| **medical** (YMYL)       | **ТОЛЬКО**: review, rating, photo                  | mid_content, dedicated     | **manual**      | **loyalty_points, recognition**  | ЗАПРЕТ qa/comment/forum; disclaimer обязателен; ФЗ-38; ст. 14.3 КоАП; фото только с согласием, без PII  |
| **legal** (YMYL)         | **ТОЛЬКО**: review, rating                         | above_footer               | **manual**      | **recognition, none**            | ЗАПРЕТ qa/forum/user_story с юр. советами; disclaimer «не заменяет консультацию»                       |
| **education**            | review, user_story, comment, qa                    | mid_content, dedicated     | hybrid          | recognition, early_access        | Отзывы студентов + user_stories об успехах; сезонность (август-сентябрь, май); Course schema            |

---

## ██ МЕТРИКИ УСПЕХА

### Опережающие

- Покрытие страниц с `ugc_enabled=true` UGC-механиками — ≥95%.
- qc_score средний по батчу ≥ 80.
- Валидные Schema.org markup (Review/FAQPage/AggregateRating) — 100% через Rich Results Test.
- 0 выдуманных имён/текстов отзывов в schema (плейсхолдер-детект).
- YMYL: 100% страниц с `disclaimer_text` + ручной модерацией.
- YMYL: 0 механик типа `qa`/`forum`/`user_story` в `medical`/`legal` без `ymyl_restriction_level='full_allowed'`.

### Запаздывающие (30–90 дней)

- Фактический объём собранных UGC vs `collection_volume_target_monthly` (≥70%).
- Доля страниц с Review schema в Rich Results SERP (Google).
- Средний рейтинг в Я.Бизнес/GBP (рост ≥0.3 за 90 дней).
- Dwell time на страницах с UGC vs без UGC (A/B) — лифт ≥20%.
- Цитирования UGC в AI-ответах (Яндекс Нейро + Google AI Mode) через Шаг 40.3.
- Рост long-tail запросов через Q&A-механики (для не-YMYL).

### Контрольные (квартальные)

- ROI UGC: (рост конверсии × средний чек) / затраты на модерацию + incentive.
- ORM-алерты (Шаг 26): <5 критичных триггеров в квартал.
- 0 юридических инцидентов (штрафы по ФЗ-38, КоАП, ЗоЗПП) — особенно для medical/legal.

### Анти-метрики

- Жалобы на использование реальных ФИО без согласия — **0**.
- UGC-спам в production (после auto-фильтров) — **<1% от объёма**.
- YMYL banned phrases в опубликованных UGC — **0** (ручная модерация гарантирует).

**Критерий «шаг выполнен хорошо»:** ≥90% страниц с `ugc_enabled=true` получили `qc_score≥80`; все Review/AggregateRating прошли Rich Results Test; `handoff_to_13/19/26/29/40.3` заполнены корректно; для YMYL 100% безопасная конфигурация.

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия**   | **Дата**    | **Изменения**                                                                                                                                                                                                                    |
|--------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| v1.0         | 2026-04-16  | Первая версия шага ★ новый в v6.0. Шаблон 3 (Генерация контента). Тип = Промпт. 8 типов UGC-механик (review, qa, comment, photo, video_testimonial, rating, forum, user_story). 7-слойный QC (28 проверок), YMYL-защита с самого начала, anti-hallucination плейсхолдеры, handoff_to_13/19/26/29/40.3, business_profiles_sync для Я.Бизнес/GBP/2GIS, moderation_global_policy. Учтены уроки аудита Шага 12.2 (confidence, data_sources, tracking_id, status, failed_pages). |
| v1.1 *(план)* | Q3 2026     | Интеграция с Шагом 40.3: обратная связь — если UGC цитируется в AI-ответах, повышение приоритета данного типа механики для похожих проектов. Добавить `ugc_performance_feedback` в input contract.                               |
| v1.2 *(план)* | Q4 2026     | Поддержка international=true: мульти-язычные UGC, локализация ui_prompts, разделение Review schema по `inLanguage`.                                                                                                              |
| v2.0 *(план)* | 2027 Q1     | ML-классификатор UGC-токсичности (после накопления ≥200 размеченных образцов). Снятие политики «rule-based only» из CORE.md для этого слоя модерации.                                                                             |

---

*Конец STEP_12.3.md. Общий объём: ~4K токенов. Следующие потребители: STEP_13.md (Schema.org+OG), STEP_19.md (Бизнес-профили), STEP_26.md (ORM), STEP_27.md (Блог/Контент-хаб), STEP_29.md (AEO), STEP_40.3.md (AI-цитирования).*
