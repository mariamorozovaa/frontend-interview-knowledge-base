Ниже — структурированный и “интервью-уровня” разбор **видов тестирования**, **пирамиды тестирования**, **инструментов**, **примеров кода** и **покрытия**.

---

# 🧪 Виды тестирования (frontend / JS / TS)

## 1. Unit-тесты (модульные тесты)

### 💡 Что это?

Тестируют **одну маленькую единицу кода**:

- функцию
- хук
- утилиту
- класс

👉 Без зависимостей (или с моками)

---

### 📌 Пример (Jest)

```ts
// sum.ts
export function sum(a: number, b: number) {
  return a + b;
}
```

```ts
// sum.test.ts
import { sum } from "./sum";

test("adds numbers", () => {
  expect(sum(2, 3)).toBe(5);
});
```

---

### 🧰 Библиотеки

- Jest (самый популярный)
- Vitest (быстрее, Vite ecosystem)
- Mocha + Chai (классика)

---

### 🎯 Что покрывают

- функции
- reducers (Redux)
- утилиты
- бизнес-логика

---

# 2. Integration tests (интеграционные)

### 💡 Что это?

Проверяют **взаимодействие нескольких модулей**

---

### 📌 Пример

```ts
function getUserInfo(userService, id) {
  return userService.getUser(id);
}
```

```ts
test("integration user service", async () => {
  const fakeService = {
    getUser: async (id) => ({ id, name: "Anna" }),
  };

  const result = await getUserInfo(fakeService, 1);

  expect(result.name).toBe("Anna");
});
```

---

### 🧰 Библиотеки

- Jest
- Testing Library (React)
- Supertest (API)

---

### 🎯 Что покрывают

- компоненты + API
- модули между собой
- сервисы

---

# 3. End-to-End (E2E) тесты

### 💡 Что это?

Тестируют систему **как пользователь**

👉 Браузер → UI → API → база

---

### 📌 Пример (Cypress)

```js
describe("Login flow", () => {
  it("user can login", () => {
    cy.visit("/login");

    cy.get("input[name=email]").type("test@mail.com");
    cy.get("input[name=password]").type("123456");

    cy.get("button[type=submit]").click();

    cy.url().should("include", "/dashboard");
  });
});
```

---

### 🧰 Библиотеки

- Cypress (очень популярный)
- Playwright (современный и мощный)
- Puppeteer (Chrome automation)

---

### 🎯 Что покрывают

- UI сценарии
- пользовательские флоу
- критические бизнес-пути

---

# 4. Component tests (React)

### 💡 Что это?

Тестируют React-компоненты отдельно

---

### 📌 Пример (React Testing Library)

```tsx
import { render, screen } from "@testing-library/react";
import Button from "./Button";

test("renders button", () => {
  render(<Button label="Click me" />);

  expect(screen.getByText("Click me")).toBeInTheDocument();
});
```

---

### 🧰 Библиотеки

- React Testing Library
- Jest
- Vitest

---

### 🎯 Что покрывают

- UI компоненты
- поведение пользователя
- props/state

---

# 5. Snapshot testing

### 💡 Что это?

Сравнение UI “как есть сейчас” с сохранённым состоянием

---

### 📌 Пример

```tsx
test("snapshot", () => {
  const { container } = render(<Button label="Click" />);
  expect(container).toMatchSnapshot();
});
```

---

### ⚠️ Минусы

- легко ломаются
- часто бесполезны при плохом использовании

---

### 🎯 Что покрывают

- визуальную стабильность UI

---

# 6. API testing

### 💡 Что это?

Тестируют backend endpoints

---

### 📌 Пример (Supertest)

```ts
import request from "supertest";
import app from "../app";

test("GET /users", async () => {
  const res = await request(app).get("/users");

  expect(res.status).toBe(200);
  expect(res.body.length).toBeGreaterThan(0);
});
```

---

### 🧰 Библиотеки

- Supertest
- Postman / Newman
- Axios + Jest

---

### 🎯 Что покрывают

- REST API
- GraphQL
- ответы сервера

---

# 7. Smoke / Sanity tests

### 💡 Что это?

- **Smoke** — "приложение вообще живо?"
- **Sanity** — "эта фича работает?"

---

### 📌 Пример

Smoke:

```ts
test("app loads", () => {
  render(<App />);
  expect(screen.getByText("Welcome")).toBeInTheDocument();
});
```

---

# 🏗 Пирамида тестирования

```
        E2E (мало, дорого)
          ▲
     Integration
          ▲
        Unit (много, дешево)
```

---

## 📊 Распределение (идеально)

| Тип тестов  | % покрытия |
| ----------- | ---------- |
| Unit        | 60–80%     |
| Integration | 15–30%     |
| E2E         | 5–10%      |

---

## ❗ Почему так?

- Unit → быстрые и дешёвые
- Integration → проверяют связи
- E2E → медленные и хрупкие

---

# 📈 Покрытие кода (Code Coverage)

## 📌 Что это?

Показывает:

- какие строки кода протестированы
- какие ветки (if/else)
- функции
- строки

---

## 📊 Типы покрытия:

- Line coverage
- Function coverage
- Branch coverage
- Statement coverage

---

## 📌 Пример Jest config

```json
{
  "collectCoverage": true,
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}
```

---

## 🎯 Рекомендуемые проценты

| Уровень проекта            | Coverage |
| -------------------------- | -------- |
| MVP / стартап              | 50–70%   |
| Production                 | 70–85%   |
| FinTech / critical systems | 85–95%   |

---

⚠️ Важно:

> 100% coverage ≠ хорошее качество тестов

---

# 🧰 Основные библиотеки тестирования

## Unit / Integration

- Jest
- Vitest
- Mocha + Chai

## React

- React Testing Library
- Enzyme (устарел)

## E2E

- Cypress
- Playwright
- Puppeteer

## API

- Supertest
- Postman / Newman

---

# 🧠 Что обычно спрашивают на собеседовании

### ❓ Почему пирамида?

👉 Потому что:

- unit дешёвые → их больше
- E2E дорогие → их меньше

---

### ❓ Что лучше: Jest или Cypress?

- Jest → unit / integration
- Cypress → E2E

---

### ❓ Что важнее: coverage или качество тестов?

👉 Качество важнее

---

# 🚀 Мини-резюме

- Unit → функции
- Integration → модули
- E2E → пользовательские сценарии
- Snapshot → UI сравнение
- API → backend
- Coverage → метрика качества
- Пирамида → баланс стоимости и пользы

---

Отлично, это как раз **уровень senior+ TypeScript мышления** — понять, как устроены `Pick`, `Omit`, `Partial`, `Record` и как писать свои.

Я объясню **не просто код**, а _как ты должен думать, чтобы их писать сам_.

---

# 🧠 База: как работают utility types

Все такие типы строятся из 3 кирпичей:

### 1. `keyof`

Получаем ключи объекта:

```ts
type User = {
  name: string;
  age: number;
};

type Keys = keyof User;
// "name" | "age"
```

---

### 2. `T[K]` (indexed access)

Достаём тип значения по ключу:

```ts
type User = {
  name: string;
  age: number;
};

type NameType = User["name"]; // string
type AgeType = User["age"]; // number
```

---

### 3. Mapped types (`in`)

Перебираем ключи:

```ts
type Copy<T> = {
  [K in keyof T]: T[K];
};
```

👉 Это уже почти все utility types.

---

# 📦 1. Как работает `Pick<T, K>`

## 📌 Что делает:

Берёт только нужные ключи из объекта.

```ts
type User = {
  name: string;
  age: number;
  email: string;
};
```

---

## 🧠 Реализация Pick

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

---

## 💥 Разбор по шагам

### 1. `K extends keyof T`

Ограничиваем ключи:

```ts
K = "name" | "age";
```

---

### 2. `[P in K]`

Перебираем только выбранные ключи:

```ts
"name" | "age";
```

---

### 3. `T[P]`

Берём тип значения:

```ts
name → string
age → number
```

---

## ✅ Использование

```ts
type UserPreview = MyPick<User, "name" | "email">;
```

---

# 🚫 2. Как работает `Omit<T, K>`

## 📌 Что делает:

Удаляет ключи из объекта.

---

## 🧠 Реализация Omit

```ts
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P];
};
```

---

## 💥 Разбор

### 1. `keyof T`

Все ключи:

```ts
"name" | "age" | "email";
```

---

### 2. `as` + conditional filtering

```ts
P extends K ? never : P
```

👉 если ключ нужно удалить → превращаем в `never`

---

### 3. `never` = удаление ключа

TypeScript просто выбрасывает его.

---

## 🔥 Пример

```ts
type UserWithoutEmail = MyOmit<User, "email">;
```

---

# ⚡ 3. Partial<T>

## 📌 Делает все поля optional

---

## 🧠 Реализация

```ts
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};
```

---

## 💥 Разбор

`?` делает поле необязательным

```ts
type UserPartial = MyPartial<User>;
```

---

## 👉 результат:

```ts
{
  name?: string;
  age?: number;
  email?: string;
}
```

---

# 🔒 4. Required<T>

## 📌 Убирает optional

```ts
type MyRequired<T> = {
  [P in keyof T]-?: T[P];
};
```

---

## 💥 ключ `-?`

👉 убирает optional (`?`)

---

# 📚 5. Readonly<T>

```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

---

## 💥 делает:

```ts
{
  readonly name: string;
  readonly age: number;
}
```

---

# 🧱 6. Record<K, V>

## 📌 создаёт объект из ключей и значений

---

## 🧠 реализация

```ts
type MyRecord<K extends keyof any, V> = {
  [P in K]: V;
};
```

---

## 💥 разбор

### `keyof any`

```ts
string | number | symbol;
```

---

## пример:

```ts
type Roles = MyRecord<"admin" | "user", boolean>;
```

---

## результат:

```ts
{
  admin: boolean;
  user: boolean;
}
```

---

# 🚀 7. DeepReadonly (реальный senior уровень)

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};
```

---

## 💥 идея:

- если поле объект → рекурсивно делаем readonly
- иначе → оставляем как есть

---

# 🧠 8. Как самому писать utility types (алгоритм)

Когда видишь задачу:

### 1. Нужно пройтись по объекту?

👉 `keyof + in`

---

### 2. Нужно взять часть ключей?

👉 `K extends keyof T`

---

### 3. Нужно удалить ключи?

👉 `as + never`

---

### 4. Нужно изменить значения?

👉 `T[P]`

---

### 5. Нужно условие?

👉 `extends ? :`

---

# 🧪 Мини-тренировка (очень важно)

Попробуй сам написать:

---

## 🔹 MyPick

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

---

## 🔹 MyOmit

```ts
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P];
};
```

---

## 🔹 MyPartial

```ts
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};
```

---

## 🔹 MyReadonly

```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

---

# 🔥 Главное понимание

Все utility types = комбинация:

```
keyof + in + T[K] + conditional types
```

---

Фреймворки придумали не потому, что “так модно”, а потому что **чистый JavaScript (и даже чистый HTML/CSS) плохо масштабируется в реальных приложениях**.

Разберём по-человечески: _что было раньше → какие проблемы → что дали фреймворки → зачем они реально нужны сегодня_.

---

# 🧠 1. Как было “до фреймворков”

Раньше веб выглядел так:

- HTML — разметка
- CSS — стили
- JS — ручное управление DOM

Пример:

```js
const btn = document.querySelector("button");

btn.addEventListener("click", () => {
  const list = document.querySelector("ul");

  const item = document.createElement("li");
  item.textContent = "New item";

  list.appendChild(item);
});
```

---

## ❗ Проблема

На маленьком проекте — ок.

На большом:

- сотни DOM-операций
- куча `querySelector`
- сложно следить за состоянием
- легко ломается логика

---

# 🧨 2. Основные проблемы “vanilla JS” (без фреймворков)

## 1. ❌ Сложность управления состоянием

Пример:

```js
let count = 0;

function render() {
  document.body.innerHTML = `Count: ${count}`;
}
```

👉 Любое изменение = ручной `render()`

---

## 2. ❌ Ручное обновление DOM

Ты сам решаешь:

- что перерисовать
- где обновить
- когда обновить

---

## 3. ❌ Нет структуры проекта

Всё превращается в:

```
script1.js
script2.js
utils-final-v3.js
fix-bug-real-final.js
```

---

## 4. ❌ Сложно переиспользовать код

Если есть логика:

- форма логина
- валидация
- обработка API

👉 её сложно переиспользовать

---

## 5. ❌ Плохая масштабируемость

Когда проект растёт:

- код становится спагетти
- зависимости перемешиваются
- баги появляются в неожиданных местах

---

# 🚀 3. Что дали фреймворки

Фреймворки (React, Angular, Vue, Next.js) появились, чтобы:

> ❗ Убрать ручное управление DOM и сделать разработку предсказуемой

---

# ⚛️ 4. Главная идея фреймворков

## 👉 "Ты описываешь ЧТО нужно сделать, а не КАК"

---

### ❌ Без фреймворка:

```js
document.getElementById("title").innerText = "Hello";
```

---

### ✅ С фреймворком:

```tsx
return <h1>{title}</h1>;
```

👉 Ты не трогаешь DOM напрямую

---

# 🧠 5. Основные задачи, которые решают фреймворки

---

## 1. 📦 Декларативный UI

Ты описываешь состояние:

```tsx
const [count, setCount] = useState(0);

return <button onClick={() => setCount(count + 1)}>{count}</button>;
```

👉 UI автоматически обновляется

---

## 2. 🔁 Реактивность

Фреймворк сам отслеживает:

- изменения состояния
- какие компоненты перерендерить

---

## 3. 🧩 Компонентная архитектура

UI делится на части:

```
Header
Sidebar
Content
Footer
```

👉 Каждая часть независима

---

## 4. 🔄 Управление состоянием

Фреймворки вводят:

- state
- props
- context
- store (Redux, Zustand)

---

## 5. ⚡ Оптимизация обновлений (Virtual DOM / diffing)

React, Vue:

- не трогают DOM напрямую
- сначала сравнивают виртуальное дерево
- обновляют только изменения

---

## 6. 🧪 Переиспользуемость кода

Можно делать:

- компоненты
- хуки
- сервисы

---

## 7. 🏗 Архитектурная структура

Фреймворки задают правила:

- как писать код
- как организовывать проект
- как разделять слои

---

# ⚡ 6. Почему это важно именно для больших приложений

Представь:

### интернет-магазин

- корзина
- каталог
- фильтры
- поиск
- профиль
- оплата

---

## Без фреймворка:

- хаос DOM
- сложно синхронизировать состояние
- баги при обновлении UI

---

## С фреймворком:

- каждый блок — компонент
- данные централизованы
- UI автоматически обновляется

---

# 🧠 7. Главная идея фреймворков (самое важное)

> Фреймворк превращает хаотичный DOM-код в управляемую систему состояний

---

# ⚖️ 8. Библиотека vs фреймворк

| Библиотека       | Фреймворк           |
| ---------------- | ------------------- |
| ты вызываешь её  | она управляет тобой |
| React (частично) | Angular             |
| меньше правил    | строгая архитектура |

---

# 🚀 9. Почему именно React стал популярным

- Virtual DOM
- компоненты
- JSX
- огромная экосистема
- гибкость

---

# 🧠 Итог (коротко как на собеседовании)

Фреймворки придумали, чтобы:

- убрать ручное управление DOM
- структурировать код
- упростить масштабирование
- управлять состоянием приложения
- ускорить разработку
- уменьшить количество багов

---
