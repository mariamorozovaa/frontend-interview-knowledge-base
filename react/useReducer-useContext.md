# useReducer, useContext, управление состоянием

## 1. useReducer: как работает и когда использовать?

```jsx
// useReducer — хук для управления сложным состоянием через функцию-редьюсер

// ========== Синтаксис ==========
const [state, dispatch] = useReducer(reducer, initialState);

// reducer — функция, которая описывает, как меняется состояние
function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "DECREMENT":
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
}

// dispatch — отправляет action
dispatch({ type: "INCREMENT" });
dispatch({ type: "DECREMENT", payload: 5 }); // с дополнительными данными

// ========== Когда использовать useReducer? ==========
// ✅ Состояние сложное (несколько полей, вложенность)
// ✅ Есть зависимые изменения (одно поле зависит от другого)
// ✅ Много useState (код становится грязным)
// ✅ Нужна централизация логики обновления
// ✅ Есть частые изменения состояния

// Примеры: формы, todo-листы, корзина, сложные UI состояния

// ========== useState vs useReducer ==========
// useState: простое состояние (число, строка, булево)
// useReducer: сложное состояние (объект с несколькими полями)
```

---

## 2. useContext: как работает и когда использовать?

```jsx
// useContext — передаёт данные через дерево компонентов без пропсов (prop drilling)

// ========== Синтаксис ==========
// 1. Создаём контекст
const UserContext = React.createContext();

// 2. Оборачиваем приложение (Provider)
function App() {
  const user = { name: "Alice", role: "admin" };
  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}

// 3. Используем в любом компоненте
function UserMenu() {
  const user = useContext(UserContext);
  return <div>Welcome, {user.name}!</div>;
}

// ========== Когда использовать useContext? ==========
// ✅ Глобальные данные (тема, язык, пользователь)
// ✅ Нужно избежать prop drilling (прокидывание через 5+ уровней)
// ✅ Данные, которые нужны многим компонентам

// Примеры: текущий пользователь, тема (dark/light), язык (i18n), auth состояние

// ========== Важно! ==========
// useContext НЕ управляет состоянием, он только передаёт данные
// Для управления состоянием + контекста используй useReducer + useContext
```

---

## 3. Плюсы сторонних библиотек (Redux, Zustand) vs useContext + useReducer

```jsx
// Проблемы useContext + useReducer:

// 1. Перерендеры — любое изменение вызывает перерендер ВСЕХ компонентов, использующих контекст
// 2. Нет оптимизации из коробки — нет селекторов, нет мемоизации
// 3. Плохая масштабируемость — трудно поддерживать в больших приложениях
// 4. Нет DevTools — нельзя смотреть историю изменений, time-travel debugging
// 5. Бойлерплейт — много кода (контексты, провайдеры, хуки)

// ========== Плюсы библиотек (Redux, Zustand, MobX) ==========

// 1. Оптимизация ререндеров — компоненты обновляются только при изменении нужных данных
const count = useSelector((state) => state.count); // только при изменении count

// 2. Селекторы — выборка части состояния
const filteredTodos = useSelector((state) => state.todos.filter((todo) => !todo.completed));

// 3. DevTools — логирование, time-travel, дебаг
// 4. Масштабируемость — удобны для больших приложений
// 5. Чистая архитектура — разделение: actions, reducers, store

// ========== Что выбрать? ==========
// Небольшое приложение: useContext + useReducer
// Среднее/большое приложение: Zustand / Redux Toolkit
```

---

## Практические задачи

### Задача 1. Переделать Counter с useState на useReducer

```jsx
import { useReducer } from "react";

// Редьюсер
const counterReducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return {
        ...state,
        count: state.count + state.step,
        history: [...state.history, `+${state.step}`],
      };
    case "DECREMENT":
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, `-${state.step}`],
      };
    case "RESET":
      return {
        ...state,
        count: 0,
        history: [],
      };
    case "SET_STEP":
      return {
        ...state,
        step: action.payload,
      };
    default:
      return state;
  }
};

const initialState = {
  count: 0,
  step: 1,
  history: [],
};

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  const { count, step, history } = state;

  return (
    <div>
      <h2>Count: {count}</h2>
      <input type="number" value={step} onChange={(e) => dispatch({ type: "SET_STEP", payload: Number(e.target.value) })} />
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
      <div>History: {history.join(", ")}</div>
    </div>
  );
}
```

---

### Задача 2. Избавиться от prop drilling через Context

```jsx
import { createContext, useContext, useState } from "react";

// 1. Создаём контекст
const UserContext = createContext();

// 2. Provider
function App() {
  const [user, setUser] = useState({ name: "Alice", role: "admin" });
  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}

// 3. Компоненты больше не принимают user через props
function Dashboard() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

function Sidebar() {
  return (
    <aside>
      <Navigation />
    </aside>
  );
}

function Navigation() {
  return (
    <nav>
      <UserMenu />
    </nav>
  );
}

function UserMenu() {
  const user = useContext(UserContext);
  return <div>Welcome, {user.name}!</div>;
}

function Content() {
  return (
    <main>
      <Header />
      <Article />
    </main>
  );
}

function Header() {
  const user = useContext(UserContext);
  return <h1>Dashboard for {user.role}</h1>;
}

function Article() {
  const user = useContext(UserContext);
  if (user.role !== "admin") {
    return <p>Access denied</p>;
  }
  return <p>Admin content here</p>;
}
```

---

### Задача 3. Форма регистрации с useReducer

```jsx
import { useReducer } from "react";

// Начальное состояние
const initialState = {
  username: "",
  email: "",
  password: "",
  confirmPassword: "",
  errors: {
    username: "",
    email: "",
    password: "",
    confirmPassword: "",
  },
};

// Редьюсер
const formReducer = (state, action) => {
  switch (action.type) {
    case "SET_FIELD":
      return {
        ...state,
        [action.field]: action.value,
        errors: { ...state.errors, [action.field]: "" },
      };
    case "SET_ERROR":
      return {
        ...state,
        errors: { ...state.errors, [action.field]: action.error },
      };
    case "RESET_FORM":
      return initialState;
    default:
      return state;
  }
};

// Функции валидации
const validate = (state) => {
  const errors = {
    username: "",
    email: "",
    password: "",
    confirmPassword: "",
  };
  let isValid = true;

  if (state.username.length < 3) {
    errors.username = "Минимум 3 символа";
    isValid = false;
  }
  if (!state.email.includes("@")) {
    errors.email = "Email должен содержать @";
    isValid = false;
  }
  if (state.password.length < 6) {
    errors.password = "Минимум 6 символов";
    isValid = false;
  }
  if (state.confirmPassword !== state.password) {
    errors.confirmPassword = "Пароли не совпадают";
    isValid = false;
  }

  return { errors, isValid };
};

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  const { username, email, password, confirmPassword, errors } = state;

  const handleSubmit = (e) => {
    e.preventDefault();
    const { errors: validationErrors, isValid } = validate(state);

    if (!isValid) {
      // Устанавливаем ошибки
      Object.keys(validationErrors).forEach((field) => {
        if (validationErrors[field]) {
          dispatch({ type: "SET_ERROR", field, error: validationErrors[field] });
        }
      });
      return;
    }

    console.log("Форма отправлена:", { username, email, password });
    dispatch({ type: "RESET_FORM" });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          placeholder="Username"
          value={username}
          onChange={(e) => dispatch({ type: "SET_FIELD", field: "username", value: e.target.value })}
        />
        {errors.username && <span style={{ color: "red" }}>{errors.username}</span>}
      </div>
      <div>
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => dispatch({ type: "SET_FIELD", field: "email", value: e.target.value })}
        />
        {errors.email && <span style={{ color: "red" }}>{errors.email}</span>}
      </div>
      <div>
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => dispatch({ type: "SET_FIELD", field: "password", value: e.target.value })}
        />
        {errors.password && <span style={{ color: "red" }}>{errors.password}</span>}
      </div>
      <div>
        <input
          type="password"
          placeholder="Confirm Password"
          value={confirmPassword}
          onChange={(e) => dispatch({ type: "SET_FIELD", field: "confirmPassword", value: e.target.value })}
        />
        {errors.confirmPassword && <span style={{ color: "red" }}>{errors.confirmPassword}</span>}
      </div>
      <button type="submit">Register</button>
    </form>
  );
}
```

---

### Задача 4. Todo приложение (useReducer + Context)

```jsx
// TodoContext.js
import { createContext, useContext, useReducer } from "react";

const TodoContext = createContext();

// Редьюсер
const todoReducer = (state, action) => {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };
    case "REMOVE_TODO":
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };
    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) => (todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo)),
      };
    case "SET_FILTER":
      return {
        ...state,
        filter: action.payload,
      };
    default:
      return state;
  }
};

const initialState = {
  todos: [],
  filter: "all", // 'all', 'active', 'completed'
};

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  return <TodoContext.Provider value={{ state, dispatch }}>{children}</TodoContext.Provider>;
}

export const useTodos = () => useContext(TodoContext);

// AddTodoForm.js
function AddTodoForm() {
  const [text, setText] = useState("");
  const { dispatch } = useTodos();

  const handleAdd = () => {
    if (text.trim()) {
      dispatch({
        type: "ADD_TODO",
        payload: {
          id: Date.now(),
          text,
          completed: false,
        },
      });
      setText("");
    }
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} onKeyPress={(e) => e.key === "Enter" && handleAdd()} />
      <button onClick={handleAdd}>Add</button>
    </div>
  );
}

// FilterButtons.js
function FilterButtons() {
  const { state, dispatch } = useTodos();

  return (
    <div>
      {["all", "active", "completed"].map((filter) => (
        <button
          key={filter}
          onClick={() => dispatch({ type: "SET_FILTER", payload: filter })}
          style={{
            fontWeight: state.filter === filter ? "bold" : "normal",
          }}>
          {filter}
        </button>
      ))}
    </div>
  );
}

// TodoList.js
function TodoList() {
  const { state } = useTodos();

  const filteredTodos = state.todos.filter((todo) => {
    if (state.filter === "active") return !todo.completed;
    if (state.filter === "completed") return todo.completed;
    return true;
  });

  return (
    <ul>
      {filteredTodos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

// TodoItem.js
function TodoItem({ todo }) {
  const { dispatch } = useTodos();

  return (
    <li>
      <input type="checkbox" checked={todo.completed} onChange={() => dispatch({ type: "TOGGLE_TODO", payload: todo.id })} />
      <span style={{ textDecoration: todo.completed ? "line-through" : "none" }}>{todo.text}</span>
      <button onClick={() => dispatch({ type: "REMOVE_TODO", payload: todo.id })}>X</button>
    </li>
  );
}

// App.js
function App() {
  return (
    <TodoProvider>
      <div>
        <h1>My Todos</h1>
        <AddTodoForm />
        <FilterButtons />
        <TodoList />
      </div>
    </TodoProvider>
  );
}
```

---

### Задача 5. Реализация useSelector и useDispatch (аналог Redux)

```jsx
// store.js
import { createContext, useContext, useReducer } from "react";

const StoreContext = createContext();

// Редьюсер
const appReducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "DECREMENT":
      return { ...state, count: state.count - 1 };
    case "ADD_TODO":
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), text: action.payload }],
      };
    default:
      return state;
  }
};

const initialState = {
  count: 0,
  todos: [],
};

export function StoreProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  return <StoreContext.Provider value={{ state, dispatch }}>{children}</StoreContext.Provider>;
}

// Хук useDispatch — возвращает dispatch
export function useDispatch() {
  const { dispatch } = useContext(StoreContext);
  return dispatch;
}

// Хук useSelector — принимает селектор и возвращает часть state
export function useSelector(selector) {
  const { state } = useContext(StoreContext);
  return selector(state);
}

// ========== Использование ==========

// CounterDisplay.js
function CounterDisplay() {
  const count = useSelector((state) => state.count);
  return <h2>Count: {count}</h2>;
}

// CounterButtons.js
function CounterButtons() {
  const dispatch = useDispatch();
  return (
    <div>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
    </div>
  );
}

// TodoInput.js
function TodoInput() {
  const [text, setText] = useState("");
  const dispatch = useDispatch();

  const handleAdd = () => {
    if (text.trim()) {
      dispatch({ type: "ADD_TODO", payload: text });
      setText("");
    }
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAdd}>Add</button>
    </div>
  );
}

// TodoList.js
function TodoList() {
  const todos = useSelector((state) => state.todos);
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}

// App.js
function App() {
  return (
    <StoreProvider>
      <CounterDisplay />
      <CounterButtons />
      <TodoInput />
      <TodoList />
    </StoreProvider>
  );
}
```

---

## Шпаргалка

### useReducer vs useState

| Характеристика       | useState            | useReducer    |
| -------------------- | ------------------- | ------------- |
| Простое состояние    | ✅                  | ❌ (overkill) |
| Сложное состояние    | ❌ (много setState) | ✅            |
| Зависимые поля       | ❌                  | ✅            |
| Централизация логики | ❌                  | ✅            |

### useContext + useReducer vs Redux

| Характеристика         | Context + useReducer | Redux / Zustand      |
| ---------------------- | -------------------- | -------------------- |
| Простота               | ✅                   | ❌                   |
| Оптимизация ререндеров | ❌                   | ✅                   |
| DevTools               | ❌                   | ✅                   |
| Масштабируемость       | ❌                   | ✅                   |
| Бойлерплейт            | Средний              | Большой (RTK меньше) |

---
