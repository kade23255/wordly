# Wordly

Минималистичный словарный тренажёр в чёрно-белом стиле.  
Одно слово в день. Авторизация через GitHub. Данные в личном Gist.

## Структура

```
wordly/
├── index.html      # Весь сайт (один файл)
├── words.json      # База слов (добавляй каждый день)
└── README.md
```

## Деплой на GitHub Pages

```bash
git init
git add .
git commit -m "init: wordly"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/wordly.git
git push -u origin main
```

Затем: **Settings → Pages → Source: main / root** → Save.  
Сайт будет доступен по адресу `https://YOUR_USERNAME.github.io/wordly`

## Настройка OAuth

### 1. Обнови URL в index.html

Найди строку:
```js
const WORDS_RAW_URL = 'https://raw.githubusercontent.com/YOUR_USERNAME/wordly/main/words.json';
```
Замени `YOUR_USERNAME` на свой логин GitHub.

### 2. Обнови callback URL в OAuth App

GitHub → Settings → Developer settings → OAuth Apps → Wordly  
Измени **Authorization callback URL** на:
```
https://YOUR_USERNAME.github.io/wordly
```

### 3. Деплой прокси для обмена токена (обязательно)

GitHub OAuth требует `client_secret` на сервере. Деплоим бесплатный Cloudflare Worker:

1. Зарегистрируйся на [cloudflare.com](https://cloudflare.com)
2. Создай Worker → вставь код ниже
3. Замени `YOUR_CLIENT_ID` и `YOUR_CLIENT_SECRET` (из настроек OAuth App)

```js
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const code = url.searchParams.get('code');
    if (!code) return new Response('Missing code', { status: 400 });

    const res = await fetch('https://github.com/login/oauth/access_token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
      body: JSON.stringify({
        client_id: 'Ov23liHWvPHj6waRktL4',
        client_secret: 'YOUR_CLIENT_SECRET',
        code
      })
    });

    const data = await res.json();

    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': 'https://YOUR_USERNAME.github.io'
      }
    });
  }
};
```

4. Задеплой и скопируй URL вида `https://wordly-oauth.YOUR_SUBDOMAIN.workers.dev`
5. В `index.html` найди:
```js
const res = await fetch(`https://github-oauth-proxy.YOUR_USERNAME.workers.dev/token?code=${code}`);
```
Замени URL на свой Worker.

## Добавление новых слов

Просто добавляй записи в `words.json` с датой `YYYY-MM-DD` и делай `git push`.  
Сайт автоматически покажет слово по дате.

```json
{
  "date": "2026-05-11",
  "word": "ineffable",
  "phonetic": "/ɪˈnef.ə.bəl/",
  "pos": "adjective",
  "def": "Too great or extreme to be expressed in words.",
  "example": "The view from the summit was ineffable.",
  "topic": "Философия",
  "tags": ["C2", "Поэзия"]
}
```

## Как работает хранение данных

- Пользователь логинится через GitHub OAuth
- Прогресс сохраняется в **приватный GitHub Gist** пользователя
- Никаких серверов, никаких баз данных — всё у пользователя
