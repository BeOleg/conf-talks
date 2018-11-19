# DataLoader

## (avoiding N+1 problem)

-----

## [DataLoader](https://github.com/facebook/dataloader) — это утилита сокращающая кол-во обращений в базу через batching запросов.

-----

### Пример: N+1 problem

#### запрашиваем список статей с именем автора

```graphql
{
  articles {
    title
    author {
      name
    }
  }
}

```

-----

#### Стандартная ситуация в мире GraphQL

#### для списка из 15 статей сделать 16 запросов в базу 😬

```text
Run Article query: findMany()
Run Author query: findById(1)
Run Author query: findById(7)
Run Author query: findById(6)
Run Author query: findById(3)
Run Author query: findById(4)
Run Author query: findById(5)
Run Author query: findById(6)
Run Author query: findById(7)
Run Author query: findById(3)
Run Author query: findById(2)
Run Author query: findById(5)
Run Author query: findById(4)
Run Author query: findById(2)
Run Author query: findById(1)
Run Author query: findById(1)

```

<span class="fragment" data-code-focus="1" />
<span class="fragment" data-code-focus="2-16" />

-----

### Кто-то скажет:

## — Фууу, какая отвратительная производительность!

## — GraphQL лажа полная!

-----

### Но я скажу:

## — На самом деле, <br />в GraphQL скормили <br/>лажовый код <br/>в resolve-методах

-----

### Что происходит под капотом:

```graphql
{
  articles {
    title
    author {
      name
    }
  }
}

```

<span class="fragment" data-code-focus="2" >`Query.articles.resolve()` — 1 запрос для получения массива из 15 статей</span>

<span class="fragment" data-code-focus="4" >`Article.author.resolve()` — 15 запросов, чтоб для каждой статьи получить автора по Id.</span>

-----

### Не нужно сразу тянуть автора по id!

### Сперва собираем все айдишники авторов <!-- .element: class="fragment" -->

### Потом одним запросом тянем из БД <!-- .element: class="fragment" -->

-----

### Вот так должно работать у хороших ребят:<br/><br/>

```text
Run Article query: findMany()
Run Author query: findByIds(1, 7, 6, 3, 4, 5, 2)

```

### <br/>2 запроса вместо 16-ти

-----

## Для этой задачи нам и нужен `DataLoader`

DataLoader — это batcher и cacher в одном флаконе.

-----

### Механика получения данных через DataLoader 

#### 1. Инициализируем DataLoader

```js
const authorDataLoader = new DataLoader(async batchLoad(ids) => { ... });

```

#### 2. Запрашиваем Авторов по id где нужно

```js
const authorPromise1 = authorDataLoader.load(1);
const authorPromise2 = authorDataLoader.load(2);
const authorPromise1_copy = authorDataLoader.load(1); // <--- CACHE detected 👍
// authorPromise1 === authorPromise1_copy
const authorPromise4 = authorDataLoader.load(4);

```

#### 3. Автоматом на `nextTick()` выполнится batch-запрос, после чего все промисы из п.2 разрезолвятся.

-----

### Нюанс объявления `batchLoad` в DataLoader

```js
// Создаем объект DataLoader и сразу в конструктор передаем
// функцию batch-загрузки по ids
const authorDataLoader = new DataLoader(
  async batchLoad(ids) => {
    // получили массив айдишников, дергаем записи одним запросом из базы
    const rows = await authorModel.findByIds(ids);
    // ВАЖНО: полученные записи мы ДОЛЖНЫ вернуть в том порядке
    //        в котором получили ids на входе
    // Eсли запись по id не будет найдена, то вернется undefined
    const sortedInIdsOrder = ids.map(id => rows.find(x => x.id === id));
    return sortedInIdsOrder;
  }
);

```

<span class="fragment" data-code-focus="4" />
<span class="fragment" data-code-focus="5-6" />
<span class="fragment" data-code-focus="7-8" />
<span class="fragment" data-code-focus="10" />

Иначе получим бардак в данных.

-----

### Вроде все просто,

### пока не начинают женить DataLoader с GraphQL

## 🤔

-----

### Я думал, что всё просто c

### GraphQL + DataLoader:

- пока в [чате по GraphQL](https://t.me/graphql_ru) не прочитал кучу негатива
- пока заново не перечитал документашку DataLoader'а

-----

### Это фиаско, котаны!

- нет толкового примера
- изобилие текста и методов уводит народ непонятно куда
  - воспринимается как замена для Redis'а
- сложно уловить что он должен быть в `context`е GraphQL
- **начинает криво использоваться**

-----

### Правило 1: DataLoader должен использоваться в рамках одного запроса.

- 👌 При получение запроса создайте DataLoader в `context`
  - Выполните запрос
  - Затем смело удаляйте контекст с даталоадером
- 👌 Для новых запросов вы создаете новые DataLoader'ы!

-----

### Правило 2: Для каждого резолвера свой DataLoader.

- 🤮 общий (глобальный) дата-лоадер ведет
  - к грязному коду, т.к. вы на уровне сервера объявляете batch-функцию
  - должен получить из БД полную запись, т.к. разные резолверы могут запросить разные поля
- 👌️ DataLoader необходимо объявлять внутри resolve-метода
  - тут вы знаете какие поля нужны юзеру, их и тянем из БД

-----

## Ну что, пришло время примера

-----

### На уровне сервера в контексте создаем `WeakMap` для будущих дата-лоадеров

```js
const server = new ApolloServer({
  schema,
  context: ({ req }) => ({
    req,
    dataloaders: new WeakMap(),
  }),
});

```

<span class="fragment" data-code-focus="5" />

<span class="fragment">Никакого убогого кода с моделями на уровне сервера ☝️</span>

<span class="fragment">Это позволит находить существующие дата-лоадары, либо ложить туда новый инстанс дата-лоадера для повторного использования. На уровне резолвера! ☝️</span>

-----

### На уровне resolve-метода

```js
const ArticleType = new GraphQLObjectType({
  name: 'Article',
  fields: () => ({
    title: { type: GraphQLString },
    authorId: { type: GraphQLString },
    author: {
      type: AuthorType,
      resolve: (source, args, context, info) => {
        // context.dataloaders был создан на уровне сервера
        const { dataloaders } = context;

        // единожы инициализируем DataLoader для получения авторов по ids
        let dl = dataloaders.get(info.fieldNodes);
        if (!dl) {
          dl = new DataLoader(async (ids: any) => {
            // обращаемся в базу чтоб получить авторов по ids
            const rows = await authorModel.findByIds(ids);
            // сортируем данные из базы в том порядке, как нам передали ids
            const sortedInIdsOrder = ids.map(id => rows.find(x => x.id === id));
            return sortedInIdsOrder;
          });
          // ложим инстанс дата-лоадера в WeakMap для повторного использования
          dataloaders.set(info.fieldNodes, dl);
        }

        // юзаем метод `load` из нашего дата-лоадера
        return dl.load(source.authorId);
      },
    },
  }),
});

```

<span class="fragment" data-code-focus="2,6" />
<span class="fragment" data-code-focus="9-10" />
<span class="fragment" data-code-focus="12-13" />
<span class="fragment" data-code-focus="14-24" />
<span class="fragment" data-code-focus="26-27" />

-----

### Почему используется WeakMap и info.fieldNodes?

- `info.fieldNodes` — это объект, и он один и тот же для одинаковых путей запроса, eg. Query.articles.author.resolve()
- `WeakMap` — чтоб можно было использовать fieldNodes в качестве ключа для поиска уже созданных дата-лоадеров

-----

### Еще одна магия info.fieldNodes

```graphql
{
  article(id: 5) {
    author { nickname } # DataLoader1
  }
  articles {
    title
    author { name email } # DataLoader2
  }
  lastComments {
    article {
      author  { name } # DataLoader3
    }
  }
}

```

По разным путям создаются разные дата-лоадеры.

-----

#### Это позволяет создавать в одном и том же резолвере разные DataLoader в зависимости от GraphQL-запроса.

```graphql
{
  article(id: 5) {
    author { nickname } # DataLoader1
  }
  articles {
    title
    author { name email } # DataLoader2
  }
  lastComments {
    article {
      author  { name } # DataLoader3
    }
  }
}

```

#### И каждый DataLoader может тянуть из базы, только те поля которые запросил пользователь.

-----

## В сухом остатке

### DataLoader клевая утилита, если ее использовать разумно.

### Если к ней относиться как к простому группировщику запросов.

-----

## В сухом остатке

- Не пытайтесь из DataLoader'а смастерить кэш
- Новые DataLoader'ы для каждого GraphQL-запроса
- Создаайте DataLoader в resolve-методе
- Ничего страшного если для одного резолвера у вас создасться несколько DataLoader'ов

-----

## Чистого кода и меньше бесполезных запросов в ваш ~~дом~~ бэкенд!

-----

### Подробнее про DataLoader с примерами можно [почитать тут](https://github.com/nodkz/conf-talks/tree/master/articles/graphql/dataloader)
