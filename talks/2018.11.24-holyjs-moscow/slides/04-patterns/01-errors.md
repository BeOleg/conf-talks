# Как работать с ошибками в GraphQL

#### Выступления:
- 27 ноября 2018: [http://bit.ly/rambler-graphql](http://bit.ly/rambler-graphql)

-----

## В GraphQL я бы выделил<br/>4 группы ошибок

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

### GraphQL-ответ через HTTP <br />всегда возвращает `код 200` ☝️

- GraphQL может вернуть много разных ошибок
- GraphQL не привязан к HTTP-протоколу

-----

### Забудьте про пачку HTTP-кодов, если вы пришли из мира RESTfull API

-----

### В мире GraphQL с HTTP всего один код `200`.

#### Ну и код `500`, если сервер помер.

-----

### Что происходит когда прилетает <br/>GraphQL-запрос на сервер?

-----

### Первым делом пакет `graphql`, <br/>проводит парсинг и валидацию<br/>GraphQL-запроса.

-----

## 2. Ошибки валидации

- ошибка невалидного GraphQL-запроса
- не передали переменную
- проверяет на соответствие с вашей GraphQL-схемой
  - запросили несуществующее поле
  - не передали обязательный аргумент
  - передали неверный тип для аргумента

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

#### Продолжим пример и сделаем так:

```diff
const ooops = {
-  type: new GraphQLList(GraphQLString),
+  type: new GraphQLList(new GraphQLNonNull(GraphQLString)),
  resolve: () => ['ok', { hey: 666 }],
};

```

#### GraphQL чуть-чуть звереет, т.к. теперь не может вернуть массив с null значением

```diff
{
  errors: [
    {
      message: 'String cannot represent value: { hey: 666 }',
      locations: [{ line: 3, column: 11 }],
      path: ['ooops', 1],
    },
  ],
  data: {
-    ooops: [ 'ok', null ]
+    ooops: null
  },
}

```

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
      extensions: { a: 'Additional error data for client' }, // <-- ☝️
    },
  ],
  data: { search: null },
  extensions: {} // <-- это экстеншн на глобальном уровне
}

```

<span class="fragment" data-code-block="1" data-code-focus="3" />
<span class="fragment" data-code-block="2" data-code-focus="7" />
<span class="fragment" data-code-block="2" data-code-focus="11" />

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

- недостаточно прав для просмотра
- недоступно в вашем регионе
- пользователь забанен
- необходимо оплатить контент
- и т.п.

-----

### У пользовательских ошибок есть одно свойство — их надо показать пользователю, чтоб он дальше что-то сделал.

-----

### Для передачи ошибок в `data`

### надо просто создать Типы ошибок:

<br/>

```js
type VideoInProgressProblem {
  estimatedTime: Int
}

type VideoNeedBuyProblem {
  price: Int
}

type VideoApproveAgeProblem {
  minAge: Int
}

```

-----

### И клиенту возвращать

- либо запись с данными `Video`
- либо запись с пользовательской ошибкой `Video*Problem`

-----

### Для этого необходимо использовать <br />Union-тип

```js
const VideoResultType = new GraphQLUnionType({
  // Даем имя типу.
  // Здорово если если вы выработаете конвенцию в своей команде
  // и к таким Union-типам будите добавлять суффикс Result
  name: 'VideoResult',

  // как хорошие бекендеры добавляем какое-нибудь описание
  description: 'Returns Video or some Problem object',

  // объявляем типы через массив, которые могут быть возвращены
  types: () => [
    VideoType,
    VideoInProgressProblemType,
    VideoNeedBuyProblemType,
    VideoApproveAgeProblemType,
  ],

  // Ну и самое главное надо объявить функцию определения типа.
  // resolve-функции возвращают JS-объект
  // нам надо исходя из JS-объекта получить GraphQL-тип
  resolveType: value => {
    if (value instanceof Video) {
      return VideoType;
    } else if (value instanceof VideoInProgressProblem) {
      return VideoInProgressProblemType;
    } else if (value instanceof VideoNeedBuyProblem) {
      return VideoNeedBuyProblemType;
    } else if (value instanceof VideoApproveAgeProblem) {
      return VideoApproveAgeProblemType;
    }
    return null;
  },
});

```

<span class="fragment" data-code-focus="5" />
<span class="fragment" data-code-focus="11-16" />
<span class="fragment" data-code-focus="21-32" />

-----

### Ну а дальше возвращать либо  запись, <br/>либо проблему (ошибку)

```js
const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      list: {
        type: new GraphQLList(VideoResultType),
        resolve: () => {
          return [
            new Video({ title: 'DOM2 in the HELL', url: 'https://url' }),
            new VideoApproveAgeProblem({ minAge: 21 }),
            new VideoNeedBuyProblem({ price: 10 }),
            new VideoInProgressProblem({ estimatedTime: 220 }),
          ];
        },
      },
    },
  }),
});

```

<span class="fragment" data-code-focus="6" />
<span class="fragment" data-code-focus="7-14" />

-----

### Тогда фронтендеры смогут писать такие запросы:

```graphql
query {
  list {
    ...on Video {
      title
      url
    }
    ...on VideoInProgressProblem {
      estimatedTime
    }
    ...on VideoNeedBuyProblem {
      price
    }
    ...on VideoApproveAgeProblem {
      minAge
    }
    __typename # магическое поле, которое вернет имя типа для каждой записи
  }
}

```

<span class="fragment" data-code-focus="2" />
<span class="fragment" data-code-focus="3-6" />
<span class="fragment" data-code-focus="7-15" />
<span class="fragment" data-code-focus="16" />

-----

### Ответ от сервера будет таким

```js
{
  data: {
    list: [
      { __typename: 'Video', title: 'DOM2 in the HELL', url: 'https://url' },
      { __typename: 'VideoApproveAgeProblem', minAge: 21 },
      { __typename: 'VideoNeedBuyProblem', price: 10 },
      { __typename: 'VideoInProgressProblem', estimatedTime: 220 },
    ],
  },
}

```

В зависимости от `__typename` можно рендерить <br/>ту или иную компоненту.</span>

-----

### Профит от таких пользовательских ошибок:

- фронтендеры точно знают какие ошибки могут быть <!-- .element: class="fragment" -->
- расписаны поля ошибок (статический анализ) <!-- .element: class="fragment" -->
- получать ошибки сразу на нужном уровне <!-- .element: class="fragment" -->
- легко понять какая именно ошибка вернулась <!-- .element: class="fragment" -->
- в результате чище, проще и безопаснее код <!-- .element: class="fragment" -->

-----

Подробнее про работу с ошибками в GraphQL [читаем тут](https://github.com/nodkz/conf-talks/tree/master/articles/graphql/errors/README.md)
