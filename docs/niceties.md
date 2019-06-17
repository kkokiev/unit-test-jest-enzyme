# Тонкости и хитрости
Или как писат ьмного тестов и не сойти с ума

## Явный setup() вместо beforeEach()
Выгода в том что в любом тесте понятно как компонент был инициализирован. Объект setup также хорошее место для вспомогательных функций.

```js
const setup = propOverrides => {
  const props = Object.assign({
    completeCount: 0,
    activeCount: 0,
    onClearCompleted: jest.fn() 
  }, propOverrides)

  const wrapper = shallow(<Footer {...props} />)

  return {
    props,
    wrapper,
    clear: wrapper.find('.clear-completed'),
    count: wrapper.find('.todo-count')
  }
}

describe('count', () => {
  test('when active count 0', () => {
    const { count } = setup({ activeCount: 0 })
    expect(count.text().toEqual('No items left')
  })

  test('when active count above 0', () => {
    const { count } = setup({ activeCount: 1 })
    expect(count.text().toEqual('1 item left')) 
  })
})
```

## Использовать helper functions
Иногда нужно написать много похожих тестов с одной переменной.
Можно использовать функцию генерящую тесты:

```js
describe('todo list', () => {
  
  const testFilteredTodos = (filter, todos) => {
    test('render ${filter} items', () => {
      const { wrapper, footer } = setup()
      footer.props().onShow(filter)
      expect(wrapper.find(TodoItem).map(node => node.props().todo)).toEqual(todos)
    }) 
  }

  testFilteredTodos(SHOW_ALL, todos)
  testFilteredTodos(SHOW_ACTIVE, [todos[0]])
  testFilteredTodos(SHOW_COMPLETED, [todos[1]])
})
```