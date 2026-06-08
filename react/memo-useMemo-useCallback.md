# Мемоизация (memo, useCallback, useMemo)

## 1. Основные триггеры ререндера компонента

```jsx
// Триггеры ререндера (4 основных):

// 1. Изменение state внутри компонента
const [count, setCount] = useState(0);
setCount(5); // → ререндер

// 2. Изменение props от родителя
<Child name={name} />; // если name изменилось → Child ререндерится

// 3. Ререндер родительского компонента
// (даже если props не изменились, дочерний компонент по умолчанию тоже ререндерится)

// 4. Изменение значения в контексте (useContext)
const value = useContext(ThemeContext); // изменился Context → ререндер
```

---

## 2. React.memo

```jsx
// React.memo — HOC (Higher Order Component), который мемоизирует компонент
// Предотвращает лишние ререндеры, если props не изменились

// ========== Синтаксис ==========
import { memo } from "react";

const ChildComponent = memo(({ name, onClick }) => {
  console.log("Child render");
  return <button onClick={onClick}>{name}</button>;
});

// ========== Когда использовать memo? ==========
// ✅ Когда компонент часто ререндерится от родителя без необходимости
// ✅ Когда компонент тяжёлый по UI или логике
// ✅ Когда props редко меняются

// ❌ НЕ использовать для маленьких простых компонентов
// ❌ НЕ использовать если props каждый раз новые (пользы нет)

// ========== Важно: memo сравнивает props поверхностно ==========
// Для объектов/массивов/функций нужно использовать useCallback/useMemo

// Плохо: объект создаётся заново каждый раз
<Child config={{ theme: "dark" }} />;

// Хорошо: объект мемоизирован
const config = useMemo(() => ({ theme: "dark" }), []);
<Child config={config} />;
```

---

## 3. useCallback

```jsx
// useCallback — сохраняет ссылку на функцию между рендерами
// Без useCallback функция создаётся заново при каждом рендере

// ========== Синтаксис ==========
const handleClick = useCallback(() => {
  console.log("Clicked");
}, []); // массив зависимостей

// ========== Когда использовать useCallback (2 основных случая) ==========

// 1. Передача функции в memo-компонент
const Child = memo(({ onClick }) => {
  console.log("Child render");
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  // ✅ без useCallback — функция создаётся заново → Child ререндерится
  // ❌ const handleClick = () => console.log("click");

  // ✅ с useCallback — ссылка стабильна → Child НЕ ререндерится
  const handleClick = useCallback(() => {
    console.log("click");
  }, []);

  return <Child onClick={handleClick} />;
}

// 2. Функция используется в зависимостях useEffect
useEffect(() => {
  doSomething();
}, [handleClick]); // если handleClick меняется → эффект запускается заново

// ========== НЕ нужно использовать useCallback если ==========
// Нет проблем с производительностью
// Функция не передаётся в memo-компонент
// Функция не используется в зависимостях useEffect
```

---

## 4. useMemo

```jsx
// useMemo — сохраняет результат вычисления между рендерами

// ========== Синтаксис ==========
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]); // пересчитается только при изменении data

// ========== Когда использовать useMemo (2 основных случая) ==========

// 1. Тяжёлые вычисления (expensive calculations)
function ProductList({ products, filterText }) {
  // ❌ будет пересчитываться при каждом рендере
  // const filtered = products.filter(p => p.name.includes(filterText));

  // ✅ пересчитывается только при изменении products или filterText
  const filtered = useMemo(() => {
    return products.filter((p) => p.name.includes(filterText));
  }, [products, filterText]);

  return (
    <div>
      {filtered.map((p) => (
        <Product key={p.id} {...p} />
      ))}
    </div>
  );
}

// 2. Сохранение ссылки на объект/массив (для memo-компонентов)
const Child = memo(({ config }) => {
  console.log("Child render");
  return <div>{config.theme}</div>;
});

function Parent() {
  // ❌ объект создаётся заново → Child ререндерится
  // const config = { theme: "dark" };

  // ✅ ссылка стабильна → Child НЕ ререндерится
  const config = useMemo(() => ({ theme: "dark" }), []);

  return <Child config={config} />;
}

// ========== Нужно ли использовать useMemo всегда? ==========
// НЕТ! useMemo тоже имеет свою цену:
// - Занимает память
// - Усложняет код
// - Может ухудшить производительность при неправильном использовании

// Правильный подход:
// 1. Сначала пиши обычный код
// 2. Если есть реальная проблема производительности — добавляй useMemo
// 3. Измеряй результат (React DevTools Profiler)
```

---

## 5. Сравнение: memo, useCallback, useMemo

| Инструмент    | Что делает                                     | Когда использовать                                            |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------- |
| `memo`        | Мемоизирует компонент (предотвращает ререндер) | Тяжёлый компонент с редкими изменениями props                 |
| `useCallback` | Сохраняет ссылку на функцию                    | Передача функции в memo-компонент или в зависимости useEffect |
| `useMemo`     | Сохраняет результат вычисления                 | Тяжёлые вычисления или сохранение ссылки на объект/массив     |

```jsx
// Пример совместного использования:
import { memo, useCallback, useMemo, useState } from "react";

// Дочерний компонент обёрнут в memo
const ProductCard = memo(({ product, onBuy }) => {
  console.log("ProductCard render", product.id);
  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price}₽</p>
      <button onClick={() => onBuy(product.id)}>Купить</button>
    </div>
  );
});

function ProductList({ products }) {
  const [sortBy, setSortBy] = useState("name");

  // useMemo: сохраняем отсортированный список (тяжёлая операция)
  const sortedProducts = useMemo(() => {
    console.log("Сортировка...");
    return [...products].sort((a, b) => {
      if (sortBy === "price") return a.price - b.price;
      return a.name.localeCompare(b.name);
    });
  }, [products, sortBy]);

  // useCallback: сохраняем функцию для memo-компонента
  const handleBuy = useCallback((productId) => {
    console.log("Buy product:", productId);
  }, []);

  return (
    <div>
      <button onClick={() => setSortBy("name")}>По имени</button>
      <button onClick={() => setSortBy("price")}>По цене</button>

      {sortedProducts.map((product) => (
        <ProductCard key={product.id} product={product} onBuy={handleBuy} />
      ))}
    </div>
  );
}
```

---

## 6. Шпаргалка

```jsx
// === memo: оборачиваем компонент ===
const MyComponent = memo(({ prop }) => {
  return <div>{prop}</div>;
});

// === useCallback: оборачиваем функцию ===
const handleClick = useCallback(() => {
  doSomething();
}, [dependency]);

// === useMemo: оборачиваем вычисление ===
const result = useMemo(() => {
  return expensiveComputation(data);
}, [data]);
```

### Когда НЕ нужны оптимизации?

| Ситуация                               | Действие                 |
| -------------------------------------- | ------------------------ |
| Компонент маленький и простой          | Не используй memo        |
| Функция не передаётся в memo-компонент | Не используй useCallback |
| Нет тяжёлых вычислений                 | Не используй useMemo     |
| Нет проблем с производительностью      | Не используй ничего      |

### Правило: сначала сделай работающий код, потом оптимизируй при необходимости

---
