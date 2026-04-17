🏭 Проект: MachCalc — AI-калькулятор металлообработки (B2B)
> Последнее обновление: 16 апреля 2026
---
🎯 Суть проекта
Калькулятор встроен в сайт metalhub.tech как страница `calculator.html`.
Сценарий "для даунов":
Загрузил чертежи (PDF/PNG/JPG, несколько файлов или drag & drop)
Написал задачу в текстовом поле (опционально)
Указал количество и срочность
Нажал — AI разобрал чертёж, подставил рыночные цены, выдал стоимость
При желании — развернул ползунки и скорректировал параметры вручную
Целевая аудитория: технологи и экономисты производств (B2B, не B2C).
---
✅ Что сделано и работает
Инфраструктура
Cloudflare Pages: `machcalc.pages.dev` — задеплоен ✅
Cloudflare Worker: `machcalc-worker.kcoprofi.workers.dev` — живой ✅
Cloudflare KV: namespace `PRICES_KV` создан, привязан к Worker ✅
`/api/health` возвращает `{"ok":true,"nds_rate":0.22,"kv_available":true}` ✅
`/api/prices` возвращает все 70 марок из таблицы ✅
`/api/prices?grade=40Х` возвращает цену (KV кеш → AI поиск → таблица) ✅
Anthropic API ключ: добавлен через Settings → Variables and Secrets ✅
`_redirects`: `/api/* https://machcalc-worker.kcoprofi.workers.dev/:splat 200` ✅
Фронтенд — `calculator.html`
Полностью в стиле metalhub.tech — Inter, синий `#1783ff`, blur-header, noise, radius 22px ✅
Хедер и футер идентичны остальным страницам сайта ✅
Drag & drop + мультизагрузка файлов (PDF, PNG, JPG) ✅
Текстовое поле задачи — AI считает по описанию если нет чертежа ✅
Ползунки скрыты под "Настроить параметры" — видны только по желанию ✅
НДС 22% · НК РФ · с 2025 г. — везде явно, ставка 20% упразднена ✅
Файл лежит в корне metalhub.tech рядом с system.html, thinking.html ✅
Worker — `machcalc/worker/src/index.js` (v2)
`POST /api/analyze` — файл → AI-разбор → JSON + автоподтягивание цены материала ✅
`GET /api/prices` — все 70 марок из таблицы ✅
`GET /api/prices?grade=МАРКА` — KV кеш (24ч) → AI web_search → таблица ✅
`GET /api/health` — статус + kv_available ✅
CORS настроен ✅
Таблица цен на металл (70 марок, апрель 2026)
9 категорий: углеродистые, легированные, инструментальные, жаропрочные,
нержавеющие, алюминиевые, титановые, медь/бронза, чугун.
Обновлять вручную раз в квартал.
Cost engine (v1)
```
материал + поковка
→ токарная + расточка + ТО + УЗК
→ × (1 + накладные%)
→ / (1 − маржа%)
→ × срочность
→ цена без НДС → + 22% НДС → итог с НДС
```
Сроки в рабочих днях: поковка / мехобработка / ТО / ОТК.
НДС 22% · НК РФ · с 2025 г. Ставка 20% упразднена.
Дизайн-система (из metalhub.tech)
```css
--bg: #0a0c10;  --accent: #1783ff;  --radius: 22px;  font-family: Inter;
--card: rgba(255,255,255,0.05);  --line: rgba(255,255,255,0.12);
```
---
🗂 Структура файлов
```
metalhub.tech/                 ← сайт (Cloudflare Pages)
├── index.html
├── system.html
├── thinking.html
├── calculator.html            ← калькулятор (наш)
├── styles.css
├── _redirects                 ← /api/* → Worker
└── assets/

machcalc/                      ← Worker
├── worker/
│   ├── wrangler.toml          ← с KV binding PRICES_KV
│   └── src/index.js           ← v2: /api/prices + /api/analyze + /api/health
└── docs/STATUS.md
```
---
⚠️ Известные проблемы
#	Проблема	Приоритет
1	Загрузка с мобильного Safari — "The string did not match the expected pattern"	высокий
2	Цены подтягиваются в Worker, но не подставляются в ползунок на фронте	в работе
3	Несколько чертежей — в AI уходит только первый файл	средний
4	D1 не подключена — нет истории расчётов	низкий
5	Нет авторизации организации	низкий
---
🔨 Следующий шаг — автоподстановка цены в calculator.html
Worker уже возвращает цену в `_price.price_per_kg` внутри ответа `/api/analyze`.
Нужно в `calculator.html` после получения ответа AI:
Взять `data._price.price_per_kg`
Подставить в ползунок `steelPrice`
Показать пользователю откуда цена (источник + дата)
Пересчитать результат
---
📐 Полная архитектура
```
metalhub.tech (Cloudflare Pages)
    ↓ /api/* → Worker
Cloudflare Worker (machcalc-worker.kcoprofi.workers.dev)
    ├── POST /api/analyze   → Anthropic API → разбор чертежа + цена материала
    ├── GET  /api/prices    → 70 марок из таблицы
    ├── GET  /api/prices?grade=X → KV кеш → AI web_search → таблица
    └── GET  /api/health    → статус

Cloudflare KV (PRICES_KV)  → кеш цен TTL 24ч ✅
Cloudflare D1              → история расчётов (будущее)
Cloudflare R2              → хранение чертежей (будущее)
```
---
💡 В очереди
Автоподстановка цены в calculator.html ← следующий шаг
Починить Safari мобильный
Несколько чертежей → несколько позиций в расчёте
История расчётов в D1
Авторизация: по ИНН + телефон или email домена
Экспорт в PDF для отправки клиенту
---
🗓 Лог сессий
Сессия 1 — 13 апреля 2026
Разобрали экономику, выбрали стек, обрезали ТЗ до MVP, создан STATUS.md
Сессия 2 — 16 апреля 2026
Сдвиг B2C → B2B, все ползунки открыты
Промпт AI-разбора + тест на реальном чертеже (Втулка цапфы, confidence 95%)
Cost engine v1 с НДС 22% (ставка 20% упразднена)
Cloudflare Pages + Worker задеплоены через Dashboard (без wrangler)
API ключ добавлен через Variables and Secrets
_redirects исправлен: одна строка
Редизайн под metalhub.tech, calculator.html размещён на сайте
Концепция UX финализирована: загрузил + описал + количество → цена
Ползунки скрыты, разворачиваются по желанию
Worker v2: добавлен /api/prices с таблицей 70 марок
Логика: KV кеш 24ч → AI web_search → резервная таблица
Cloudflare KV namespace PRICES_KV создан и привязан к Worker через Dashboard
Все три эндпоинта проверены и работают ✅
Следующий шаг: автоподстановка цены материала в calculator.html
