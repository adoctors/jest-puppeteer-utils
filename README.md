# Jest-puppeteer-utils

# Гайд по jest+puppeteer+jest-puppeteer. Кровь и пот

### Введение. Docs:

Для удобства я собрал библиотечку с необходимыми методами:

### Проблемные кейсы

#### Кейс #1: Кэш

По-умолчанию кэш хранится между запусками в headless режиме - ты просто не
сможешь отловить реквесты/респонсы при повторном запуске теста. Спускай кэш при
создании страниц (этот узд включен в функцию `go` библиотеки):

```javascript
const page = await browser.newPage()
await page.setCacheEnabled(false) 👏
```

#### Кейс #2: Отлов после перехода

Если есть желание отловить данные сразу после использования **goto**, используй
это:

```javascript
const [{ data, ok }] = await Promise.all([
  waitForNetworkAction(page, 'КУСОК_УРЛА'), // waitForNetworkAction функция библиотеки
  page.goto('УРЛ'),
]);
```

#### Кейс #3: Ждать div'чики

В доке предлагают селектить элемент так:

```javascript
const element = await page.$('#МОЙ_КРУТОЙ_СЕЛЕКТОР');
```

^^^ Этот код не всегда сможет поймать элемент. А этот сможет. Всегда:

```javascript
const renderedItemName = await page.waitForSelector('#МОЙ_КРУТОЙ_СЕЛЕКТОР', {
  timeout: 60000,
});
```

#### Кейс #4: Логин

Авторизацию лучше делать в отдельной странице, в beforeEach вне тестов

(в глобале -> `setupFilesAfterEnv: [... , '<rootDir>/jest/jest.globals.js']`):

```javascript
export const login = async () => {
  ///// какой-то код логина
};

beforeEach(async () => {
  await login();
});

// после можно подождать и закрыть браузер
afterEach(async () => {
  await page.waitFor(global.isDebugging() ? 3000 : 0);
  await browser.close();
});
```
