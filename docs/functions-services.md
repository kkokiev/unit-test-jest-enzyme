# Тестируем функции и сервисы

## Тестирование функций

Самые простые из всех тестов, проверяем что функция возвращает, а не как.
Может обрабатывать несколько кейсов внутри одного теста. Следует тестировать все возможные возвраты, если их больше 1:

```js
import { toLowerCaseOrToString } from '/utils';

describe('toLowerCaseOrToString', () => {
  it('should return empty string', () => {
    expect(toLowerCaseOrToString(null)).toEqual('');
  });

  it('should return lower case string', () => {
      expect(toLowerCaseOrToString('ABC')).toEqual('abc');
  });
});
```

## Тестирование сервисов

Главное внимание, сами тесты не сложные

### Тестируем reducer

```js
describe('appReducer', () => {

  тестируем обработку каждого экшена
  it('handles UPDATE_TARIFF_LIMIT', () => {
    мок для initialState
    const initialState = new Map({
        someVal: true
    });
    
    мокаем action
    const action = updateTariffLimit({
        someVal: false                 
    });
    
    обрабатываем action в reducer 
    const nextState = appReducer(initialState, action);
    
    сверяем результат
    expect(nextState).toEqual(
        fromJS({
          someVal: false
        })
    )
  });
});
```

### Тестируем redux-saga

[Рецепт для саги с api и error от Dan Abramov](https://stackoverflow.com/questions/35654334/how-to-test-api-request-failures-with-redux-saga?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

```js
it('someMiddleware', () => {
    мокаем action
    const action = {
          payload: {
            ...
         }
     }

    создаем генератор запуская action в middleware
    const generator = someMIddleware(action);

    используя .next() сверяем каждое действие внутри саги
    expect(generator.next().value).toEqual(call(api.post, '/someapi/link', data);
    expect(generator.next().value).toEqual(...);
    
    для проверки ошибок
    expect(genaretor.throw({ error: 'error' }).value).toEqual(call(...));
});

```