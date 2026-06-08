# Работа с DOM

## Вопросы для закрепления

### 1. Что такое document?

```js
// document — это глобальный объект, представляющий HTML-документ,
// загруженный в браузере. Он является точкой входа в DOM (Document Object Model)
// и позволяет управлять содержимым страницы.

// Пример:
console.log(document.title);      // получаем заголовок страницы
document.title = "Новый заголовок"; // меняем заголовок
```

### 2. Как обращаться к элементам?

```js
// ========== Методы поиска элементов ==========

// 2.1. getElementsByClassName — ищет элементы по имени класса (возвращает HTMLCollection)
const items = document.getElementsByClassName("item");
console.log(items[0]);

// 2.2. getElementById — ищет элемент по id (возвращает один элемент)
const header = document.getElementById("main-header");

// 2.3. querySelector — ищет ПЕРВЫЙ элемент, подходящий под CSS-селектор
const firstButton = document.querySelector(".btn");

// 2.4. querySelectorAll — ищет ВСЕ элементы по селектору (возвращает NodeList)
const allButtons = document.querySelectorAll(".btn");
allButtons.forEach(btn => console.log(btn));
```

### 3. Как изменять элемент в DOM?

```js
// ========== 3.1. Изменение стилей ==========
const box = document.querySelector(".box");
box.style.backgroundColor = "red";   // меняем фон
box.style.width = "200px";           // меняем ширину
box.style.display = "none";          // скрываем

// ========== 3.2. Работа с атрибутами ==========
const link = document.querySelector("a");
link.getAttribute("href");           // получить атрибут
link.setAttribute("href", "https://example.com"); // установить атрибут
link.removeAttribute("target");      // удалить атрибут

// ========== 3.3. Изменение содержимого ==========
const content = document.querySelector(".content");
content.innerHTML = "<strong>Жирный текст</strong>"; // вставляет HTML
content.innerText = "Обычный текст";  // вставляет ТОЛЬКО текст (безопаснее)
content.textContent = "Тоже текст";   // похож на innerText, но быстрее

// ========== 3.4. Добавление дочерних элементов ==========
const parent = document.querySelector(".parent");
const child = document.createElement("div");  // создаем элемент
child.textContent = "Новый дочерний элемент";
parent.appendChild(child);        // старый способ
// или
parent.append(child);             // современный способ
```

### 4. Метод addEventListener

```js
// addEventListener — метод для "прослушивания" событий на элементе

const button = document.querySelector("button");

// Синтаксис: элемент.addEventListener(событие, функция)

button.addEventListener("click", () => {
    console.log("Кнопка нажата!");
});

// Другие популярные события:
// - click (клик мыши)
// - input (ввод текста)
// - submit (отправка формы)
// - keydown (нажатие клавиши)
// - mouseenter (наведение мыши)

// Удаление обработчика (важно для оптимизации):
function handleClick() {
    console.log("Клик!");
}
button.addEventListener("click", handleClick);
button.removeEventListener("click", handleClick);
```

---

## Практические задачи с решениями

### Задача 1. Табы (вкладки)

**HTML:**
```html
<div class="tabs">
    <button class="tab" data-target="content1">Вкладка 1</button>
    <button class="tab" data-target="content2">Вкладка 2</button>
</div>
<div id="content1" class="content">Содержимое 1</div>
<div id="content2" class="content hidden">Содержимое 2</div>
```

**CSS:**
```css
.hidden { display: none; }
.content { padding: 20px; border: 1px solid #ccc; }
```

**Решение:**
```js
// script.js
const tabs = document.querySelectorAll(".tab");
const contents = document.querySelectorAll(".content");

tabs.forEach(tab => {
    tab.addEventListener("click", () => {
        // Получаем id контента из атрибута data-target
        const targetId = tab.getAttribute("data-target");
        
        // Скрываем все контенты
        contents.forEach(content => {
            content.classList.add("hidden");
        });
        
        // Показываем выбранный контент
        document.getElementById(targetId).classList.remove("hidden");
    });
});
```

---

### Задача 2. Todo список с удалением

**HTML:**
```html
<input id="taskInput" placeholder="Введите задачу" />
<button id="addTask">Добавить</button>
<ul id="taskList"></ul>
```

**Решение:**
```js
const taskInput = document.querySelector("#taskInput");
const addButton = document.querySelector("#addTask");
const taskList = document.querySelector("#taskList");

// Функция добавления задачи
function addTask() {
    const taskText = taskInput.value.trim();
    
    if (taskText === "") {
        alert("Введите текст задачи!");
        return;
    }
    
    // Создаем элементы
    const li = document.createElement("li");
    const deleteBtn = document.createElement("button");
    
    li.textContent = taskText;
    deleteBtn.textContent = "Удалить";
    deleteBtn.style.marginLeft = "10px";
    
    // Удаление при клике на кнопку
    deleteBtn.addEventListener("click", () => {
        li.remove();
    });
    
    li.appendChild(deleteBtn);
    taskList.appendChild(li);
    
    // Очищаем поле ввода
    taskInput.value = "";
}

addButton.addEventListener("click", addTask);

// Добавление по нажатию Enter
taskInput.addEventListener("keypress", (e) => {
    if (e.key === "Enter") {
        addTask();
    }
});
```

---

### Задача 3. Форма с валидацией email

**HTML:**
```html
<form id="form">
    <input id="email" type="email" placeholder="Введите email" />
    <span id="emailError" class="error" style="color: red;"></span>
    <button type="submit">Отправить</button>
</form>
```

**Решение:**
```js
const form = document.querySelector("#form");
const emailInput = document.querySelector("#email");
const emailError = document.querySelector("#emailError");

// Регулярное выражение для проверки email
function isValidEmail(email) {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
}

form.addEventListener("submit", (e) => {
    e.preventDefault(); // Отменяем отправку формы
    
    const email = emailInput.value.trim();
    
    if (!isValidEmail(email)) {
        emailError.textContent = "Введите корректный email (пример: name@domain.com)";
        return;
    }
    
    // Если всё корректно
    emailError.textContent = "";
    alert("Форма успешно отправлена!");
    form.reset(); // Очищаем форму
});
```

---

### Задача 4. Сортировка карточек по алфавиту

**HTML:**
```html
<button id="sort">Сортировать</button>
<div id="cards">
    <div class="card">Банан</div>
    <div class="card">Яблоко</div>
    <div class="card">Апельсин</div>
</div>
```

**Решение (исправленное):**
```js
const sortButton = document.querySelector("#sort");
const cardsContainer = document.querySelector("#cards");

sortButton.addEventListener("click", () => {
    // Получаем все карточки как массив
    const cards = Array.from(cardsContainer.querySelectorAll(".card"));
    
    // Сортируем по тексту
    const sortedCards = cards.sort((a, b) => {
        return a.textContent.localeCompare(b.textContent);
    });
    
    // Очищаем контейнер
    cardsContainer.innerHTML = "";
    
    // Добавляем отсортированные карточки обратно
    sortedCards.forEach(card => {
        cardsContainer.appendChild(card);
    });
});
```

---

### Задача 5. Динамический фильтр по списку

**HTML:**
```html
<input id="search" placeholder="Поиск..." />
<ul id="peopleList">
    <li>Анна</li>
    <li>Борис</li>
    <li>Виктор</li>
    <li>Галина</li>
</ul>
```

**Решение:**
```js
const searchInput = document.querySelector("#search");
const peopleList = document.querySelector("#peopleList");
const items = peopleList.querySelectorAll("li");

searchInput.addEventListener("input", (e) => {
    const searchText = e.target.value.toLowerCase();
    
    items.forEach(item => {
        const text = item.textContent.toLowerCase();
        
        if (text.includes(searchText)) {
            item.style.display = "";      // показываем
        } else {
            item.style.display = "none";  // скрываем
        }
    });
});
```

---

## Дополнительные задачи для практики

### Задача 6. Модальное окно

```html
<button id="openModal">Открыть модалку</button>
<div id="modal" class="modal hidden">
    <div class="modal-content">
        <span id="closeModal">&times;</span>
        <p>Привет, я модальное окно!</p>
    </div>
</div>
```

```css
.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.5);
}
.modal.hidden { display: none; }
.modal-content {
    background: white;
    margin: 15% auto;
    padding: 20px;
    width: 300px;
}
```

```js
const openBtn = document.querySelector("#openModal");
const modal = document.querySelector("#modal");
const closeBtn = document.querySelector("#closeModal");

openBtn.addEventListener("click", () => {
    modal.classList.remove("hidden");
});

closeBtn.addEventListener("click", () => {
    modal.classList.add("hidden");
});

modal.addEventListener("click", (e) => {
    if (e.target === modal) {
        modal.classList.add("hidden");
    }
});
```

---

### Задача 7. Счётчик с кнопками

```html
<span id="counter">0</span>
<button id="increment">+</button>
<button id="decrement">-</button>
<button id="reset">Сброс</button>
```

```js
let count = 0;
const counterSpan = document.querySelector("#counter");
const incrementBtn = document.querySelector("#increment");
const decrementBtn = document.querySelector("#decrement");
const resetBtn = document.querySelector("#reset");

function updateCounter() {
    counterSpan.textContent = count;
}

incrementBtn.addEventListener("click", () => {
    count++;
    updateCounter();
});

decrementBtn.addEventListener("click", () => {
    count--;
    updateCounter();
});

resetBtn.addEventListener("click", () => {
    count = 0;
    updateCounter();
});
```

---

## Шпаргалка по DOM-методам

| Что делаем | Метод |
|------------|-------|
| Найти элемент по id | `getElementById('id')` |
| Найти по классу | `getElementsByClassName('class')` |
| Найти по селектору (один) | `querySelector('.class')` |
| Найти по селектору (все) | `querySelectorAll('.class')` |
| Создать элемент | `createElement('div')` |
| Добавить в конец | `parent.appendChild(child)` |
| Удалить элемент | `element.remove()` |
| Изменить текст | `element.textContent = 'text'` |
| Изменить HTML | `element.innerHTML = '<p>'` |
| Добавить класс | `element.classList.add('class')` |
| Убрать класс | `element.classList.remove('class')` |
| Переключить класс | `element.classList.toggle('class')` |
| Установить атрибут | `element.setAttribute('src', 'img.jpg')` |
| Получить атрибут | `element.getAttribute('src')` |
| Добавить обработчик | `element.addEventListener('click', fn)` |

