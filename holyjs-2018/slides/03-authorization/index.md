# АВТОРИЗАЦИЯ

-----

### Что такое...

|                    | <!-- .element: class="fragment" data-fragment-index="1" -->|
|--------------------|----------------------|
| **Аутентификация** | процедура проверки подлинности пользователя путём сравнения введённого им логина и пароля. <!-- .element: class="fragment" data-fragment-index="1" --> |
| **Идентификация**  | процедура распознания пользователя по токену, сессии или кукам. <!-- .element: class="fragment" data-fragment-index="1" --> |
| **Авторизация**    | процедура проверки прав доступа к ресурсам на выполнение определённых действий. <!-- .element: class="fragment" data-fragment-index="1" --> |

-----

## 1. Аутентификация — Sign In

- **Без GraphQL**
  - отправляем логин/пароль через старый добрый POST, получаем токен
  - используем OAuth, получаем токен

- **Через GraphQL**
  - крутим query или mutation, через аргументы передаем логин/пароль в респонсе получаем токен; плюс сразу можем получить вагон данных

-----

## В результате Аутентификации должен быть ТОКЕН

-----

## 2. Идентификация — JWT, cookie

<br/>

### В мире GraphQL для генерации токенов сильно прижился [JWT](https://jwt.io/)

<br/>

Читать про JSON Web Token в [википедии](https://ru.wikipedia.org/wiki/JSON_Web_Token).

-----

#### JWT-токен состоит из:

<p style="font-size: 1.2em; font-weight: bold;">
<span style="color: #00b9f1" class="fragment" data-fragment-index="1">header</span>.<span style="color: #d63aff" class="fragment" data-fragment-index="2">payload</span>.<span style="color: #fb015b" class="fragment" data-fragment-index="3">sign</span>
</p>

**Наример:**

<span style="color: #00b9f1" class="fragment" data-fragment-index="1">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.</span>
<br/><span style="color: #d63aff" class="fragment" data-fragment-index="2">eyJzdWIiOjEsImlhdCI6MTU0MTI1MDE2M30.</span>
<br/><span style="color: #fb015b" class="fragment" data-fragment-index="3">M7xYg8GuEgwbqTrta0xnN7WmNEXOCKiQGDdogt_Kduk</span>

**Внутрянка:**

<span style="color: #00b9f1" class="fragment" data-fragment-index="1">base64({ "alg": "HS256", "typ": "JWT" }).</span>
<br/><span style="color: #d63aff" class="fragment" data-fragment-index="2">base64({ "sub": 1, "iat": 1541250163 }).</span>
<br/><span style="color: #fb015b" class="fragment" data-fragment-index="3">HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), JWT_SECRET_KEY)</span>

-----

## C JWT легко работать на сервере

```js
import jwt from 'jsonwebtoken';

const JWT_SECRET_KEY = 'qwerty ;)';

// Генерация токена
const token = jwt.sign({ sub: 2 }, JWT_SECRET_KEY);

// Проверка токена
const payload = jwt.verify(token, JWT_SECRET_KEY);
// { "sub": 1, "iat": 1541250163 }
// sub - это subject, например id пользователя
// iat - это время генерации токена
// а еще есть `iss`, `aud`, `exp`, `nbf`, `jti` - смотри спеку

```

-----

## Где хранить ТОКЕН на клиенте?

<br />

- `Если браузер` — то в куках с флагом httpOnly
  - делается только бэкендером, фронтендеру ничего делать не нужно <!-- .element: class="fragment" -->
  - если фронтендер в браузере хоть как-то сохранит токен, то привет XSS ☠️ <!-- .element: class="fragment" -->
- `Если мобильное приложение` — то в "AsyncStorage"
  - передаем токен через HTTP-заголовки с каждым запросом <!-- .element: class="fragment" -->

-----

## ☝️ Пссс, Бэкендер! ☝️

### Ты должен:

- уметь принимать и ставить токен через httpOnly куки
- а если там пусто, то посмотреть в HTTP-заголовках
- иначе Индетификация не прошла и перед тобой аноним 👻

-----

## 3. Авторизация — Прикручиваем ACL

Ее можно и нужно настраивать на следующих уровнях:

- на уровне сервера (apollo, express, koa и пр.)
- на уровне GraphQL-схемы (глобально на первых полях схемы)
- на уровне полей (в resolve методах)
- на уровне связей между типами (в resolve методах)

-----

## 3.1. Авторизация на уровне сервера (apollo, express, koa и пр.)

- считать токен из кук, либо заголовков
- провалидировать токен и произвести индетификацию пользователя
- передать пользователя в `context` graphql
- либо завернуть запрос, если токен невалиден или пользователь забанен

-----

### Пишем функцию помогайку получения пользователя из реквеста:

```js
async function getUserFromReq(req: $Request) {
  const token = req?.cookies?.token || req?.headers?.authorization;
  if (token) {
    const payload = jwt.verify(token, JWT_SECRET_KEY);
    const userId = payload?.sub;
    if (userId) {
      const user = await users.find(userId);
      if (user) {
        if (user.isBanned) throw new Error('Looser!');
        return user;
      }
    }
  }
  return null;
}

```

<span class="fragment" data-code-focus="2" />
<span class="fragment" data-code-focus="4" />
<span class="fragment" data-code-focus="7" />
<span class="fragment" data-code-focus="9,10,14" />

-----

### Ну и на сервере формируем GraphQL-контекст

```js
const server = new ApolloServer({
  schema,
  context: async ({ req }) => {
    let user;
    try {
      user = await getUserFromReq(req);
    } catch (e) {
      throw new AuthenticationError('You provide incorrect token!');
    }
    const role = user?.role || 'GUEST';
    return { req, user, role };
  },
});

```

<span class="fragment" data-code-focus="3" />
<span class="fragment" data-code-focus="6" />
<span class="fragment" data-code-focus="8" />
<span class="fragment" data-code-focus="10" />
<span class="fragment" data-code-focus="11" />

- Контекст формируется для каждого http-запроса
- Переменные req, user и role будут доступны в агрументе context во всех методах resolve(source, args, `context`)

-----

## 3.2. Авторизация на уровне GraphQL-схемы

## (глобально на верхних полях схемы)

-----

## Это когда на первом уровне в `Query` размещаются  так называемые namespace-типы.

## Еще их можно назвать поля-роли `viewer`, `me`, `admin` и пр.

-----

```graphql
query {
  viewer { # любые пользователи имееют доступ к получению данных
    getNews
    getAds
  }
  me { # здесь отображаются данные только для текущего пользователя
    nickname
    photo
  }
  admin { # а здесь методы, которые доступны только админам
    shutdown
    exposePersonalData
  }
}

```

Тоже самое можно применить к мутациям.

-----

## Как это работает?

### Если resolve-метод для поля вернул null, undefined или выбросил ошибку, то обработка вложенных полей не происходит.

-----

```js
const AdminNamespace = new GraphQLObjectType({
  name: 'AdminNamespace',
  fields: () => ({
    shutdown: { ... },
    exposePersonalData: { ... },
  }),
});

const Query = new GraphQLObjectType({
  name: 'Query',
  fields: () => ({
    viewer: { ... },
    me: { ... },
    admin: {
      type: AdminNamespace,
      resolve: (_, __, context) => {
        if (context.role === 'ADMIN') {
          // LIFEHACK: возвращаем пустой объект
          // чтоб GraphQL вызвал resolve-методы у вложенных полей
          return {};
        }

        // А теперь у нас два варианта. Либо выбросить ошибку:
        throw new Error('Hey, пошел прочь от советской власти!');

        // Либо тихо вернуть пустышку в ответе для поля `admin`
        // и не выполнять никакие вложенные резолверы
        return null;
      },
    },
  }),
});

```

<span class="fragment" data-code-focus="1-7" />
<span class="fragment" data-code-focus="12-14" />
<span class="fragment" data-code-focus="15-16" />
<span class="fragment" data-code-focus="17-21" />
<span class="fragment" data-code-focus="23-24" />
<span class="fragment" data-code-focus="26-28" />

-----

Неймспейсы еще хороши тем, что позволяют красиво нарезать ваше API и не делать из него помойку (как например это делает Prisma).

-----

![Photo](./namespaces.png) <!-- .element:  style="max-height: 80vh;" class="plain"  -->

-----

## 3.3. Авторизация на уровне полей

## (в resolve-методах)

-----

### Когда у вас в `context` есть информация о текущем пользователе и его роли, то можно настроить авторизацию для конкретных полей.

-----

### Дано:

У нас есть Пользователь и мы храним <br/> его последний IP-адрес в поле `lastIp`.

<br/>

### Задача:

Разрешить отображение ip-адреса <br/> только админу и самому пользователю.

-----

```js
const UserType = new GraphQLObjectType({
  name: 'User',
  fields: () => ({
    name: {
      type: new GraphQLNonNull(GraphQLString),
    },
    lastIp: {
      type: GraphQLString,
      resolve: (source, _, context) => {
        const { id, lastIp } = source;

        // return IP for ADMIN
        if (context.role === 'ADMIN') return lastIp;

        // return IP for current user
        if (id === context.user.id) return lastIp;

        // для всех остальных айпишник не светим
        return null;

        // либо можно выбросить ошибку
        // throw new Error('Hidden due private policy');
      },
    },
  }),
});

```

<span class="fragment" data-code-focus="2,4,7" />
<span class="fragment" data-code-focus="9" />
<span class="fragment" data-code-focus="10" />
<span class="fragment" data-code-focus="9,13" />
<span class="fragment" data-code-focus="9,10,16" />
<span class="fragment" data-code-focus="18-19,21-22" />

-----

## 3.4. Авторизация на уровне связей между типами

## (в resolve-методах)

-----

## Это практически тоже самое что и авторизация на уровне полей.

<br/>

### Но есть ньюанс ☝️

-----

## Вы должны проверить не просто возможность получения связанных объектов,

## но и сами полученные объекты, на право отображания.

-----

```js
const UserType = new GraphQLObjectType({
  name: 'User',
  fields: () => ({
    name: {
      type: new GraphQLNonNull(GraphQLString),
    },
    metaList: {
      type: new GraphQLList(MetaDataType),
      resolve: async (source, _, context) => {
        const { id } = source;

        // тут если надо проверяем есть ли доступ (как в пункте 3)

        // если доступ есть, то получаем данные
        let metaList = await Meta.find(o => o.userId === id);

        // проверяем доступ на отображение полученных данных
        // (это отличие от пункта 3)
        metaList = metaList.filter(m => m.forRole === context.role);

        return metaList;
      },
    },
  }),
});

```

<span class="fragment" data-code-focus="2,4,7" />
<span class="fragment" data-code-focus="8" />
<span class="fragment" data-code-focus="12" />
<span class="fragment" data-code-focus="14-15" />
<span class="fragment" data-code-focus="17-19" />

-----

## Авторизация на уровне связей между типами делает проверку

- до получения данных
- и после получения данных

#### <br/>Иначе можно отдать данные, <br/>которые отдавать не стоит 😉

-----

## Appendix:

### Почему я использую три токена

### `user`, `account`, `admin`

-----

- `user` — чтобы индетифицировать текущего пользователя

- `account` — чтобы индетифицировать доступ к данным

- `admin` — чтобы индетифицировать админа

-----

## `user`

### Никакие данные я обычно с id-шником пользователя не храню.

### Все данные я вяжу к `account`

-----

## Юзеры это ненадежный материал

Например: какой-то менеджер регистрирует личный кабинет, через полгода увольняется; на его место приходит новый менеджер, и теперь везде надо перебивать старое мыло пользователя на новое.

-----

## `account`

### Всё что создается —<br/>статья, документ, заказ, заявка и пр. <br/><br/>Всё привязывается к аккаунту.

-----

## Аккаунт хранит в себе id пользователей.

-----

## Самый кайф в том, что:

- к одному аккаунту могут иметь доступ несколько пользователей
- а один пользователь может иметь доступ к нескольким аккаунтам (только дайте ему выпадайку, чтоб удобно было переключать аккаунты).

-----

## `admin`

### Чтобы индетифицировать админа системы

### И давать ему доступ в закрытую админскую часть

-----

## А еще фишка в том, что админ может получить токен `user` и `account` и ...

## используя стандартные интерфейсы зайти в любой  личкаб и посмотреть глазами пользователя, что у него происходит<!-- .element: class="fragment" -->

-----

## Три токена

## `user` `account` `admin`

### Берите и пользуйтесь на здоровье 😉