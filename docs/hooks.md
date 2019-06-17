# React hooks

Неплохая статья на тему что вообще такое react-hooks и как их тестировать [A quick guide to testing React Hooks](https://blog.logrocket.com/a-quick-guide-to-testing-react-hooks-fa584c415407/) На момент написания статьи Enzyme еще не поддерживал react-hooks, и автор предлагает использовать дополнительную библиотеку [react-testing-library](https://github.com/testing-library/react-testing-library)


Корки из [React Hooks: Test custom hooks with Enzyme](https://dev.to/flexdinesh/react-hooks-test-custom-hooks-with-enzyme-40ib)
Напишем кастомный hook для апдейта и валидации поля формы

`useFormField.js`
```js
import React, { useState } from 'react';

const useFormField = (initialVal = '') => {
  const [val, setVal] = useState(initialVal);
  const [isValid, setValid] = useState(true);

  const onChange = (e) => {
    setVal(e.target.value);

    if (!e.target.value) {
      setValid(false);
    } else if (!isValid) setValid(true);
  }

  return [val, onChange, isValid];
}

export default useFormField;
```

У хуков есть одно ограничение. Хоть это всего лишь javascript функции, они будут работать только внутри компонетов реакта.
Нельзя просто вызвать эту функцию и проверить вернувшийся результат.
Можно передать хук как prop в компонент, и написать wrapper component для рендера и тестирования хука.
Теперь к хуку есть доступ и можно тестировать его поведение.

`useFormField.test.js`
```js
function HookWrapper(props) {
  const hook = props.hook ? props.hook() : undefined;
  return <div hook={hook} />;
}

describe('useFormField', () => {
  it('should render', () => {
    let wrapper = shallow(<HookWrapper />);

    expect(wrapper.exists()).toBeTruthy();
  });

  it('should set init value', () => {
    let wrapper = shallow(<HookWrapper hook={() => useFormField('')} />);

    let { hook } = wrapper.find('div').props();
    let [val, onChange, isValid] = hook;
    expect(val).toEqual('');

    wrapper = shallow(<HookWrapper hook={() => useFormField('marco')} />);

    // destructuring objects - {} should be inside brackets - () to avoid syntax error
    ({ hook } = wrapper.find('div').props());
    [val, onChange, isValid] = hook;
    expect(val).toEqual('marco');
  });

  it('should set the right val value', () => {
    let wrapper = shallow(<HookWrapper hook={() => useFormField('marco')} />);

    let { hook } = wrapper.find('div').props();
    let [val, onChange, isValid] = hook;
    expect(val).toEqual('marco');

    onChange({ target: { value: 'polo' } });

    ({ hook } = wrapper.find('div').props());
    [val, onChange, isValid] = hook;
    expect(val).toEqual('polo');
  });

  it('should set the right isValid value', () => {
    let wrapper = shallow(<HookWrapper hook={() => useFormField('marco')} />);

    let { hook } = wrapper.find('div').props();
    let [val, onChange, isValid] = hook;
    expect(val).toEqual('marco');
    expect(isValid).toEqual(true);

    onChange({ target: { value: 'polo' } });

    ({ hook } = wrapper.find('div').props());
    [val, onChange, isValid] = hook;
    expect(val).toEqual('polo');
    expect(isValid).toEqual(true);

    onChange({ target: { value: '' } });

    ({ hook } = wrapper.find('div').props());
    [val, onChange, isValid] = hook;
    expect(val).toEqual('');
    expect(isValid).toEqual(false);
  });
});
```
