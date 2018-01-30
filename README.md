# Приложение для создания и редактирования информации о встречах сотрудников (Порядок выполнения работы представлен ниже)

Написано для Node.js 8 и использует библиотеки:
* express
* sequelize
* graphql

## Задание
Код содержит ошибки разной степени критичности. Некоторых из них стилистические, а некоторые даже не позволят вам запустить приложение. Вам необходимо найти и исправить их.

Пункты для самопроверки:
1. Приложение должно успешно запускаться
2. Должно открываться GraphQL IDE - http://localhost:3000/graphql/
3. Все запросы на получение или изменения данных через graphql должны работать корректно. Все возможные запросы можно посмотреть в вкладке Docs в GraphQL IDE или в схеме (typeDefs.js)
4. Не должно быть лишнего кода
5. Все должно быть в едином codestyle

## Запуск
```
npm i
npm run dev
```

Для сброса данных в базе:
```
npm run reset-db
```

---

# Порядок выполнения работы

## Загружаем репозиторий и запускаем

Запустить приложение не удалось
Была найдена ошибка при создании экземпляра Sequelize.
```
models/index.js:7
```
Из документации становиться ясно, что в функцию - конструктор необходимо передать четыре аргумента - название базы данных, имя пользователя, пароль и объект с опциями.
Поскольку мы не работаем удалённо с базой данных, добавляем `null` в качестве недостающего аргумента.

**Ура! Приложение запустилось!**

## GraphQL IDE

По адресу http://localhost:3000/graphql/ должен открываться GraphQL IDE, но к сожалению этого не происходит..
Ошибку удалось легко найти - небольшая опечатка.
```
index.js:14
```

**Приложение и GraphQL IDE работают**

## Тесты

Было решено покрыть приложение тестами, тем самым выполнив две задачи:
* написать тесты
* и найти все ошибки)

### Библиотеки

Для тестирования были установлены библиотеки:
* jest
* graphql-tester

### Запуск

Перед запуском необходимо запустить приложение:
```
npm run dev
```

Для запуска тестов:
```
npm run test
```

### Тесты запросов

Благодаря тестам удалось найти две неисправности:
* установлена опция `offset`, из-за чего в выборке не хватало одного элемента:
```
graphql/resolvers/query.js:8
```

* даты начала и окончания расположены в неправильном порядке:
```
create-mock-data.js:69,68
```

### Тесты мутаций

#### Create user
Произошла ошибка, мы не можем добавить нового пользователя, поскольку передали не корректные данные.
В `UserInput` не указанно обязательное поле `avatarUrl` с типом `String`:
```
graphql/typeDefs.js
```

А так же, возможно поле `homeFloor` должно быть обязательным, так как играет важную роль в работе приложения.

#### Create event
Произошла ошибка, запрос не вернул список пользователей и комнату.
При изучении функции мутации стало ясно, что система не дожидается окончания сохранения комнаты для события:
```
graphql/resolvers/mutation.js@createEvent
```

Вызовы функций `setRoom` и `setUsers` были обёрнуты в `Promise.all`.
К сожалению это не решило проблему :(
Очевидно, что ошибка не в мутациях, скорее всего проблема в связях таблиц.
В объявлении связей ошибок обнаружено не было.
Ошибка оказалась в объявлении функций для полей `Event`, в обеих функциях не хватает оператора `return`:
```
graphql/resolvers/index.js:14,17
```

При поиске ошибки было обнаружено, что возможно аргумент `usersIds` в мутации `createEvent` должен быть обязательным:
```
graphql/typeDefs.js:67
```

#### Remove user from event
В функции мутатора не хватает ожидания выполнения функции `removeUser`:
```
graphql/resolvers/mutation.js@removeUserFromEvent
```

#### Add user to event
Как оказалось функция мутатора не была добавлена, исправим..
В методе присутствует проверка `hasUser`, так как если два раза попытаться добавить одного и того же пользователя выпадет ошибка.

#### Change event room
Устанавливается совсем не та комната, идентификатор которой был передан.
При изучении функции мутатора, была найдена опечатка - вместо `roomId` был указан просто `id`, а так же был пропущен оператор `return`:
```
graphql/resolvers/mutation.js@removeUserFromEvent
```

Ещё в функции не хватает ожидания выполнения функции `setRoom`.

## Доработки

В мутатор `updateEvent`, была добавлена возможность передавать новый список идентификаторов пользователей и идентификатор комнаты.
Это стало необходимым при разработке клиента.

Добавлена библиотека cors для предоставления ресурсов удалённому клиенту.
