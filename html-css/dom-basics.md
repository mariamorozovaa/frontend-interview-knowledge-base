# 📘 JavaScript база знаний: Работа с DOM

## 1. Что такое document?

**Ответ:**

`document` — это глобальный объект, который представляет HTML-документ в браузере.

Он является точкой входа в DOM (Document Object Model) и позволяет управлять страницей.

```js
console.log(document.title);

document.title = "Новый заголовок";
```

---

## 2. Как искать элементы в DOM?

**Ответ:**

### Основные способы поиска:

```js
// По id
document.getElementById("header");

// По классу (устаревший HTMLCollection)
document.getElementsByClassName("item");

// CSS-селектор (первый элемент)
document.querySelector(".item");

// CSS-селектор (все элементы)
document.querySelectorAll(".item");
```

---

## 3. В чем разница querySelector и querySelectorAll?

**Ответ:**

- `querySelector` → возвращает первый найденный элемент
- `querySelectorAll` → возвращает список всех элементов (NodeList)

```js
document.querySelector(".btn"); // один элемент
document.querySelectorAll(".btn"); // список элементов
```

---

## 4. Как изменить DOM-элемент?

**Ответ:**

Можно менять:

- стили
- атрибуты
- текст
- HTML
- структуру

```js
const box = document.querySelector(".box");

// стили
box.style.color = "red";

// текст
box.textContent = "Привет";

// HTML
box.innerHTML = "<b>Жирный текст</b>";
```

---

## 5. Как создавать элементы в DOM?

**Ответ:**

```js
const div = document.createElement("div");
div.textContent = "Новый элемент";

document.body.appendChild(div);
```

---

## 6. Как работают события в DOM?

**Ответ:**

События обрабатываются через `addEventListener`.

```js
const btn = document.querySelector("button");

btn.addEventListener("click", () => {
  console.log("Клик!");
});
```

---

## 7. Какие бывают популярные события?

**Ответ:**

- click — клик мыши
- input — ввод текста
- submit — отправка формы
- keydown — нажатие клавиши
- mouseenter — наведение мыши

---

# 🧠 Практические задачи

---

## Задача 1. Изменение текста кнопки

```js id="btn-text"
const btn = document.querySelector("button");

btn.addEventListener("click", () => {
  btn.textContent = "Нажато!";
});
```

---

## Задача 2. Скрытие / показ блока

```html
<button id="toggle">Показать/Скрыть</button>
<div id="box">Контент</div>
```

```js id="toggle-box"
const btn = document.querySelector("#toggle");
const box = document.querySelector("#box");

btn.addEventListener("click", () => {
  box.style.display = box.style.display === "none" ? "block" : "none";
});
```

---

## Задача 3. Добавление элементов в список

```html
<input id="input" />
<button id="add">Добавить</button>
<ul id="list"></ul>
```

```js id="add-list"
const input = document.querySelector("#input");
const btn = document.querySelector("#add");
const list = document.querySelector("#list");

btn.addEventListener("click", () => {
  const li = document.createElement("li");

  li.textContent = input.value;
  list.appendChild(li);

  input.value = "";
});
```

---

## Задача 4. Удаление элемента

```js id="remove-item"
const items = document.querySelectorAll("li");

items.forEach((item) => {
  item.addEventListener("click", () => {
    item.remove();
  });
});
```

---

## Задача 5. Переключение класса

```js id="toggle-class"
const box = document.querySelector(".box");

box.addEventListener("click", () => {
  box.classList.toggle("active");
});
```

---

## Задача 6. Фильтрация списка

```html
<input id="search" />
<ul>
  <li>Anna</li>
  <li>Boris</li>
  <li>Victor</li>
</ul>
```

```js id="filter-list"
const input = document.querySelector("#search");
const items = document.querySelectorAll("li");

input.addEventListener("input", (e) => {
  const value = e.target.value.toLowerCase();

  items.forEach((item) => {
    const text = item.textContent.toLowerCase();

    item.style.display = text.includes(value) ? "" : "none";
  });
});
```

---

## Задача 7. Счётчик

```js id="counter"
let count = 0;

const span = document.querySelector("#count");

document.querySelector("#plus").addEventListener("click", () => {
  count++;
  span.textContent = count;
});

document.querySelector("#minus").addEventListener("click", () => {
  count--;
  span.textContent = count;
});
```

---

# 📊 Шпаргалка DOM

| Действие         | Метод                  |
| ---------------- | ---------------------- |
| Найти по id      | getElementById         |
| Найти по классу  | getElementsByClassName |
| Найти 1 элемент  | querySelector          |
| Найти все        | querySelectorAll       |
| Создать элемент  | createElement          |
| Добавить элемент | appendChild / append   |
| Удалить элемент  | remove                 |
| Изменить текст   | textContent            |
| Изменить HTML    | innerHTML              |
| Стили            | style                  |
| Классы           | classList              |

---

# 🚀 Итог

DOM — это способ JavaScript управлять HTML-страницей.

Главные навыки:

✔ находить элементы
✔ изменять их
✔ работать с событиями
✔ создавать динамику
