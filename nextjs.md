# ⚛️ 1. Server Components и Client Components в Next.js

Next.js (App Router) разделяет компоненты на **2 мира**:

---

## 🟢 Server Components (по умолчанию)

Это компоненты, которые:

- выполняются **только на сервере**
- НЕ попадают в JS-бандл браузера
- не имеют состояния React (`useState`, `useEffect` нельзя)
- могут напрямую работать с:
  - базой данных
  - файловой системой
  - API

### Пример:

```tsx id="srv1"
export default async function Page() {
  const data = await fetch("https://api.com/posts");

  return <div>{data.title}</div>;
}
```

---

### Плюсы:

- меньше JS в браузере
- быстрее загрузка
- безопасный доступ к backend

---

## 🔵 Client Components

Помечаются:

```tsx id="cli1"
"use client";
```

Это компоненты, которые:

- работают в браузере
- могут использовать:
  - useState
  - useEffect
  - события (onClick)

- имеют интерактивность

---

### Пример:

```tsx id="cli2"
"use client";

export default function Button() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

---

## ⚡ Главное различие

| Server Component       | Client Component       |
| ---------------------- | ---------------------- |
| выполняется на сервере | выполняется в браузере |
| нет state/hooks        | есть hooks             |
| меньше JS              | больше JS              |
| доступ к DB            | доступ к DOM           |

---

## 🔥 Важный момент

Server Components могут импортировать Client Components, но НЕ наоборот.

---

# ⚙️ 2. Server Actions (`"use server"`)

Server Actions — это функции, которые выполняются **на сервере**, но вызываются из клиента.

---

## Пример:

```tsx id="sa1"
"use server";

export async function createUser(formData: FormData) {
  await db.user.create({
    data: {
      name: formData.get("name"),
    },
  });
}
```

---

## Использование:

```tsx id="sa2"
<form action={createUser}>
  <input name="name" />
  <button>Send</button>
</form>
```

---

## Что это даёт:

- не нужно писать API routes
- меньше boilerplate
- безопасный серверный код
- прямой вызов backend-функций

---

## 🔥 Как это работает внутри:

1. браузер отправляет запрос
2. Next.js сериализует вызов
3. сервер выполняет функцию
4. результат возвращается

---

# 🌐 3. Глобальные переменные в Next.js

Это одна из самых частых ловушек на интервью.

---

## ❗ Важно:

В Next.js (особенно App Router):

> сервер может переиспользовать один и тот же процесс между запросами

---

## Пример:

```ts id="gv1"
let counter = 0;

export async function GET() {
  counter++;
  return Response.json({ counter });
}
```

---

## Что происходит:

- переменная живёт в памяти Node.js процесса
- может использоваться разными пользователями
- значение НЕ сбрасывается между запросами

---

## ❗ Проблема:

- утечки состояния между пользователями
- race conditions
- непредсказуемое поведение

---

## Когда это ОК:

- кеш
- метрики
- dev-only логика

---

## Когда это ОПАСНО:

- auth
- user data
- бизнес-логика

---

# 🧠 4. Что такое “глобальная переменная” в Next.js?

Это любая переменная:

```ts id="gv2"
const cache = {};
```

объявленная:

- вне компонента
- вне handler'а
- на уровне модуля

---

## Важно понимать:

Next.js работает в Node.js runtime:

👉 значит:

- модуль загружается 1 раз
- переменные “живут” в памяти сервера

---

## Но есть нюанс (очень важно):

### В serverless (Vercel):

- функция может перезапускаться
- глобальные переменные НЕ гарантированы

---

## Итог:

| Где          | Поведение             |
| ------------ | --------------------- |
| Node server  | живёт долго           |
| Serverless   | может сбрасываться    |
| Edge runtime | ещё более нестабильно |

---

# 💥 5. Ошибка гидрации (Hydration Error)

Это одна из самых частых ошибок в Next.js.

---

## Что такое hydration?

Hydration = процесс:

> React берёт HTML с сервера и “оживляет” его в браузере

---

## Пример:

Сервер:

```html
<div>0</div>
```

Клиент:

```js
<div>{Math.random()}</div>
```

---

## ❌ Ошибка:

Если HTML с сервера ≠ HTML на клиенте → hydration error

---

## Пример причины:

```tsx id="hyd1"
export default function Page() {
  return <div>{Math.random()}</div>;
}
```

---

## Почему это ломается:

- сервер отрендерил одно значение
- клиент отрендерил другое

---

## Частые причины hydration error:

### 1. window / document

```ts
if (typeof window !== "undefined") {
}
```

---

### 2. Math.random / Date.now

```ts
Date.now();
Math.random();
```

---

### 3. localStorage

```ts
localStorage.getItem("x");
```

---

### 4. условный рендер

```tsx
if (isClient) return <A />;
return <B />;
```

---

## Как исправляют:

### 1. useEffect

```tsx
useEffect(() => {
  setMounted(true);
}, []);
```

---

### 2. dynamic import (SSR off)

```ts
dynamic(() => import("./Component"), { ssr: false });
```

---

### 3. стабилизировать данные

---

## 🔥 Суть hydration проблемы:

> React ожидает, что сервер и клиент дадут ОДИНАКОВЫЙ HTML

---

# ⚡ Короткая супер-выжимка

- Server Components → сервер, без JS в браузере
- Client Components → интерактивность в браузере
- `"use server"` → серверные функции вызываемые с клиента
- global variables → живут в памяти сервера между запросами
- hydration error → mismatch HTML server vs client

---
