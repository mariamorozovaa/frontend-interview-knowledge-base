# Жизненный цикл, useEffect, useLayoutEffect

## 1. Методы жизненного цикла React компонента

### 1.1 Методы классового компонента

```jsx
// У классовых компонентов lifecycle-методы делятся на этапы:

// ========== Mount (монтирование) ==========
// constructor()      → инициализация state
// render()           → первый рендер
// componentDidMount() → после монтирования (запросы, подписки)

// ========== Update (обновление) ==========
// getDerivedStateFromProps() → синхронизация state с props
// shouldComponentUpdate()    → оптимизация (вернуть false, чтобы не ререндерить)
// render()                   → перерендер
// getSnapshotBeforeUpdate()  → получить данные до обновления DOM
// componentDidUpdate()       → после обновления

// ========== Unmount (размонтирование) ==========
// componentWillUnmount() → очистка (таймеры, подписки)

// ========== Error (ошибки) ==========
// getDerivedStateFromError() → показать fallback UI
// componentDidCatch()        → отправить ошибку в лог
```

### 1.2 Какие методы реализует useEffect?

```jsx
// useEffect в функциональных компонентах заменяет:

// 1. componentDidMount → useEffect(() => {}, [])
// 2. componentDidUpdate → useEffect(() => {}, [deps])
// 3. componentWillUnmount → return () => {} (cleanup)
```

### 1.3 ErrorBoundary компонент

```jsx
// ErrorBoundary — компонент (только классовый), который ловит ошибки в дочерних компонентах
// В функциональных компонентах НЕТ аналога

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true }; // показать fallback UI
  }

  componentDidCatch(error, errorInfo) {
    console.error("Ошибка:", error, errorInfo); // отправить в лог
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так</h1>;
    }
    return this.props.children;
  }
}

// Использование:
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>;
```

---

## 2. useEffect

### 2.1 Ключевые условия срабатывания

```jsx
// useEffect запускается:
// 1. После первого рендера (mount)
// 2. После изменения зависимостей (второй параметр)
// 3. После каждого рендера (если нет массива зависимостей)

useEffect(() => {
  console.log("Запуск после каждого рендера");
}); // ❌ без массива зависимостей

useEffect(() => {
  console.log("Запуск только при монтировании");
}, []); // ✅ пустой массив

useEffect(() => {
  console.log("Запуск при изменении count или name");
}, [count, name]); // ✅ зависимости
```

### 2.2 Что если не указывать второй параметр?

```jsx
// Если не указать массив зависимостей — эффект запускается после КАЖДОГО рендера

function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Запускается при каждом рендере");
    // 🚨 ОПАСНО: если здесь будет setState → бесконечный цикл!
    // setCount(count + 1); // БЕСКОНЕЧНЫЙ ЦИКЛ!
  });

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### 2.3 Cleanup (функция очистки)

```jsx
// return внутри useEffect — функция очистки (cleanup)
// Выполняется:
// 1. Перед повторным запуском эффекта (при изменении зависимостей)
// 2. При размонтировании компонента

useEffect(() => {
  const timer = setInterval(() => {
    console.log("Тик");
  }, 1000);

  // ✅ очистка таймера при размонтировании
  return () => {
    clearInterval(timer);
    console.log("Таймер очищен");
  };
}, []);

// ========== Практические применения cleanup ==========
// 1. Очистка таймеров: clearInterval(), clearTimeout()
// 2. Отписка от событий: removeEventListener()
// 3. Закрытие WebSocket: socket.close()
// 4. Отмена запросов: AbortController.abort()
// 5. Отписка от подписок (например, из Firebase)
```

### 2.4 useEffect срабатывает до или после отрисовки?

```jsx
// ❗ useEffect срабатывает ПОСЛЕ отрисовки и обновления DOM

// Порядок:
// 1. React вычисляет новый JSX
// 2. React обновляет DOM
// 3. Браузер рисует на экране (paint)
// 4. ✅ useEffect выполняется (после отрисовки)

// Это позволяет не блокировать отрисовку
```

### 2.5 useEffect с пустым массивом зависимостей

```jsx
// useEffect(() => {}, []) — запускается ОДИН раз после первого рендера

// Используется для:
// 1. Загрузка данных с сервера (fetch)
// 2. Подписка на события (addEventListener)
// 3. Инициализация сторонних библиотек
// 4. Установка таймеров
// 5. Логирование открытия страницы

useEffect(() => {
  fetchUsers().then(setUsers);
  startWebSocket();
  initAnalytics();

  return () => {
    // cleanup при размонтировании
    stopWebSocket();
  };
}, []);
```

---

## 3. useLayoutEffect

### 3.1 Когда срабатывает — до или после отрисовки?

```jsx
// useLayoutEffect срабатывает ПОСЛЕ обновления DOM, но ДО отрисовки браузером

// Порядок выполнения:
// 1. React вычисляет новый JSX
// 2. React обновляет DOM
// 3. ✅ useLayoutEffect выполняется (синхронно, БЛОКИРУЕТ paint)
// 4. Браузер рисует на экране (paint)
// 5. useEffect выполняется

// Визуально:
// [Render] → [DOM update] → [useLayoutEffect] → [Paint] → [useEffect]
```

### 3.2 В каких случаях использовать useLayoutEffect?

```jsx
// Когда нужно синхронно измерить DOM или изменить layout до того, как пользователь увидит мерцание

// 1. Измерение размеров элемента:
useLayoutEffect(() => {
  const width = ref.current.getBoundingClientRect().width;
  setWidth(width);
}, []);

// 2. Скролл до элемента при монтировании:
useLayoutEffect(() => {
  ref.current.scrollIntoView();
}, []);

// 3. Позиционирование тултипа/popup:
useLayoutEffect(() => {
  const { top, left } = buttonRef.current.getBoundingClientRect();
  setTooltipPosition({ top: top + 30, left });
}, []);

// 4. Анимация, которая не должна дёргаться
```

### 3.3 Что будет при долгих вычислениях в useLayoutEffect?

```jsx
// ❌ ОПАСНО: тяжёлый код в useLayoutEffect

useLayoutEffect(() => {
  // Тяжёлые вычисления (например, обработка 10000 элементов)
  for (let i = 0; i < 100000000; i++) {}
  setData(processed);
}, []);

// Результат:
// 1. Браузер заморозится
// 2. Пользователь увидит лаг и зависание
// 3. UI не будет отзывчивым

// ✅ Решение: использовать useEffect для тяжёлых операций
```

### 3.4 Сравнение useEffect и useLayoutEffect

| Характеристика                  | useEffect                               | useLayoutEffect                     |
| ------------------------------- | --------------------------------------- | ----------------------------------- |
| Время выполнения                | После отрисовки (асинхронно)            | После DOM, до отрисовки (синхронно) |
| Блокирует paint?                | Нет                                     | Да                                  |
| Когда использовать              | Большинство случаев (запросы, подписки) | Измерение DOM, позиционирование     |
| Видит ли пользователь мерцание? | Да (может быть фликер)                  | Нет                                 |

```jsx
// Правило: ПО УМОЛЧАНИЮ ИСПОЛЬЗУЙ useEffect
// useLayoutEffect нужен только в редких случаях, когда есть мерцание
```

---

## Практические задачи

### Задача 1. Сколько раз сработает useEffect?

```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("useEffect");
  }); // ❌ без массива зависимостей

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

// Ответ:
// useEffect сработает при КАЖДОМ рендере:
// 1. При монтировании (1 раз)
// 2. При каждом клике на кнопку (каждый клик → ререндер)

// Решение (сработает один раз):
useEffect(() => {
  console.log("useEffect");
}, []); // ✅ пустой массив
```

### Задача 2. Порядок useLayoutEffect и useEffect

```jsx
function App() {
  useLayoutEffect(() => {
    console.log("useLayoutEffect");
  }, []);

  useEffect(() => {
    console.log("useEffect");
  }, []);

  return <div>Приложение</div>;
}

// Вывод в консоль:
// useLayoutEffect
// useEffect

// Почему: useLayoutEffect выполняется после обновления DOM, но до paint.
// useEffect выполняется после paint.
```

### Задача 3. Будет ли работать useEffect с обычной переменной?

```jsx
let count = 0;

function App() {
  useEffect(() => {
    console.log("update count", count);
  }, [count]);

  return (
    <button
      onClick={() => {
        count = count + 1;
      }}>
      click
    </button>
  );
}

// Ответ:
// ❌ useEffect НЕ будет срабатывать
// Причина: count — обычная переменная, изменение НЕ вызывает ререндер
// React не знает об изменении, поэтому useEffect не проверяет зависимости

// ✅ Решение: использовать useState
```

### Задача 4. Резиновый блок (адаптация к размеру окна)

```jsx
function App() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth / 3,
        height: window.innerHeight / 3,
      });
    }

    handleResize(); // установить при монтировании
    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return (
    <div
      style={{
        width: size.width,
        height: size.height,
        background: "green",
      }}>
      <p>
        Размер окна: {size.width} x {size.height}
      </p>
    </div>
  );
}
```

### Задача 5. Круг, который следит за мышкой

```jsx
function App() {
  const [pos, setPos] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMouseMove(e) {
      setPos({ x: e.clientX, y: e.clientY });
    }

    window.addEventListener("mousemove", handleMouseMove);

    return () => {
      window.removeEventListener("mousemove", handleMouseMove);
    };
  }, []);

  return (
    <div
      style={{
        position: "absolute",
        top: pos.y,
        left: pos.x,
        width: 40,
        height: 40,
        borderRadius: "50%",
        background: "red",
        transition: "top 0.1s linear, left 0.1s linear",
        pointerEvents: "none",
      }}
    />
  );
}
```

### Задача 6. Загрузка списка пород собак

```jsx
function App() {
  const [isLoading, setIsLoading] = useState(false);
  const [breeds, setBreeds] = useState([]);

  useEffect(() => {
    async function fetchBreeds() {
      try {
        setIsLoading(true);
        const response = await fetch("https://dog.ceo/api/breeds/list/all");

        if (!response.ok) {
          throw new Error("Ошибка загрузки");
        }

        const result = await response.json();
        setBreeds(Object.keys(result.message));
      } catch (error) {
        console.error(error);
      } finally {
        setIsLoading(false);
      }
    }

    fetchBreeds();
  }, []);

  if (isLoading) {
    return <div>Loading...</div>;
  }

  return (
    <ul>
      {breeds.map((breed) => (
        <li key={breed}>{breed}</li>
      ))}
    </ul>
  );
}
```

---

## Шпаргалка

### useEffect зависимости

| Массив зависимостей     | Когда запускается                        |
| ----------------------- | ---------------------------------------- |
| `useEffect(fn)`         | После КАЖДОГО рендера                    |
| `useEffect(fn, [])`     | Один раз (при монтировании)              |
| `useEffect(fn, [a, b])` | При монтировании + при изменении a или b |

### useEffect vs useLayoutEffect

| useEffect                    | useLayoutEffect                             |
| ---------------------------- | ------------------------------------------- |
| После отрисовки (асинхронно) | После DOM, до отрисовки (синхронно)         |
| Не блокирует paint           | Блокирует paint                             |
| Использовать по умолчанию    | Только для измерений DOM и позиционирования |

### Cleanup (возвращаемая функция)

```jsx
useEffect(() => {
  // подписка
  const subscription = subscribe();

  // ✅ отписка при размонтировании
  return () => {
    subscription.unsubscribe();
  };
}, []);
```

---
