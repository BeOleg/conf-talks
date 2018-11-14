# Как работать с ошибками в GraphQL

-----

## В GraphQL существует<br/>4 группы ошибок

- Фатальные ошибки
- Ошибки валидации
- Runtime ошибки
- Пользовательские ошибки

-----

### Когда клиент отправляет GraphQL-запрос по HTTP, то первым его встречает код вашего сервера.

-----

## 1. Фатальные ошибки

- кончилась память
- забыли установить пакет
- грубая синтаксическая ошибка в коде
- ошибки сервера

-----

## В таком случае вы получаете

## 500 Internal Server Error <!-- .element: class="text-red" -->

#### <br/>Ваш запрос даже не дошел до пакета `graphql`

-----

### Если фатальных ошибок нет, <br/>то GraphQL-запрос передается в пакет `graphql`.

-----

### Пакет `graphql` возвращает ответ по спецификации

```js
{
  // для возврата данных
  data: {},

  // для возврата ошибок, массив между прочим 😳
  errors: [...],

  // объект для пользовательских данных, сюда пихайте что хотите
  extensions: {}
}

```

GraphQL возвращает массив ошибок,
<br/>ведь в одном запросе может быть много ошибок 😈

-----

### Первым делом пакет `graphql`, <br/>проводит парсинг и валидацию<br/>GraphQL-запроса.

- проверяет корректность запроса
- проверяет на соответствие с вашей GraphQL-схемой
- проверяет корректность переменных

-----

## 2. Ошибки валидации

- ошибка невалидного GraphQL-запроса
- запросили несуществующее поле
- не передали обязательный аргумент
- не передали переменную

-----

### Ошибка при запросе несуществующего поля

```js
{
  errors: [
    {
      message: 'Cannot query field "wrong" on type "Query".',
      locations: [{ line: 3, column: 11 }],
    },
  ],
}

```

При этом сервер возвращает код 200 и это нормально.

-----

### Забыли передать переменную

```js
{
  errors: [
    {
      message: 'Variable "$q" of required type "String!" was not provided.',
      locations: [{ line: 2, column: 16 }],
    },
  ],
}

```

-----

### Если нет ошибок парсинга и валидации, то пакет `graphql` начинает выполнять запрос вызывая resolve-методы.

-----

## 3. Runtime ошибки в resolve-методах

- throw new Error("")
- undefined is not a function (юзайте Flowtype или TypeScript)
- ошибка невалидного значения в return

-----

### Если возникла ошибка, то

- обработка ветки графа приостанавливается (вложенные resolve-методы вызываться не будут)
- на месте элемента, где произошла ошибка возвращается `null`
- ошибка добавляется в массив `errors`

#### <br/>☝️ НО при этом соседние ветки продолжают работать

-----

##### Пример: throw new Error("")

```js
const searchResolver = (_, args) => {
  if (!args.q) throw new Error('missing q');
  return { text: args.q };
};

```

```graphql
query {
  s1: search(q: "ok") { text }
  s2: search { text }
  s3: search(q: "good") { text }
}

```

```js
{
  errors: [{
    message: 'missing q',
    locations: [{ line: 4, column: 11 }],
    path: ['s2']
  }],
  data: {
    s1: { text: 'ok' },
    s2: null,
    s3: { text: 'good' }
  },
}

```

<span class="fragment" data-code-block="1" data-code-focus="2" />
<span class="fragment" data-code-block="2" data-code-focus="3" />
<span class="fragment" data-code-block="3" data-code-focus="3,5,9" />

-----

##### Пример: ошибка невалидного значения в return

```js
const ooops = {
  type: new GraphQLList(GraphQLString),
  resolve: () => ['ok', { hey: 666 }],
};

```

```js
{
  errors: [
    {
      message: 'String cannot represent value: { hey: 666 }',
      locations: [{ line: 3, column: 11 }],
      path: ['ooops', 1],
    },
  ],
  data: {
    ooops: [
      'ok',
      null
    ]
  },
}

```

<span class="fragment" data-code-block="1" data-code-focus="2" />
<span class="fragment" data-code-block="1" data-code-focus="3" />
<span class="fragment" data-code-block="2" data-code-focus="4,6,12" />

-----

##### А еще есть `extensions` в ошибках, чтобы передать клиентам дополнительные данные об ошибке

```js
const searchResolver = () => {
  const e = new Error('WoW');
  e.extensions = { a: 'Additional error data for client' };
  throw e;
};

```

```js
{
  errors: [
    {
      message: 'WoW',
      locations: [{ line: 1, column: 9 }],
      path: ['search'],
      extensions: { a: 'Additional error data for client' },
    },
  ],
  data: { search: null },
}

```

<span class="fragment" data-code-block="1" data-code-focus="3" />
<span class="fragment" data-code-block="2" data-code-focus="7" />

-----

### Как работать с такими ошибками на клиентской стороне?

Показать одну ошибку в модалке не проблема! <!-- .element: class="fragment" -->

-----

### А если ошибок две?

### А если ошибки надо показывать в разных частях приложения? <!-- .element: class="fragment" -->

### Как разобрать массив ошибок и нужные передать по дереву компонент? <!-- .element: class="fragment" -->

-----

### Куралесим велосипед на клиенте?

### 😈

### ✋ Не спешим! <br/> 💃 Есть красивое решение! <!-- .element: class="fragment" -->

-----

### Помните формат GraphQL-ответа?

```js
{
  // для возврата данных
  data: {},

  // для возврата массива ошибок
  errors: [...],

  // объект для пользовательских данных, сюда пихайте что хотите
  extensions: {}
}

```

<span>А что если передавать ошибки не только в `errors`,<br/> но и в `data` 🤔</span> <!-- .element: class="fragment" -->

-----

### Передавать в `data` не все ошибки подряд, а только...

-----

## 4. Пользовательские ошибки

- запись не найдена
- недостаточно прав для просмотра
- недоступно в вашем регионе
- пользователь забанен

-----

### Для передачи ошибок в `data`

### надо просто создать Типы ошибок:

<br/>

```js
type ErrorNoAccess {
  message: String
}

type ErrorNotFound {
  message: String
}

```

-----

### И клиенту возвращать

- либо запись с данными
- либо запись с пользовательской ошибкой

-----



-----

-----

-----

Подробнее про работу с ошибками в GraphQL [читаем тут](https://github.com/nodkz/conf-talks/tree/master/particles/graphql/errors/README.md)
