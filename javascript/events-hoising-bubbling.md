## 🧭 1. Как работает всплытие и погружение событий?

В DOM-событиях есть **3 фазы распространения события**:

### 1) Погружение (capturing phase)

Событие идёт **сверху вниз**:
`document → html → body → ... → target`

### 2) Целевая фаза (target phase)

Событие достигло конкретного элемента.

### 3) Всплытие (bubbling phase)

Событие идёт **снизу вверх**:
`target → body → html → document`

---

### Пример:

```html
<div id="outer">
  <button id="btn">Click</button>
</div>
```

```js
document.getElementById("outer").addEventListener("click", () => {
  console.log("outer");
});

document.getElementById("btn").addEventListener("click", () => {
  console.log("button");
});
```

При клике:

```
button → outer
```

---

## 🔁 2. Что идёт первым — погружение или всплытие?

👉 Сначала всегда идёт **погружение (capturing)**
👉 Потом **target phase**
👉 Потом **всплытие (bubbling)**

```text
CAPTURE → TARGET → BUBBLE
```

---

## 🎯 3. Как добавить обработчик на фазу погружения?

Используется третий аргумент `addEventListener`:

```js
element.addEventListener(
  "click",
  () => {
    console.log("capture phase");
  },
  true,
);
```

или:

```js
element.addEventListener("click", handler, {
  capture: true,
});
```

👉 `true` = включить capture-фазу

---

## 🫧 4. Как работает всплытие?

По умолчанию события идут в фазе **bubbling**:

```js
document.getElementById("outer").addEventListener("click", () => {
  console.log("outer");
});

document.getElementById("btn").addEventListener("click", () => {
  console.log("button");
});
```

Результат:

```
button
outer
```

📌 Потому что событие поднимается вверх по DOM-дереву.

---

## 🛑 5. Как остановить распространение события?

### 🔹 stopPropagation()

Останавливает дальнейшее распространение (вверх или вниз):

```js
btn.addEventListener("click", (e) => {
  e.stopPropagation();
  console.log("button only");
});
```

👉 outer больше не вызовется

---

### 🔹 stopImmediatePropagation()

Останавливает:

- другие обработчики на этом же элементе
- и дальнейшее распространение

```js
btn.addEventListener("click", (e) => {
  e.stopImmediatePropagation();
});
```

---

### 🔹 preventDefault() (важно не путать)

Останавливает **действие браузера**, но НЕ всплытие:

```js
a.addEventListener("click", (e) => {
  e.preventDefault(); // не перейдёт по ссылке
});
```

---

## ⚡ Короткая шпаргалка

| Метод                        | Что делает                                  |
| ---------------------------- | ------------------------------------------- |
| `stopPropagation()`          | Останавливает всплытие                      |
| `stopImmediatePropagation()` | Останавливает всё (включая другие handlers) |
| `preventDefault()`           | Останавливает стандартное поведение         |

---

## 🧠 Как сказать на собеседовании

> События в DOM проходят 3 фазы: capturing, target и bubbling.
> По умолчанию обработчики работают на фазе bubbling.
> Фазу capturing можно включить через `addEventListener(..., true)`.
> Для остановки распространения используется `stopPropagation()`, а для остановки дефолтного поведения — `preventDefault()`.

---
