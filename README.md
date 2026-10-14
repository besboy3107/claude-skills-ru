# Claude Code Skills для российских разработчиков

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-blue)](https://claude.ai/code)

Готовые скиллы для [Claude Code](https://claude.ai/code) — решают типичные проблемы при разработке в России: блокировки API, Telegram боты, Яндекс.Директ.

## Что такое скиллы Claude Code?

Скиллы — это файлы `SKILL.md` в папке `~/.claude/skills/`. Claude автоматически применяет их контекст при работе над задачами. Не нужно каждый раз объяснять как настроить прокси или подключить Telegram бота — Claude уже знает.

## Скиллы в этом репо

### 🔌 ru-llm-proxy — LLM API через прокси

**Проблема:** OpenAI, Groq, OpenRouter, Anthropic блокируют российские IP. Все запросы падают с 401/403.

**Решение:** SOCKS5 прокси через SSH туннель или VPN. Скилл учит Claude всегда добавлять прокси при написании AI-кода для RU серверов.

**Покрывает:**
- Python (httpx + AsyncOpenAI)
- Node.js (undici ProxyAgent)
- .env конфиг (5 переменных)
- Groq → OpenRouter фолбэк

---

### 📨 telegram-ru-proxy — Telegram боты на RU сервере

**Проблема:** `fetch` и `axios` к `api.telegram.org` молча падают с российских серверов. Сообщения не доходят, ошибок нет.

**Решение:** `telegraf` + `socks-proxy-agent`. Скилл учит Claude всегда использовать правильный паттерн для Telegram ботов на RU хостинге.

**Покрывает:**
- Node.js (telegraf + SocksProxyAgent)
- Отправка уведомлений без telegraf
- Python (python-telegram-bot + HTTPXRequest)
- Webhook vs Polling на RU сервере

---

### 📊 yandex-direct-ru — Яндекс.Директ и Метрика

**Контекст:** Лучшие практики контекстной рекламы в российском рынке.

**Покрывает:**
- Структура кампаний (Поиск / РСЯ / МК)
- Настройка целей в Метрике
- Диагностика "клики есть, конверсий нет"
- Стратегии ставок
- Минус-слова (универсальный список)
- Промо-статья как альтернатива ПромоСтраницам

---

## Установка

```bash
# 1. Клонировать репо
git clone https://github.com/besboy3107/claude-skills-ru.git

# 2. Скопировать нужные скиллы в папку Claude
cp -r claude-skills-ru/ru-llm-proxy ~/.claude/skills/
cp -r claude-skills-ru/telegram-ru-proxy ~/.claude/skills/
cp -r claude-skills-ru/yandex-direct-ru ~/.claude/skills/
```

### Для Windows

```powershell
# Клонировать
git clone https://github.com/besboy3107/claude-skills-ru.git

# Скопировать (PowerShell)
Copy-Item -Recurse claude-skills-ru\ru-llm-proxy "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse claude-skills-ru\telegram-ru-proxy "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse claude-skills-ru\yandex-direct-ru "$env:USERPROFILE\.claude\skills\"
```

После копирования Claude Code автоматически подхватит скиллы в следующей сессии.

## Как это работает

Claude Code читает все `SKILL.md` файлы из `~/.claude/skills/` при старте сессии. Когда ты пишешь код с LLM API или Telegram — скилл автоматически включается и Claude знает правильные паттерны.

**Пример:** Напишешь Claude "добавь отправку уведомлений в Telegram" — и он сразу использует `socks-proxy-agent` без лишних объяснений.

## Структура скилла

```
skill-name/
└── SKILL.md    # Описание + примеры кода
```

SKILL.md содержит:
- Frontmatter с `name` и `description` (тригеры для автоподключения)
- Описание проблемы
- Готовые паттерны кода
- Что нельзя делать

## Вклад в проект

Если у тебя есть скилл для типичных российских задач — PR приветствуется. Требования:
- Никаких личных данных (IP, токены, домены, имена)
- Только универсальные паттерны
- Русский язык для описания, английский для кода

## Лицензия

MIT — используй как хочешь.
