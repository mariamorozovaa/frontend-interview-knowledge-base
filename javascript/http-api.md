Ниже — структурированный и чуть более глубокий разбор, как это обычно ждут на middle/senior собеседованиях.

---

# 🌐 HTTP, REST, сеть и браузер

---

## 1. Что такое HTTP?

**HTTP (HyperText Transfer Protocol)** — это протокол прикладного уровня, который используется для передачи данных между клиентом (браузером) и сервером.

### 📌 Особенности:

- работает по модели **request–response**
- не хранит состояние (stateless)
- работает поверх TCP (или TLS в HTTPS)
- используется для загрузки страниц, API, файлов

---

## 2. Из чего состоит HTTP-запрос?

HTTP-запрос состоит из 4 частей:

### 1) Request line

```http
GET /users HTTP/1.1
```

- метод (GET, POST…)
- путь (endpoint)
- версия HTTP

---

### 2) Headers

```http
Content-Type: application/json
Authorization: Bearer token
```

---

### 3) Body (не всегда)

```json
{
  "name": "Alex"
}
```

---

### 4) Query params (часто часть URL)

```
/users?page=2&limit=10
```

---

## 3. Что такое HTTPS?

**HTTPS = HTTP + TLS (SSL)**

Это защищённая версия HTTP, которая:

- шифрует данные
- защищает от MITM-атак
- проверяет подлинность сервера

---

## 4. Что такое SSL?

**SSL (Secure Sockets Layer)** — криптографический протокол для шифрования соединения.

👉 Сейчас SSL устарел, используется его версия:

> **TLS (Transport Layer Security)**

---

### Что делает TLS:

- шифрует трафик
- проверяет сертификаты
- обеспечивает целостность данных

---

## 5. Что возвращает fetch?

```js
fetch("/api");
```

👉 возвращает **Promise**, который резолвится в объект:

> `Response`

---

### Важно:

fetch НЕ возвращает сразу данные!

```js
const res = await fetch("/api");
const data = await res.json();
```

---

### Response содержит:

- status (200, 404…)
- headers
- body (stream)
- ok (true/false)

---

## 6. Что такое REST API?

**REST (Representational State Transfer)** — архитектурный стиль построения API.

REST API — это API, которое работает по REST-принципам через HTTP.

---

### Пример:

```
GET /users
POST /users
GET /users/1
DELETE /users/1
```

---

## 7. Принципы REST

### 🔑 1. Stateless

Каждый запрос независим

---

### 🔑 2. Client–Server

Фронт и бэк разделены

---

### 🔑 3. Uniform Interface

Единый стиль API:

- ресурсы через URL
- методы HTTP

---

### 🔑 4. Cacheable

Ответы могут кешироваться

---

### 🔑 5. Layered system

Можно использовать прокси, балансировщики

---

### 🔑 6. Resource-based

Всё — ресурс:

- /users
- /orders
- /products

---

## 8. Когда использовать HTTP методы?

---

### GET

👉 получение данных

```http
GET /users
```

---

### POST

👉 создание

```http
POST /users
```

---

### PUT

👉 полная замена

```http
PUT /users/1
```

---

### PATCH

👉 частичное обновление

```http
PATCH /users/1
```

---

### DELETE

👉 удаление

```http
DELETE /users/1
```

---

## 9. Можно ли сделать весь backend на GET?

❌ Теоретически можно, но это **плохая практика**

---

### Почему нельзя:

- GET должен быть безопасным (не менять данные)
- кешируется браузером
- теряется семантика API
- проблемы с безопасностью
- ограничения URL (длина)

---

👉 Это ломает REST и делает систему непредсказуемой

---

## 10. Зачем нужен OPTIONS?

**OPTIONS** — запрос для получения информации о сервере.

---

### Используется для:

- CORS preflight запросов
- проверки разрешённых методов

---

### Пример:

```http
OPTIONS /api/users
```

Ответ:

```
Allow: GET, POST, DELETE
```

---

## 11. Коды HTTP ответов

---

### 🟢 2xx — успех

- 200 OK
- 201 Created
- 204 No Content

---

### 🟡 3xx — редиректы

- 301 Moved Permanently
- 302 Found

---

### 🔴 4xx — ошибка клиента

- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 409 Conflict

---

### ⚫ 5xx — ошибка сервера

- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable

---

## 12. Что происходит, когда вводишь URL?

---

### 🔥 Полный процесс:

### 1. DNS lookup

```
example.com → IP
```

---

### 2. Установка TCP соединения

- handshake (SYN, SYN-ACK, ACK)

---

### 3. TLS handshake (если HTTPS)

- проверка сертификата
- обмен ключами

---

### 4. HTTP request

```
GET /
```

---

### 5. Сервер отвечает

```
HTML документ
```

---

### 6. Браузер:

- строит DOM
- строит CSSOM
- создаёт Render Tree
- layout
- paint
- composite

---

## 13. Что такое WebSocket?

**WebSocket — протокол постоянного соединения между клиентом и сервером.**

---

### Отличие от HTTP:

| HTTP             | WebSocket             |
| ---------------- | --------------------- |
| request/response | постоянное соединение |
| одноразовый      | двусторонний канал    |
| медленнее        | realtime              |

---

### Используется для:

- чатов
- игр
- trading apps
- live updates

---

## 14. Что такое AbortSignal?

**AbortSignal — механизм отмены fetch-запросов.**

---

### Пример:

```js
const controller = new AbortController();

fetch("/api", {
  signal: controller.signal,
});

controller.abort();
```

---

### Где используется:

- отмена запросов
- debounce поисков
- переходы между страницами

---

## 15. Что такое ReadableStream?

**ReadableStream — поток данных, который читается частями.**

---

### Используется когда:

- большие файлы
- видео
- потоковые ответы API

---

### Пример:

```js
const response = await fetch("/large-file");
const reader = response.body.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  console.log("chunk:", value);
}
```

---

### Почему это важно:

- не грузит всё в память
- можно обрабатывать данные “на лету”

---

# 📌 Короткая версия для собеседования

HTTP — это протокол клиент-серверного общения. HTTPS — его защищённая версия с TLS. fetch возвращает Promise с Response, а не данные. REST — архитектурный стиль API, основанный на ресурсах и HTTP методах. WebSocket даёт постоянное соединение для realtime. AbortController позволяет отменять запросы. ReadableStream позволяет читать данные потоками.

---
