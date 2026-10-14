---
name: telegram-ru-proxy
description: "Whenever writing Telegram bot code that will run on a Russian server, use telegraf with SOCKS proxy. Raw fetch/axios to api.telegram.org silently fails on RU servers - messages disappear without any error. Triggers on: creating Telegram bots, adding bot notifications, sendMessage calls, telegraf setup, Telegram webhook configuration."
---

# Telegram Bot на RU сервере - proxy обязателен

Прямые запросы к `api.telegram.org` с российских IP блокируются Роскомнадзором.
`undici` (Node.js встроенный fetch) игнорирует `HTTP_PROXY` - сообщения молча теряются без ошибки.

## Правильный паттерн - telegraf с proxy

```js
import { Telegraf } from 'telegraf'
import { SocksProxyAgent } from 'socks-proxy-agent'

function createTelegrafBot(token) {
  const proxyUrl = process.env.HTTPS_PROXY || 'socks5://127.0.0.1:10801'
  const agent = new SocksProxyAgent(proxyUrl)
  return new Telegraf(token, {
    telegram: { agent }
  })
}

export const bot = createTelegrafBot(process.env.TELEGRAM_BOT_TOKEN)
```

## Отправка уведомлений без telegraf-инстанса

```js
import fetch from 'node-fetch'
import { SocksProxyAgent } from 'socks-proxy-agent'

async function sendTelegramMessage(chatId, text, parseMode = 'HTML') {
  const agent = new SocksProxyAgent(process.env.HTTPS_PROXY || 'socks5://127.0.0.1:10801')
  const url = `https://api.telegram.org/bot${process.env.TELEGRAM_BOT_TOKEN}/sendMessage`
  await fetch(url, {
    method: 'POST',
    agent,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ chat_id: chatId, text, parse_mode: parseMode })
  })
}
```

## Python - python-telegram-bot с proxy

```python
import os
from telegram import Bot
from telegram.request import HTTPXRequest

proxy = os.getenv("HTTPS_PROXY", "socks5://127.0.0.1:10801")
request = HTTPXRequest(proxy=proxy)
bot = Bot(token=os.getenv("TELEGRAM_BOT_TOKEN"), request=request)

await bot.send_message(chat_id=chat_id, text=text, parse_mode="HTML")
```

## Зависимости

```bash
# Node.js
npm install socks-proxy-agent node-fetch telegraf

# Python
pip install python-telegram-bot httpx[socks]
```

## .env для Telegram бота

```env
TELEGRAM_BOT_TOKEN=your_bot_token_here
TELEGRAM_CHAT_ID=your_chat_id_here
HTTPS_PROXY=socks5://127.0.0.1:10801
```

## Webhook vs Polling на RU сервере

- **Polling** - проще, работает из коробки с proxy, подходит для большинства задач
- **Webhook** - нужен публичный HTTPS. Nginx + certbot. Telegram -> your-server:443 -> бот
- Для продакшн: webhook через nginx (`yoursite.ru/bot-webhook`)

## Типичная ошибка

```
// НЕ РАБОТАЕТ на RU сервере:
await fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
  method: 'POST',
  body: JSON.stringify(...)
})
// Запрос уходит в никуда, ошибки нет, сообщение не доходит
```

## Проверка

```bash
# Проверить доступность Telegram API через proxy
curl -x socks5://127.0.0.1:10801 \
  "https://api.telegram.org/bot<YOUR_TOKEN>/getMe"
# Должен вернуть JSON с данными бота
```
