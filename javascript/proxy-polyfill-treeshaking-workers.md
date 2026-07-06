# ⚙️ Proxy, polyfill, паттерны, workers, модули, tree traversal

---

## 1. Что такое Proxy?

**Proxy** — это встроенный объект в JavaScript, который позволяет **перехватывать и переопределять базовые операции над объектом**.

---

### 📌 Что можно перехватывать:

- чтение свойств (`get`)
- запись (`set`)
- удаление (`deleteProperty`)
- вызов функций
- проверку наличия (`in`)

---

### 📌 Пример:

```js id="p1"
const user = {
  name: "Alex",
};

const proxy = new Proxy(user, {
  get(target, prop) {
    return prop in target ? target[prop] : "Нет такого поля";
  },

  set(target, prop, value) {
    console.log(`Set ${prop} = ${value}`);
    target[prop] = value;
    return true;
  },
});

console.log(proxy.name); // Alex
console.log(proxy.age); // Нет такого поля
```

---

### 📌 Где используется:

- реактивность (Vue 3)
- валидация данных
- логирование доступа
- защита объектов

---

## 2. Что такое polyfill?

**Polyfill** — это код, который реализует функциональность, отсутствующую в старых браузерах.

---

### 📌 Пример:

```js id="p2"
if (!Array.prototype.includes) {
  Array.prototype.includes = function (value) {
    return this.indexOf(value) !== -1;
  };
}
```

---

### 📌 Зачем нужен:

- поддержка старых браузеров
- совместимость JS-фич (ES6+)

---

## 3. Что такое паттерны?

**Паттерны (design patterns)** — это типовые решения часто встречающихся задач в разработке.

---

### 📌 Основные группы:

### 🧱 Порождающие:

- Singleton
- Factory
- Builder

### 🧩 Структурные:

- Proxy
- Decorator
- Adapter

### ⚙️ Поведенческие:

- Observer
- Strategy
- Command

---

### 📌 Пример (Observer):

```js id="p3"
class EventBus {
  constructor() {
    this.listeners = {};
  }

  on(event, cb) {
    this.listeners[event] = this.listeners[event] || [];
    this.listeners[event].push(cb);
  }

  emit(event, data) {
    (this.listeners[event] || []).forEach((cb) => cb(data));
  }
}
```

---

### 📌 Зачем нужны:

- переиспользование решений
- читаемость архитектуры
- масштабируемость

---

## 4. Что такое Workers в JS?

**Web Workers** — это способ выполнять код **в отдельном потоке**, не блокируя UI.

---

### 📌 Особенности:

- не блокируют главный поток
- нет доступа к DOM
- общение через `postMessage`

---

### 📌 Пример:

```js id="w1"
// main.js
const worker = new Worker("worker.js");

worker.postMessage("start");

worker.onmessage = (e) => {
  console.log(e.data);
};
```

```js id="w2"
// worker.js
onmessage = (e) => {
  postMessage("done");
};
```

---

### 📌 Используется для:

- тяжёлых вычислений
- обработки данных
- криптографии
- изображений

---

## 5. Что такое Shared Worker?

**Shared Worker** — это worker, который **разделяется между несколькими вкладками/окнами**.

---

### 📌 Отличие от Web Worker:

| Web Worker   | Shared Worker     |
| ------------ | ----------------- |
| одна вкладка | несколько вкладок |
| изолирован   | общий контекст    |

---

### 📌 Пример:

```js id="s1"
const worker = new SharedWorker("worker.js");

worker.port.postMessage("hello");
```

---

### 📌 Используется для:

- синхронизации вкладок
- общего состояния (например чат, кэш)

---

## 6. Как обходить большое дерево? Почему не рекурсия?

---

### 📌 Основные способы обхода:

- DFS (глубина)
- BFS (ширина)

---

## ❌ Почему рекурсия плохо:

### 1. Переполнение стека

```js id="d1"
function dfs(node) {
  dfs(node.children);
}
```

---

### 2. Неэффективность при глубоком дереве

- стек ограничен (~10k вызовов)
- риск crash

---

## ✅ Лучше — итерация:

### DFS через стек:

```js id="d2"
function dfs(root) {
  const stack = [root];

  while (stack.length) {
    const node = stack.pop();
    console.log(node.value);

    if (node.children) {
      stack.push(...node.children);
    }
  }
}
```

---

### BFS через очередь:

```js id="d3"
function bfs(root) {
  const queue = [root];

  while (queue.length) {
    const node = queue.shift();
    console.log(node.value);

    if (node.children) {
      queue.push(...node.children);
    }
  }
}
```

---

## 7. Что такое модули в JS?

**Модули** — это способ разделения кода на независимые части.

---

### 📌 ES Modules:

```js id="m1"
// export
export const sum = (a, b) => a + b;

// import
import { sum } from "./math.js";
```

---

### 📌 CommonJS (Node.js):

```js id="m2"
module.exports = { sum };
const { sum } = require("./math");
```

---

### 📌 Зачем нужны:

- изоляция кода
- повторное использование
- dependency management
- уменьшение глобального scope

---

## 8. Что такое Tree Shaking?

**Tree Shaking — это удаление неиспользуемого кода при сборке приложения.**

---

### 📌 Как это работает:

Если ты импортировал только часть модуля:

```js id="t1"
import { sum } from "./math";
```

а внутри есть:

```js id="t2"
export const sum = () => {};
export const multiply = () => {};
```

👉 `multiply` будет удалён из финального bundle

---

### 📌 Требования:

- ES Modules (import/export)
- сборщик (Webpack, Vite, Rollup)
- side-effect free код

---

### 📌 Пример пользы:

Без tree shaking:

```
bundle = 500 KB
```

С tree shaking:

```
bundle = 120 KB
```

---

### 📌 Где важно:

- React apps
- большие UI библиотеки
- design systems

---

# 📌 Коротко для собеседования

Proxy позволяет перехватывать операции над объектом. Polyfill — это реализация отсутствующих функций для старых браузеров. Паттерны — это типовые архитектурные решения. Workers позволяют выполнять код в отдельном потоке. Shared Worker делится между вкладками. Большие деревья лучше обходить итеративно, чтобы избежать переполнения стека. Модули позволяют разделять код на независимые части. Tree shaking — это удаление неиспользуемого кода при сборке.

---
