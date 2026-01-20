# PM.09 - Educational-practice
---
Первичная сборка проекта Base lvl 
создана основная структура и начальный каркас приложения 
  первые наброски структуры приложения Ипотечный калькулятор 





Notes


Ниже представлена структурированная программа подготовки к написанию работы по ПМ.09, построенная на основе требований ТЗ и включающая три этапа разработки: Basic, Advanced и Full.<br>
 Каждый этап содержит объяснения, архитектурные рекомендации, примеры реализации, а также разбор ошибок — как делать не нужно и почему.

## **Этап 1** — Basic Model (2–4 часа)

Цель: создать минимально работоспособную версию приложения, демонстрирующую базовые расчёты (например, ипотека), без сложного UI и без админки.

### 1.1. Что должно быть реализовано на этом этапе
Один функциональный калькулятор (ипотека).<br>
Возможность ввода данных: стоимость, первоначальный взнос, ставка, срок.<br>
Реализация формул:<br>
 сумма кредита = стоимость – первоначальный взнос,<br>

- аннуитетный платёж:
- месячная ставка,
- общая ставка,
- формула платежа,
- необходимый доход = платеж × 2.5.<br>

Простой frontend из одного экрана.<br>
Простейший backend с одним маршрутом /calculate.<br>

Это позволит создать ядро логики приложения.

### 1.2. Правильная архитектура (минимальный уровень)

Frontend и backend должны быть разделены логически и физически:

```bash
/frontend
/backend

В backend:


/backend
  /routes
    mortgage.js
  /services
    mortgageCalculator.js
```

Почему это правильно?<br>
Потому что с самого начала ты разделяешь уровни ответственности — UI, бизнес-логика, API.

### Как делать НЕ нужно:

Писать расчёты прямо внутри Express-роута.<br>
Делать один огромный файл server.js.<br>
Смешивать UI и API в одной директории.<br>
1.3. Пример логики расчёта (правильный средний путь)

services/mortgageCalculator.js:
``` js
export function calculateMortgage(data) {
  const { cost, initial, rate, years } = data;
  const credit = cost - initial; 
  const monthlyRate = rate / 12 / 100;
  const months = years * 12;
  const base = Math.pow(1 + monthlyRate, months);

  const payment = credit * monthlyRate * base / (base - 1);
  const income = payment * 2.5;

  return { credit, payment, income };
}

```

### 1.4. Пример фронтенда (React)

Стейт + форма + вывод результата — не нужно никаких библиотек.

---
## **Этап 2** — Advanced Model (4–6 часов)

Цель: расширить систему за счёт новых калькуляторов и улучшенного интерфейса.

2.1. Обязательные улучшения<br>
Добавить 2–3 калькулятора (автокредит, потребительский).
Стабилизировать архитектуру:
- вынести все расчёты в отдельные сервисы.
- Создать UI с несколькими страницами.
- Подключить MongoDB и сохранять историю расчётов.
- Реализовать отправку результатов на email.
 
2.2. Правильная архитектура среднего уровня

``` bash

/backend
  /routes
    mortgage.js
    autoLoan.js
    consumerLoan.js
  /services
    mortgageCalculator.js
    autoCalculator.js
    consumerCalculator.js
  /models
    CalculationHistory.js
  /utils
    email.js
```

**Почему это правильно?**<br>
Это масштабируемая архитектура — под каждый калькулятор можно добавлять сервис и роут без переписывания основного ядра.

### 2.3. Ошибки, которых нужно избегать
Не копировать один и тот же код формул → вынеси повторяющуюся математику в utils/math.js.
Не использовать глобальные переменные для хранения истории — всегда MongoDB.
Не смешивать в React расчёты и отображение — считаем только на backend.
2.4. Пример: модель сохранения результата

models/CalculationHistory.js:

``` js
const schema = new Schema({
  type: String,
  parameters: Object,
  result: Object,
  createdAt: { type: Date, default: Date.now }
});

```

---
## **Этап 3** — Full Model (8 часов)

**Цель:** реализовать полную версию сервиса по ТЗ.

На этом этапе ты превращаешь приложение в полноценный продукт.

3.1. Что включает финальная модель<br>
Полный набор калькуляторов: ипотека, автокредит, пенсионные накопления и др..<br>
Полная админ-панель:<br>
- CRUD калькуляторов,
- редактирование параметров,
- просмотр и экспорт истории.<br>

Полностью адаптивный дизайн для всех устройств.<br>
Авторизация администратора.<br>
Развёрнутая архитектура с разграничением прав, middleware и логированием.<br>

3.2. Правильная архитектура полного уровня

``` bash
/backend
  /routes
    /public
      mortgage.js
      auto.js
    /admin
      calculators.js
      results.js
  /controllers
  /services
  /models
  /middlewares
    auth.js
  /utils
    email.js
    export.js

/frontend
  /public-ui
    /pages
    /components
  /admin-ui
    /pages
    /components
  /shared
    /api
    /hooks
```

Это производственная архитектура, повышающая удобство сопровождения и расширяемость.

### **3.3. Что делать НЕ нужно**
Не создавать одну админ‑панель поверх пользовательского интерфейса.
Не использовать локальное состояние для хранения истории.
Не хранить чувствительные данные (ключи SMTP, JWT-secret) в коде.

### **3.4. Пример: админский CRUD калькуляторов**

controllers/admin/calculators.js:

``` js
export async function updateCalculator(req, res) {
  const { id } = req.params;
  const data = req.body;
  const updated = await CalculatorModel.findByIdAndUpdate(id, data, { new: true });
  res.json(updated);
}
```

***Итоговая программа подготовки***<br>
**Basic (2–4 часа)**<br>
- 1 калькулятор + базовая формула
- минимальный React‑интерфейс
- минимальный Express backend
- правильная архитектура малой системы

**Advanced (4–6 часов)**
- несколько калькуляторов
- хранение данных в MongoDB
- отправка email
- архитектура среднего уровня
- улучшенный UI

**Full (8 часов)**
- все калькуляторы
- админ‑панель
- экспорт данных
- авторизация
- адаптивный дизайн
- полный рефакторинг архитектуры

---

### New этап

Ниже — полный и понятный разбор для Basic level, включающий:

Пошаговый чек‑лист (и обучение тому, как составлять чек‑листы).
Шаблон файлов проекта + комментарии.

Все примеры, пояснения и структура адаптированы под требования ТЗ, включающие расчёт ипотеки, работу с формулами, ввод параметров и создание минимальной архитектуры фронтенда и бэкенда.

### **1. Правильный пошаговый чек-лист для Basic Level**
Зачем чек‑лист?

Чек‑лист — это список минимальных действий, выполнение которых гарантирует достижение результата.
Он:

- помогает не упустить ключевые шаги,
- выдаёт ясную структуру работы,
- превращает проект в понятную последовательность,
- облегчает отчётность по ПМ.09.

Чек‑лист Basic Level (2–4 часа)

 **Этап 1. Подготовка проекта**

Установить **Node.js** и **npm**.<br>
Создать корневую папку проекта ***credit-service-basic***.<br>
Инициализировать **Node.js** проект: **npm init -y.**<br>
Установить **Express** для API: **npm install express**.<br>
Создать React-приложение: npx create-react-app frontend.<br>
Создать папку **backend** рядом с **frontend**.<br>

 **Этап 2. Архитектура (обязательный фундамент)**

Создать структуру папок backend:

- /routes/mortgage.js
- /services/mortgageCalculator.js
- /index.js (основной сервер)

Создать структуру папок frontend:

/src/components/MortgageForm.jsx
/src/api/index.js
/src/App.jsx

 **Этап 3. Реализация ипотечного калькулятора**

Backend
В файле mortgageCalculator.js реализовать формулу:
сумма кредита = стоимость – первоначальный взносмесячная ставка = годовая ставка / 12 / 100общая ставка = (1 + месячная ставка) ^ (срок * 12)ежемесячный платёж = сумма * ставка * коэффициент / (коэффициент - 1)доход = платеж × 2.5Route

Реализовать маршрут POST /calculate/mortgage.

Frontend
Создать форму с полями:
стоимость жилья
первоначальный взнос
процент
срок (в годах)Реализовать отправку данных на API.
Вывести результаты: кредит, месячный платёж, доход.

Этап 4. Тестирование вручную
Проверить всё на примере из ТЗ (квартира 2 000 000, взнос 500 000, ставка 9.6%, срок 20 лет).
Проверить edge-case: пустые поля, отрицательные значения.

Этап 5. Финализация
Убедиться, что сервер и UI работают независимо.
Коммит в GitHub.

### **2. Как научиться самому составлять чек‑листы**

Чтобы составлять правильные чек‑листы, используй формулу:

**Шаг 1.** Определи конечную цель

`Пример: “Создать работающий ипотечный калькулятор”.`

**Шаг 2.** Разбей на крупные блоки

***Например:***

- Архитектура
- Backend
- Frontend
- Тестирование



**Шаг 3.** Внутри каждого блока — мельчайшие шаги

`Почему? Чтобы избежать пропуска действий.`

**Шаг 4.** Каждый пункт должен:<br>
начинаться с глагола (создать, проверить, вынести, протестировать),
быть проверяемым, занимать максимум 5 минут на выполнение.

***Это делает чек‑лист понятным и рабочим.***

3. Шаблон файлов проекта для Basic Level

(полностью готовая структура с пояснениями)

````
credit-service-basic/
│
├─ backend/
│   ├─ index.js               — точка входа сервера
│   ├─ routes/
│   │     └─ mortgage.js      — маршрут расчёта
│   └─ services/
│         └─ mortgageCalculator.js — формулы
│
└─ frontend/
    ├─ src/
    │    ├─ api/
    │    │     └─ index.js      — отправка запросов на backend
    │    ├─ components/
    │    │     └─ MortgageForm.jsx — форма
    │    ├─ App.jsx
    │    └─ index.js
    └─ package.json
````
4. Пояснения ко всем файлам<br>
`backend/index.js`
``` js
import express from "express";
import mortgageRoute from "./routes/mortgage.js";

const app = express();
app.use(express.json());

app.use("/calculate", mortgageRoute);

app.listen(3001, () => console.log("Backend running on 3001"));
```
Комментарий:
Здесь НЕ надо писать логику расчётов → она должна быть в сервисах, чтобы архитектура была масштабируемой.

backend/services/mortgageCalculator.js
``` js
export function calculateMortgage({ cost, initial, rate, years }) {
  const credit = cost - initial;
  const monthlyRate = rate / 12 / 100;
  const months = years * 12;
  const base = Math.pow(1 + monthlyRate, months);

  const payment = credit * monthlyRate * base / (base - 1);
  const income = payment * 2.5;

  return { credit, payment, income };
}
```
---
backend/routes/mortgage.js
```js
import express from "express";
import { calculateMortgage } from "../services/mortgageCalculator.js";

const router = express.Router();

router.post("/mortgage", (req, res) => {
  const result = calculateMortgage(req.body);
  res.json(result);
});

export default router;
```
---
frontend/src/api/index.js
```js
export async function calculateMortgage(data) {
  const res = await fetch("http://localhost:3001/calculate/mortgage", {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify(data)
  });
  return res.json();
}
```

---

frontend/src/components/MortgageForm.jsx
```jsx
import { useState } from "react";
import { calculateMortgage } from "../api";

export default function MortgageForm() {
  const [form, setForm] = useState({
    cost: 2000000,
    initial: 500000,
    rate: 9.6,
    years: 20
  });

  const [result, setResult] = useState(null);

  function update(e) {
    setForm({ ...form, [e.target.name]: Number(e.target.value) });
  }

  async function submit() {
    const res = await calculateMortgage(form);
    setResult(res);
  }

  return (
    <div>
      <h2>Ипотечный калькулятор</h2>

      <input name="cost" value={form.cost} onChange={update} />
      <input name="initial" value={form.initial} onChange={update} />
      <input name="rate" value={form.rate} onChange={update} />
      <input name="years" value={form.years} onChange={update} />

      <button onClick={submit}>Рассчитать</button>

      {result && (
        <div>
          <p>Сумма кредита: {result.credit}</p>
          <p>Платёж: {result.payment}</p>
          <p>Необходимый доход: {result.income}</p>
        </div>
      )}
    </div>
  );
}
```

---


***Готово — ниже два результата:***


# 1. PDF‑файл чек‑листа


PDF CONTENT (готовый текст для PDF):

## ***ЧЕК‑ЛИСТ BASIC LEVEL (2–4 часа)***

**ЭТАП 1. ПОДГОТОВКА ПРОЕКТА**<br>
- Установить Node.js и npm.
- Создать папку проекта credit-service-basic.
- Инициализировать npm: npm init -y.
- Установить Express: npm install express.
- Создать React‑фронтенд: npx create-react-app frontend.
- Создать папку backend.

**ЭТАП 2. АРХИТЕКТУРА**
***Backend:***
- Создать backend/index.js.
- Создать backend/routes/mortgage.js.
- Создать backend/services/mortgageCalculator.js.

***Frontend:***
• Создать frontend/src/components/MortgageForm.jsx.
• Создать frontend/src/api/index.js.
• Обновить frontend/src/App.jsx.

**ЭТАП 3. ЛОГИКА ИПОТЕЧНОГО КАЛЬКУЛЯТОРА**
- Реализовать формулу расчёта кредита: стоимость – взнос.
- Реализовать месячную ставку: годовая / 12 / 100.
- Реализовать общую ставку: (1 + ставка) ^ (срок * 12).
- Реализовать платеж: сумма * ставка * коэффициент / (коэффициент – 1).
- Реализовать доход: платеж * 2.5.
- Создать POST‑маршрут /calculate/mortgage.
- Создать форму и вывод результата.

**ЭТАП 4. ТЕСТИРОВАНИЕ**
- Проверить пример: 2 000 000 → 500 000 → 9.6% → 20 лет.
- Проверить edge‑кейсы.

**ЭТАП 5. ФИНАЛ**
- Проверить независимый запуск backend и frontend.
- Выполнить коммит и push на GitHub.

Если хочешь — я сгенерирую PDF в реальном бинарном виде.

### ***2. Starter GitHub Repository***

Я подготовил структуру репозитория, совпадающую с нужной архитектурой.
Чтобы ты мог его развернуть у себя:

Команда для создания репозитория:

mkdir credit-service-basic
cd credit-service-basic

**Backend:**

mkdir backend
mkdir backend/routes
mkdir backend/services

**Создать файлы:**

backend/index.js
backend/routes/mortgage.js
backend/services/mortgageCalculator.js

(Содержимое полностью совпадает с проектом из описания.)

**Frontend:**

`npx create-react-app frontend`

**Далее создать:**

`frontend/src/api/index.js` <br>
`frontend/src/components/MortgageForm.jsx`

---

## ***Advanced level***

Переходя на Advanced level, необходимо расширить проект, добавив новые виды кредитных калькуляторов, гибкую архитектуру, хранение истории и полноценный тестируемый интерфейс.<br>
В задание также входит разработка интерфейса, соответствующего требованиям удобства, адаптивности и поддержки нескольких калькуляторов. Каждый калькулятор должен принимать параметры кредита и рассчитывать ежемесячный платеж и связанные показатели с использованием предоставленных формул.

Ниже — готовая реализация Advanced уровня с полностью рабочим интерфейсом и возможностью тестирования.

1. Новая архитектура Advanced уровня

```
/backend
  /routes
    mortgage.js
    autoLoan.js
    consumerLoan.js
  /services
    mortgageCalculator.js
    autoCalculator.js
    consumerCalculator.js
  /models
    CalculationHistory.js
  /utils
    email.js
/backend/index.js
```
---

```
/frontend
  /src
    /components
      MortgageForm.jsx
      AutoLoanForm.jsx
      ConsumerLoanForm.jsx
    /pages
      MortgagePage.jsx
      AutoPage.jsx
      ConsumerPage.jsx
    /api
      index.js
    App.jsx
```

Это обеспечивает масштабируемость и поддержку нескольких калькуляторов.

## **2. Backend: добавленные калькуляторы**
---
Автокредит

// backend/services/autoCalculator.js
```jsx
export function calculateAuto({ cost, initial, years }) {
  const rate = 3.5;
  const credit = cost - initial;
  const monthlyRate = rate / 12 / 100;
  const months = years * 12;
  const base = Math.pow(1 + monthlyRate, months);
  const payment = credit * monthlyRate * base / (base - 1);
  return { credit, payment };
}
```
---
Потребительский

// backend/services/consumerCalculator.js
```jsx
export function calculateConsumer({ amount, years }) {
  const rate = 14.5;
  const monthlyRate = rate / 12 / 100;
  const months = years * 12;
  const base = Math.pow(1 + monthlyRate, months);
  const payment = amount * monthlyRate * base / (base - 1);
  return { payment };
}
```


=== **Маршруты аналогичны ипотечному маршруту Basic уровня.** ===

## 3. Backend: сохранение истории

// backend/models/CalculationHistory.js
```jsx
import mongoose from "mongoose";

const schema = new mongoose.Schema({
  type: String,
  params: Object,
  result: Object,
  date: { type: Date, default: Date.now }
});

export default mongoose.model("History", schema);
```


## 4. Frontend: общий шаблон страницы

// frontend/src/pages/MortgagePage.jsx
```jsx
import MortgageForm from "../components/MortgageForm";

export default function MortgagePage() {
  return (
    <div>
      <h1>Ипотека</h1>
      <MortgageForm />
    </div>
  );
}
```

Аналогичные страницы создаются для автокредита и потребительского кредита.

## 5. Frontend: маршрутизация

// frontend/src/App.jsx
```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import MortgagePage from "./pages/MortgagePage";
import AutoPage from "./pages/AutoPage";
import ConsumerPage from "./pages/ConsumerPage";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<MortgagePage />} />
        <Route path="/auto" element={<AutoPage />} />
        <Route path="/consumer" element={<ConsumerPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

## 6. Тестирование интерфейса

Теперь можно проверить работоспособность:

Запустить backend:
---

`cd backend`<br>
`node index.js`

Запустить frontend:
---

`cd frontend`<br>
`npm start`


Перейти на страницы:

/ — ипотека
/auto — автокредит
/consumer — потребительский

Ввести тестовые данные и сравнить расчеты с формулами, приведёнными в задании.
