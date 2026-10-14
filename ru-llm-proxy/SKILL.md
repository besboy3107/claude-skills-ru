---
name: ru-llm-proxy
description: "CRITICAL: Whenever writing any code that calls LLM APIs (OpenAI, OpenRouter, Groq, Anthropic, Telegram) on a Russian server (any RU IP), always add SOCKS5 proxy via localhost:10801. Without proxy all LLM requests return 401/403. Triggers on: writing Python/Node.js AI code, adding API calls, creating .env files, setting up Telegram bots, deploying on Russian hosting."
---

# RU LLM Proxy - ОБЯЗАТЕЛЬНЫЙ ПАТТЕРН

Серверы в России заблокированы у OpenAI, Groq, OpenRouter, Anthropic API.
**Без SOCKS proxy все LLM-запросы вернут 401/403 - ВСЕГДА.**

Это касается любого российского хостинга: Timeweb, Selectel, Beget, Serverspace, и т.д.

## Как настроить SOCKS туннель

На своей машине (или jump-сервере вне RU):

```bash
# Открыть SOCKS5 туннель через SSH
ssh -D 10801 -f -C -q -N -p <YOUR_SSH_PORT> user@<YOUR_RELAY_SERVER_IP>
# Туннель будет доступен на localhost:10801 на сервере
```

Или используй OpenVPN / WireGuard туннель на выходной нод вне RU.

## .env - обязательные 5 переменных

```env
HTTP_PROXY=socks5://127.0.0.1:10801
HTTPS_PROXY=socks5://127.0.0.1:10801
ALL_PROXY=socks5://127.0.0.1:10801
http_proxy=socks5://127.0.0.1:10801
https_proxy=socks5://127.0.0.1:10801
```

## Python - httpx с proxy

```python
import httpx
import os

proxy = os.getenv("HTTPS_PROXY", "socks5://127.0.0.1:10801")

# OpenAI / OpenRouter / Groq
from openai import AsyncOpenAI
client = AsyncOpenAI(
    api_key=os.getenv("OPENROUTER_API_KEY"),
    base_url="https://openrouter.ai/api/v1",
    http_client=httpx.AsyncClient(proxy=proxy)
)

# Прямой запрос
async with httpx.AsyncClient(proxy=proxy) as client:
    resp = await client.post(url, json=payload, headers=headers)
```

## Node.js - ProxyAgent

```js
import { ProxyAgent } from 'undici'
import OpenAI from 'openai'

const proxy = process.env.HTTPS_PROXY || 'socks5://127.0.0.1:10801'
const dispatcher = new ProxyAgent(proxy)

const client = new OpenAI({
  apiKey: process.env.OPENROUTER_API_KEY,
  baseURL: 'https://openrouter.ai/api/v1',
  fetchOptions: { dispatcher }
})
```

## Запуск сервиса - node --env-file

```bash
# ОБЯЗАТЕЛЬНО через --env-file, иначе proxy переменные не подхватятся
node --env-file=.env index.js
```

Если используешь старый Node.js без `--env-file`:

```js
import 'dotenv/config'  // должен быть первым импортом
```

## Groq -> OpenRouter фолбэк

```python
# Основной: Groq (бесплатный быстрый инференс)
GROQ_API_KEY=gsk_...
# Фолбэк: OpenRouter free tier
OPENROUTER_API_KEY=sk-or-v1-...

# В коде:
try:
    response = await groq_client.chat.completions.create(...)
except Exception:
    response = await openrouter_client.chat.completions.create(...)
```

## Что НЕЛЬЗЯ делать

- Удалять proxy переменные при обновлении инфраструктуры
- Использовать голый `fetch`/`axios` без ProxyAgent в Node.js (`undici` игнорирует HTTP_PROXY)
- Запускать сервис без `--env-file` или без явного `dotenv.config()`
- Считать, что "у меня работало локально" = работает на RU сервере

## Проверка работоспособности proxy

```bash
# Проверить, доступен ли OpenAI через proxy
curl -x socks5://127.0.0.1:10801 https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# Проверить IP выхода
curl -x socks5://127.0.0.1:10801 https://ifconfig.me
```

## Альтернативы SOCKS: прямой прокси через OpenVPN

Если VPN-туннель поднят и трафик идёт через него автоматически - переменные `*_PROXY` не нужны.
Проверить: `curl https://ifconfig.me` должен вернуть IP вне России.
