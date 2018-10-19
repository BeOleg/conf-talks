# Section #1

# Зачем что-то менять?

## В чем профит от использования GraphQL?

-----

# GraphQL приходит на смену REST API

## Он тупо удобнее 😇 <!-- .element: class="fragment" -->

-----

# У REST API часто адовая документация 👿

-----

#### Самодельная и без интерактива

![gov.stat](./gov-stat-screen-1.png)

-----

#### Страшненькая и неудобная

![gov.stat](./gov-stat-screen-2.png)

-----

## Частенько используют [Swagger](https://swagger.io/)

## для стандартизации, документации и интерактива вашего API

# Но он тоже не айс 🤪 <!-- .element: class="fragment" -->

-----

#### В Swagger есть интерактив, стандарт да и выглядит покрасивше

![swagger-inspector](./swagger-inspector.png)

-----

#### Но от этого не легче 😭

Как вообще этой "фигней" пользоваться?

![swagger-rabota-ua](./swagger-rabota-ua.png)

-----

#### Минусы REST API, даже со Swagger'ом:

- Каждый ендпоинт описывается примитивными типами, нет "сложных" типов <!-- .element: class="fragment" -->
- Нет связей, не известно как оптимальнее всего запросить связанные ресурсы <!-- .element: class="fragment" -->
- Жирные ответы, если не прикрутить фильтр по полям <!-- .element: class="fragment" -->
- Нет возможности построить сложный агрегационный запрос <!-- .element: class="fragment" -->
- Частенько много сетевых запросов <!-- .element: class="fragment" -->
- Боль с вложенными агрументами <!-- .element: class="fragment" -->

-----

#### Самый жирный минус:

## С REST API фронтендеры пишут кучу бойлерплейт кода, для получения связанных данных между ресурсами. <!-- .element: class="fragment" -->

Часто гадают на кофейной гуще 🤔 <!-- .element: class="fragment" -->

Часто с матами 🤬 <!-- .element: class="fragment" -->

Ведь бэкендеры хорошо знают связи между данными, но способ передачи знаний обычно плохо отработан 😶 <!-- .element: class="fragment" -->

-----

# GraphQL чума!

# 💃💃💃

## Когда много связанных данных

-----

# Что такое GraphQL?

![GraphQL](../assets/logo/graphql.png) <!-- .element: style="width: 200px; text-align: center;" class="plain" -->

## Это *удобный* формат для общения между сервером и клиентом

-----

[![GraphQL Query](./graphql-query.png) <!-- .element: class="plain" -->](https://graphql-compose.herokuapp.com/northwind/?query=%7B%0A%20%20viewer%20%7B%0A%20%20%20%20%23%20%D0%93%D0%B8%D0%B1%D0%BA%D0%BE%D1%81%D1%82%D1%8C%20%D0%B0%D0%B3%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%0A%20%20%20%20customer%28filter%3A%20%7BcustomerID%3A%20%22AROUT%22%7D%29%20%7B%0A%20%20%20%20%20%20%23%20%D0%92%D1%8B%D0%B1%D0%BE%D1%80%20%D0%BF%D0%BE%D0%BB%D0%B5%D0%B9%20%D0%B2%20%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B5%0A%20%20%20%20%20%20customerID%0A%20%20%20%20%20%20companyName%0A%20%20%20%20%20%20%23%20%D0%92%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D1%8B%0A%20%20%20%20%20%20%23%20%D0%9E%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5%20%D1%81%D0%B2%D1%8F%D0%B7%D0%B5%D0%B9%20%D0%BC%D0%B5%D0%B6%D0%B4%D1%83%20%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%B0%D0%BC%D0%B8%2F%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8F%D0%BC%D0%B8%0A%20%20%20%20%20%20orderList%28limit%3A%203%2C%20skip%3A%2010%29%20%7B%0A%20%20%20%20%20%20%20%20%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0A%20%20%20%20%20%20%20%20...OrderData%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20%23%20%D0%97%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B8%D1%82%D1%8C%20%D0%BD%D0%B5%D1%81%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE%20%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2%20%D0%B2%20%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B5%0A%20%20%20%20%23%20%D0%93%D0%B8%D0%B1%D0%BA%D0%BE%D1%81%D1%82%D1%8C%20%D0%B0%D0%B3%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%0A%20%20%20%20order1%3A%20order%28filter%3A%20%7BorderID%3A%2011001%7D%29%20%7B%0A%20%20%20%20%20%20%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0A%20%20%20%20%20%20...OrderData%0A%20%20%20%20%7D%0A%20%20%20%20%23%20%D0%97%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B8%D1%82%D1%8C%20%D0%BD%D0%B5%D1%81%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE%20%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2%20%D0%B2%20%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B5%0A%20%20%20%20%23%20%D0%93%D0%B8%D0%B1%D0%BA%D0%BE%D1%81%D1%82%D1%8C%20%D0%B0%D0%B3%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%0A%20%20%20%20order2%3A%20order%28filter%3A%20%7BorderID%3A%2011002%7D%29%20%7B%0A%20%20%20%20%20%20%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0A%20%20%20%20%20%20...OrderData%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A%0A%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0Afragment%20OrderData%20on%20Order%20%7B%0A%20%20%23%20%D0%92%D1%8B%D0%B1%D0%BE%D1%80%20%D0%BF%D0%BE%D0%BB%D0%B5%D0%B9%20%D0%B2%20%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B5%0A%20%20orderID%0A%20%20orderDate%0A%20%20freight%0A%7D)

-----

<iframe src="https://graphql-compose.herokuapp.com/northwind/?query=%7B%0A%20%20viewer%20%7B%0A%20%20%20%20%23%20%D0%93%D0%B8%D0%B1%D0%BA%D0%BE%D1%81%D1%82%D1%8C%20%D0%B0%D0%B3%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%0A%20%20%20%20customer%28filter%3A%20%7BcustomerID%3A%20%22AROUT%22%7D%29%20%7B%0A%20%20%20%20%20%20%23%20%D0%92%D1%8B%D0%B1%D0%BE%D1%80%20%D0%BF%D0%BE%D0%BB%D0%B5%D0%B9%20%D0%B2%20%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B5%0A%20%20%20%20%20%20customerID%0A%20%20%20%20%20%20companyName%0A%20%20%20%20%20%20%23%20%D0%92%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D1%8B%0A%20%20%20%20%20%20%23%20%D0%9E%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5%20%D1%81%D0%B2%D1%8F%D0%B7%D0%B5%D0%B9%20%D0%BC%D0%B5%D0%B6%D0%B4%D1%83%20%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%B0%D0%BC%D0%B8%2F%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8F%D0%BC%D0%B8%0A%20%20%20%20%20%20orderList%28limit%3A%203%2C%20skip%3A%2010%29%20%7B%0A%20%20%20%20%20%20%20%20%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0A%20%20%20%20%20%20%20%20...OrderData%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20%23%20%D0%97%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B8%D1%82%D1%8C%20%D0%BD%D0%B5%D1%81%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE%20%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2%20%D0%B2%20%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B5%0A%20%20%20%20%23%20%D0%93%D0%B8%D0%B1%D0%BA%D0%BE%D1%81%D1%82%D1%8C%20%D0%B0%D0%B3%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%0A%20%20%20%20order1%3A%20order%28filter%3A%20%7BorderID%3A%2011001%7D%29%20%7B%0A%20%20%20%20%20%20%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0A%20%20%20%20%20%20...OrderData%0A%20%20%20%20%7D%0A%20%20%20%20%23%20%D0%97%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B8%D1%82%D1%8C%20%D0%BD%D0%B5%D1%81%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE%20%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BE%D0%B2%20%D0%B2%20%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B5%0A%20%20%20%20%23%20%D0%93%D0%B8%D0%B1%D0%BA%D0%BE%D1%81%D1%82%D1%8C%20%D0%B0%D0%B3%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%0A%20%20%20%20order2%3A%20order%28filter%3A%20%7BorderID%3A%2011002%7D%29%20%7B%0A%20%20%20%20%20%20%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0A%20%20%20%20%20%20...OrderData%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A%0A%23%20%D0%A4%D1%80%D0%B0%D0%B3%D0%BC%D0%B5%D0%BD%D1%82%D1%8B%20%28%D0%B4%D0%BB%D1%8F%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B0%29%0Afragment%20OrderData%20on%20Order%20%7B%0A%20%20%23%20%D0%92%D1%8B%D0%B1%D0%BE%D1%80%20%D0%BF%D0%BE%D0%BB%D0%B5%D0%B9%20%D0%B2%20%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B5%0A%20%20orderID%0A%20%20orderDate%0A%20%20freight%0A%7D" width="100%" height="720px" />

-----

#### Фишки GraphQL:

- Описание бэкендерами связей между ресурсами c фильтрацией <!-- .element: class="fragment" -->
- Выбор полей фронетнедерами в ответе из коробки <!-- .element: class="fragment" -->
- Вложенные запросы <!-- .element: class="fragment" -->
- Гибкость агрументов на любом уровне вложенности <!-- .element: class="fragment" -->
- Получение нескольких ресурсов в одном запросе <!-- .element: class="fragment" -->
- Фрагменты (для компонентного подхода) <!-- .element: class="fragment" -->
- Поддерживает сложные типы (статическая типизация 🌶) <!-- .element: class="fragment" -->
- Экзотика полиморфизма (Interfaces, Union types) <!-- .element: class="fragment" -->

-----

**На сервере объявляете о своих _возможностях_**

в предоставлении данных (бэкендеры создают GraphQL схему).

![GraphQL](../assets/logo/graphql.png) <!-- .element: style="width: 200px; text-align: center;" class="plain" -->

**На клиенте заявляете о своих _потребностях_**

в получении данных (фронтендеры пишут GraphQL запросы).

-----

### GraphQL — это не база данных!

GraphQL помогает бэкендерам задать конву - описание структуры, типов и связей в ваших данных.

-----

### Работает с любыми базами данных, на любом языке программирования!

Грубо говоря, GraphQL требует от бекендеров описать набор функций для получения данных 👌 <!-- .element: class="fragment" -->

-----

## Концептуальная разница GraphQL и REST API в том

## что логику получения связанных ресурсов перенесли с клиента на сервер. <!-- .element: class="fragment" -->

Сняли жуткий головняк с фронтендеров 👍 <!-- .element: class="fragment" -->

-----

#### Более подробнее расписано тут

- [Что такое GraphQL?](https://github.com/nodkz/conf-talks/blob/master/particles/graphql/README.md)
- [Swagger vs GraphQL](https://github.com/nodkz/conf-talks/blob/master/particles/swagger/README.md)
