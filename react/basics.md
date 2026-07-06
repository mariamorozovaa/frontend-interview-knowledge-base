# React — вопросы и ответы

---

## 1. Почему перешли с HTML + JS на React?

Потому что в классическом подходе:

- DOM сложно масштабировать вручную
- много “imperative” кода (`document.querySelector`)
- сложно поддерживать состояние UI
- легко допустить баги при обновлении DOM

### Проблемы старого подхода:

- ручные обновления DOM
- дублирование логики
- сложно переиспользовать UI

### React решает:

- декларативный подход (“что должно быть”, а не “как сделать”)
- компонентная архитектура
- Virtual DOM
- удобное управление состоянием

---

## 2. Что такое JSX?

JSX — это синтаксическое расширение JS, позволяющее писать HTML внутри JavaScript.

```jsx
const element = <h1>Hello</h1>;
```

### На самом деле:

JSX → превращается в `React.createElement`

```javascript
React.createElement("h1", null, "Hello");
```

---

## 3. Является ли React реактивным?

👉 Частично — да, но не как Vue/Solid.

React:

- НЕ отслеживает зависимости автоматически
- НЕ реактивный в классическом смысле
- обновляется через `setState` / `useState`

👉 Поэтому React называют:

> “UI library with reactive rendering model”

---

## 4. Что такое компонент?

Компонент — это:

> независимая, переиспользуемая часть UI

```jsx
function Button() {
  return <button>Click</button>;
}
```

---

## 5. Жизненный цикл компонентов

### Class components:

1. Mounting (создание)
2. Updating (обновление)
3. Unmounting (удаление)

### Functional components (hooks):

- useEffect заменяет lifecycle

```text
Mount → useEffect(() => {}, [])
Update → useEffect(() => {}, [deps])
Unmount → return cleanup
```

---

## 6. Как React понимает, что нужно перерендерить?

React делает:

- сравнение state/props (shallow comparison)
- если изменился reference → rerender

### Пример:

```javascript
setState(newValue);
```

👉 React ставит компонент в очередь на rerender

---

## 7. Virtual DOM vs Real DOM

### Virtual DOM:

- JS-объект
- легкий
- быстро сравнивается

### Real DOM:

- тяжёлый
- медленные операции

### Процесс:

1. создаётся новый Virtual DOM
2. diff с предыдущим
3. минимальные изменения в real DOM

---

## 8. Триггеры ререндера

- изменение `state`
- изменение `props`
- изменение `context`
- rerender родителя
- forceUpdate (class components)

---

## 9. React Fragment

Позволяет возвращать несколько элементов без лишнего DOM узла:

```jsx
<>
  <div />
  <div />
</>
```

---

## 10. StrictMode

Используется только в dev:

- находит side effects
- вызывает некоторые функции дважды (для проверки)
- предупреждает об unsafe lifecycle

---

## 11. HOC (Higher Order Component)

Функция, которая:

> принимает компонент → возвращает новый компонент

```javascript
function withAuth(Component) {
  return function Wrapped(props) {
    return <Component {...props} />;
  };
}
```

---

## 12. “Lower Order Component” ❗

Такого термина в React нет.

👉 Обычно имеют в виду:

- обычный компонент
- базовый (presentational) компонент

---

## 13. Что такое hook?

Hook — функция, позволяющая использовать state и lifecycle в functional components.

---

## 14. useState + как работает внутри

```javascript
const [state, setState] = useState(0);
```

### Упрощённо внутри:

```javascript
let memory = [];

function useState(initial) {
  const index = currentIndex;

  memory[index] = memory[index] ?? initial;

  function setState(value) {
    memory[index] = value;
    rerender();
  }

  currentIndex++;
  return [memory[index], setState];
}
```

👉 React хранит состояние в “ячейках”

---

## 15. useEffect

```javascript
useEffect(() => {
  console.log("mounted");

  return () => {
    console.log("cleanup");
  };
}, [deps]);
```

### Когда вызывается:

- после render
- после commit в DOM

---

## 16. useLayoutEffect

Работает:

> ДО того, как браузер отрисует экран

### Отличие:

- useEffect → async (после paint)
- useLayoutEffect → sync (до paint)

Используется для:

- измерения DOM
- предотвращения flicker

---

## 17. useRef

Хранит:

- mutable value
- DOM reference

```javascript
const ref = useRef(0);
ref.current++;
```

👉 изменение ref НЕ вызывает rerender

---

## 18. useMemo

Кеширует результат вычисления:

```javascript
const value = useMemo(() => heavyCalc(a, b), [a, b]);
```

---

## 19. useCallback

Кеширует функцию:

```javascript
const fn = useCallback(() => {
  console.log("hello");
}, []);
```

---

## 20. Как работает useCallback

Упрощённо:

```javascript
let cache = null;

function useCallback(fn, deps) {
  if (depsChanged(deps)) {
    cache = fn;
  }
  return cache;
}
```

👉 возвращает одну и ту же ссылку между рендерами

---

## 21. React.memo

HOC, который:

> предотвращает rerender если props не изменились

```javascript
export default React.memo(Component);
```

---

## 22. Зачем cleanup в useEffect?

Чтобы избежать:

- утечек памяти
- подписок после unmount
- лишних сетевых запросов

```javascript
useEffect(() => {
  const id = setInterval(() => {}, 1000);

  return () => clearInterval(id);
}, []);
```

---

## 23. Зачем key, если не массив?

👉 Почти всегда key нужен ТОЛЬКО в списках

Если использовать вне списка:

- обычно ошибка архитектуры
- React не использует key там

---

## 24. Как сохранить ссылку функции?

Использовать:

```javascript
useCallback(() => {}, []);
```

или

```javascript
useRef(() => {});
```

---

## 25. Зачем state managers, если есть Context?

Context:

- не оптимизирован для частых обновлений
- вызывает rerender всех consumers

State managers (Redux, Zustand):

- более точечные обновления
- селекторы
- middleware
- devtools

👉 Context = DI (dependency injection)
👉 State manager = state architecture

---
