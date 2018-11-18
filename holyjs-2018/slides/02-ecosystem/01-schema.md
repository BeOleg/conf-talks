# GraphQL Schema

![Диаграмма экосистемы](./diagram-ecosystem-schema.svg) <!-- .element: style="width: 90vw;" class="plain"  -->

-----

# GraphQL Schema — это

### описание ваших типов данных на сервере, <!-- .element: class="fragment" -->

### связей между ними <!-- .element: class="fragment" -->

### и логики получения этих самых данных. <!-- .element: class="fragment" -->

-----

## GraphQL-схема это точка входа, это корень всего вашего API.

-----

### И у этого корня три "головы" 🐲🐲🐲

- `query` — для операций получения данных
- `mutation` — для операций изменения данных
- `subscription` — для подписки на события

-----

##### Hello world schema (build phase)

<pre><code data-trim>
import { GraphQLSchema, GraphQLObjectType, GraphQLString } from 'graphql';

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
      hello: {
        type: GraphQLString,
        resolve: () => 'world',
      }
    }
  }),
  // mutation: { ... },
  // subscription: { ... },
});
</code></pre>

Когда вы описываете структуру своих данных и методы получения.

-----

##### Hello world schema (runtime phase)

<pre><code data-trim>
import { graphql } from 'graphql';
import { schema } from './your-schema';

const query = '{ hello }';
const result = await graphql(schema, query);

// returns: { data: { hello: "world" } }
</code></pre>

##### Когда вы выполняете клиентский запрос на вашей схеме методом graphql() который: <!-- .element: class="fragment" -->

- производит парсинг GraphQL-запроса <!-- .element: class="fragment" -->
- производит валидацию запроса на соответствие GraphQL-схемы <!-- .element: class="fragment" -->
- выполняет запрос, пробегаясь по дереву схемы <!-- .element: class="fragment" -->
- валидирует возвращаемый ответ <!-- .element: class="fragment" -->

-----

## GraphQL по натуре строго типизированный

## Шаг влево, шаг вправо от схемы — <!-- .element: class="fragment" -->

# ☠️ РАССТРЕЛ ☠️ <!-- .element: class="fragment" -->

-----

## ☠️ РАССТРЕЛ ☠️

- фронтендера если криво запросил данные <!-- .element: class="fragment" -->
- бекендера если криво вернул данные <!-- .element: class="fragment" -->

-----

### GraphQL-схема своей строгостью заставляет

### Бэкендеров и Фронтендеров

### 👩‍❤️‍💋‍👩 жить дружно 👨‍❤️‍💋‍👨

### 💋❤️🍻

#### И винить во всех грехах GraphQL, т.к. теперь он крайний! <!-- .element: class="fragment" -->