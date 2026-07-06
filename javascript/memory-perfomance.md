# 🧠 GC (Garbage Collector), утечки памяти и оптимизация

---

## 1. Как работает Garbage Collector (GC)?

Garbage Collector в JavaScript — это механизм, который **автоматически освобождает память**, удаляя объекты, которые больше недостижимы из кода.

### 🔑 Основная идея

JS использует **алгоритм достижимости (reachability)**:

> Если объект нельзя достичь из “корней” (root), он считается мусором.

---

### 📌 Root-объекты:

- `window` (в браузере)
- глобальные переменные
- текущий стек вызовов (Call Stack)
- активные таймеры / обработчики

---

### 🧹 Как GC решает, что удалить:

```js
let user = { name: "Alex" };

user = null;
// объект {name: "Alex"} больше недостижим → будет удалён GC
```

---

### ⚙️ Как работает внутри (упрощённо):

1. GC находит “корни”
2. Обходит все ссылки от корней
3. Помечает достижимые объекты
4. Всё остальное удаляет (mark-and-sweep)

---

## 2. Причины утечек памяти

Утечка памяти возникает, когда **объекты остаются в памяти, хотя уже не нужны**.

### Основные причины:

- глобальные переменные
- неочищенные `setInterval` / `setTimeout`
- забытые обработчики событий
- замыкания (closures), удерживающие ссылки
- кэш без ограничений
- DOM-элементы, удалённые из DOM, но сохранённые в переменных
- подписки (event listeners, observers), которые не удаляются

---

## 3. Основные ошибки, ведущие к утечкам памяти

---

### ❌ 1. Глобальные переменные

```js
userData = {}; // без let/const → висит в window
```

---

### ❌ 2. Неочищенные таймеры

```js
const id = setInterval(() => {
  console.log("tick");
}, 1000);

// забыли clearInterval(id)
```

---

### ❌ 3. Утечка через события

```js
button.addEventListener("click", handler);

// если DOM удалили, но listener не сняли → утечка
```

---

### ❌ 4. Замыкания

```js
function create() {
  const bigData = new Array(1000000);

  return function () {
    console.log(bigData.length);
  };
}
```

👉 `bigData` остаётся в памяти, пока жива функция

---

### ❌ 5. Хранение DOM ссылок

```js
const el = document.querySelector(".box");
document.body.removeChild(el);

// но ссылка el остаётся → память не освобождается
```

---

### ❌ 6. Неконтролируемый cache

```js
const cache = {};

function save(key, value) {
  cache[key] = value;
}
```

---

## 4. Как понять, что проект нужно оптимизировать?

---

### 🚨 Симптомы:

- страница начинает “тормозить” со временем
- увеличивается потребление памяти
- вкладка браузера становится тяжёлой
- лаги при скролле / анимациях
- долгий first render или re-render
- падение FPS (< 60)
- “зависания” после долгой работы

---

### 🔍 Инструменты диагностики:

- Chrome DevTools → Memory tab
- Performance tab (FPS, long tasks)
- Heap snapshots
- Allocation timeline

---

## 5. Способы оптимизации проекта

---

## ⚡ 1. Управление памятью

- удалять event listeners
- чистить setInterval / setTimeout
- избегать глобальных переменных
- обнулять ссылки на DOM

```js
element.removeEventListener("click", handler);
clearInterval(id);
```

---

## ⚡ 2. Оптимизация рендера

- использовать `React.memo`
- `useMemo`, `useCallback`
- избегать лишних re-render

---

## ⚡ 3. Работа с DOM

- минимизировать прямые изменения DOM
- использовать virtual DOM (React/Vue)
- batch updates

---

## ⚡ 4. Оптимизация вычислений

- debounce / throttle

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

---

## ⚡ 5. Оптимизация списков

- виртуализация (react-window, react-virtualized)

---

## ⚡ 6. Оптимизация сетевых запросов

- caching
- pagination
- lazy loading
- SWR / React Query

---

## ⚡ 7. Оптимизация загрузки

- code splitting
- dynamic import
- lazy components

```js
const Page = React.lazy(() => import("./Page"));
```

---

## ⚡ 8. Оптимизация анимаций

- использовать `transform`, `opacity`
- избегать layout thrashing
- requestAnimationFrame

---

## 📌 Короткая версия для собеседования

GC удаляет объекты, которые больше недостижимы из корневых ссылок. Утечки памяти возникают, когда объекты остаются в памяти из-за глобальных переменных, замыканий, неочищенных таймеров и обработчиков событий. Оптимизация включает контроль памяти, очистку подписок, уменьшение re-render, виртуализацию списков, lazy loading и оптимизацию сетевых запросов.

---
