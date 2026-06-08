# useRef, React Fragment

## 1. useRef

### 1.1 В чем основной плюс useRef в отличие от useState?

```jsx
// useRef позволяет хранить значение между рендерами БЕЗ перерисовки компонента
// useState хранит значение и вызывает повторный рендер при изменении

// ========== Сравнение ==========
const [state, setState] = useState(0); // изменение → ререндер
const ref = useRef(0); // изменение → НЕТ ререндера

// Основной плюс useRef:
// Можно изменять значение, не вызывая лишних перерисовок компонента
```

### 1.2 Когда используют useRef? Примеры

```jsx
// 1. Работа с DOM-элементами (фокус, размеры, скролл)
const inputRef = useRef(null);
inputRef.current.focus(); // установить фокус
const width = divRef.current.offsetWidth; // получить ширину

// 2. Хранение значений, которые не должны вызывать ререндер
const timerRef = useRef(null);
timerRef.current = setTimeout(() => {}, 1000);
clearTimeout(timerRef.current);

// 3. Хранение предыдущего значения
const prevValueRef = useRef();
useEffect(() => {
  prevValueRef.current = value;
}, [value]);

// 4. Хранение счетчиков, интервалов, WebSocket
const intervalRef = useRef(null);
intervalRef.current = setInterval(() => {}, 1000);

// 5. Хранение объектов, которые должны жить между рендерами, но не обновлять UI
const formDataRef = useRef({}); // данные формы без ререндера
```

### 1.3 Разница между useRef и переменной вне компонента

```jsx
// Переменная вне компонента:
let globalCount = 0;

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>State: {count}</p>
      <button onClick={() => setCount(count + 1)}>State+</button>
      <p>Global: {globalCount}</p>
      <button onClick={() => globalCount++}>Global+</button>
    </div>
  );
}

// ========== Проблемы глобальной переменной ==========
// 1. ОБЩАЯ для всех экземпляров компонента
//    (все три счетчика используют одну переменную)

// 2. Изменение НЕ вызывает ререндер
//    (число на экране не обновляется)

// ========== useRef (правильное решение) ==========
function Counter() {
  const countRef = useRef(0); // уникальный для каждого компонента
  const [, forceUpdate] = useState(0); // принудительный ререндер

  return (
    <div>
      <p>Ref: {countRef.current}</p>
      <button
        onClick={() => {
          countRef.current++;
          forceUpdate((prev) => prev + 1); // принудительно обновляем UI
        }}>
        Ref+
      </button>
    </div>
  );
}

// ========== Итог: useRef vs глобальная переменная ==========
// useRef              | глобальная переменная
// --------------------|--------------------
// уникальный для      | общая для всех
// каждого компонента  | экземпляров
// --------------------|--------------------
// не вызывает ререндер| не вызывает ререндер
// --------------------|--------------------
// привязан к жизни    | живёт всё время
// компонента          | работы приложения
```

---

## 2. React Fragment

### 2.1 Для чего нужен Fragment?

```jsx
// Fragment нужен, чтобы группировать несколько элементов
// БЕЗ создания лишнего HTML-элемента в DOM

// ❌ Без Fragment (появляется лишний div)
function App() {
  return (
    <div>
      {" "}
      {/* ← лишний div в DOM */}
      <h1>Заголовок</h1>
      <p>Текст</p>
    </div>
  );
}

// ✅ С Fragment (нет лишнего элемента)
function App() {
  return (
    <>
      <h1>Заголовок</h1>
      <p>Текст</p>
    </>
  );
}

// В DOM будет:
// <h1>Заголовок</h1>
// <p>Текст</p>
// (без обёртки!)
```

### 2.2 Разница между `<> </>` и `<React.Fragment> </>`

```jsx
// ========== Короткая запись (<> </>) ==========
// ✅ Проще и короче
// ❌ НЕ поддерживает атрибуты (например, key)

function TodoList({ items }) {
    return (
        <>
            {items.map(item => (
                // ❌ Ошибка! Нельзя добавить key к короткому фрагменту
                // <> key={item.id}>
                //     <h3>{item.title}</h3>
                //     <p>{item.description}</p>
                // </>
            ))}
        </>
    );
}

// ========== Полная запись (<React.Fragment>) ==========
// ✅ Поддерживает атрибуты (key)
// ❌ Более длинная запись

function TodoList({ items }) {
    return (
        <React.Fragment>
            {items.map(item => (
                <React.Fragment key={item.id}>  {/* ✅ key работает */}
                    <h3>{item.title}</h3>
                    <p>{item.description}</p>
                </React.Fragment>
            ))}
        </React.Fragment>
    );
}
```

---

## Практические задачи

### Задача 1. useRef не вызывает ререндер

```jsx
function App() {
  const countRef = React.useRef(0);

  const handleClick = () => {
    countRef.current++;
    console.log("Clicked:", countRef.current);
  };

  return (
    <div>
      <p>Счётчик: {countRef.current}</p>
      <button onClick={handleClick}>+</button>
    </div>
  );
}

// Ответ:
// На экране всегда будет 0 (useRef не вызывает ререндер)
// В консоли будут увеличиваться числа (1, 2, 3...)
// Интерфейс не обновляется
```

### Задача 2. Фокус на input через useRef

```jsx
function App() {
  const inputRef = React.useRef(null);

  const handleClick = () => {
    inputRef.current.focus(); // устанавливаем фокус на input
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={handleClick}>Фокус на input</button>
    </div>
  );
}
```

### Задача 3. Исправление счётчика (глобальная переменная → useState)

```jsx
// ❌ Исходный код с проблемами:
import { useState } from "react";

let count = 0; // глобальная переменная — общая для всех счетчиков

export const Counter = () => {
  const [rerender, setRerender] = useState(0);

  return (
    <div>
      <p>{count}</p>
      <button
        onClick={() => {
          count++;
          setRerender((prev) => prev + 1);
        }}>
        +
      </button>
      <button
        onClick={() => {
          count--;
          setRerender((prev) => prev + 1);
        }}>
        -
      </button>
    </div>
  );
};

// ========== Проблемы ==========
// 1. count общий для всех счетчиков (меняются все вместе)
// 2. Нужен костыль с rerender для обновления UI

// ========== Исправленный код ==========
import { useState } from "react";

export const Counter = () => {
  const [count, setCount] = useState(0); // каждый счетчик имеет свой count

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount((prev) => prev + 1)}>+</button>
      <button onClick={() => setCount((prev) => prev - 1)}>-</button>
    </div>
  );
};

function App() {
  return (
    <div>
      <h1>Счетчики</h1>
      <div style={{ display: "flex", flexDirection: "column" }}>
        <Counter /> {/* независимый */}
        <Counter /> {/* независимый */}
        <Counter /> {/* независимый */}
      </div>
    </div>
  );
}
```

---

## Шпаргалка

### useRef vs useState

| Характеристика                  | useRef | useState |
| ------------------------------- | ------ | -------- |
| Хранит значение между рендерами | ✅     | ✅       |
| Вызывает ререндер при изменении | ❌     | ✅       |
| Доступ к DOM элементам          | ✅     | ❌       |
| Хранение таймеров/интервалов    | ✅     | ❌       |
| Предыдущее значение             | ✅     | ❌       |

### React Fragment

| Особенность                | `<> </>` | `<React.Fragment>` |
| -------------------------- | -------- | ------------------ |
| Короткая запись            | ✅       | ❌                 |
| Поддержка `key`            | ❌       | ✅                 |
| Поддержка других атрибутов | ❌       | ❌ (только key)    |

### Когда что использовать?

| Ситуация                             | Решение                      |
| ------------------------------------ | ---------------------------- |
| Нужно сгруппировать элементы без key | `<> </>`                     |
| Нужен key для списка фрагментов      | `<React.Fragment key={...}>` |
| Доступ к DOM элементу                | `useRef`                     |
| Хранение значения без ререндера      | `useRef`                     |
| Значение должно обновлять UI         | `useState`                   |

---

## Codewars

### 6kyu (нужно решить 5 задач)

> 📌 **Формат сдачи результатов:** скриншот на котором видно текст задания + код решения

---
