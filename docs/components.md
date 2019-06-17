# Компоненты

В целом рекомендуется использовать в первую очередь shallow и only making assertions on which components your thing renders.
Другими словами, если компонент А рендерит компонент Б который рендерит С, то тесты А ничего не должны знать про С и должны проверять что А рендерит Б с правильными пропсами. Тесты для компонента Б будут проверять С.

`Shallow` и `Mount` отличаются не только способом рендера но и доступностью lifecycle методов.
В связи с сильными изменениями lifecycle методов в 16.х версии реакта, рекомендуется дополнительно покурить доки перед использованием.

При тестировании компонента может возникнуть огромное количество частных и специфических моментов.
Попробую рассмотреть наиболее частые из них.

### Достаём компонент обернутый в connect:

```js
const wrapper = shallow(<ConnectedComponent />).dive();
```

### Снимки ака snapshots

В первый раз, когда запускается тест, создаётся файл снимка. После этого можно посмотреть созданный файл, чтобы проверить, соответствует ли отрендеренный компонент ожидаемому результату. Snapshot, при необходимости можно перезаписать.

`link.js`

```jsx
import React, { Component } from 'react';
import { string } from 'prop-types';

class Link extends Component {
  state = { clicked: false };

  handleClick = () => this.setState({ clicked: true });
  
  render() {
    const { title, url } = this.props;
    
    return <a href={url} onClick={this.handleClick}>{title}</a>;
  }
}

Link.propTypes = {
  title: string.isRequired,
  url: string.isRequired
};
export default Link;
```

`link.test.js`
```js
import React from 'react';
import { shallow } from 'enzyme';
import { shallowToJson } from 'enzyme-to-json';

import Link from './link';

describe('Link', () => {
  it('should render correctly', () => {
    const output = shallow(
      <Link title="mockTitle" url="mockUrl" />
    );
    expect(shallowToJson(output)).toMatchSnapshot();
  });
});
```

### Тестирование состояния

```js
it('should handle state changes', () => {
  const output = shallow(
    <Link title="mockTitle" url="mockUrl" />
  );
  
  expect(output.state().clicked).toEqual(false);
  output.simulate('click');
  expect(output.state().clicked).toEqual(true);
});
```

### Достаём методы компонента
Используем `.instance()` из Enzyme для доступа к методам компонента

`MyComponnet.js`
```jsx
import React, { Component } from 'react';

class MyComponent2 extends Component {
  _method = () => {
    return true;
  };
  render() {
    return null;
  }
}

export default MyComponent2;
```

`MyComponent.test.js`
```js
import React from 'react';
import { shallow } from 'enzyme';
import MyComponent2 from '../MyComponent2';

describe('MyComponent2._method', () => {
  it('returns true when called', () => {
    const wrapper = shallow(<MyComponent2 />);
    const instance = wrapper.instance();

    // spy on the instance instead of the component
    spyOn(instance, '_method').and.callThrough();

    expect(instance._method()).toBe(true);
    expect(instance._method).toHaveBeenCalled();
  });
});
```

### Тестирование логики методов компонента
Слишком специфическая задача, чтобы было возможно создать краткий пример.

Статьи на тему:
[How to Test React Components using Jest and Enzyme](https://blog.bitsrc.io/how-to-test-react-components-using-jest-and-enzyme-fab851a43875)
[How to test methods and callback using Mocha, Chai, & Enzyme in React-Redux](https://stackoverflow.com/questions/39396357/how-to-test-methods-and-callback-using-mocha-chai-enzyme-in-react-redux) у Enzyme api очень похож на mocha и chai

Наконец, если хочется проверить, диспатчит ли метод компонента action, то не нужно проверять логику
Нужно проверять state после вызова метода.

[Как вариант, смотреть store.getActions](https://stackoverflow.com/questions/48172819/testing-dispatched-actions-in-redux-thunk-with-jest?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

...или просто проверить вызывался ли `dispatch` с `toHaveBeenCalledWith`: 

```js
expect(instance.props.dispatch).toHaveBeenCalledWith({
  payload: { value: 'date', index }
});
```

### Еще раз, общая идея тестирования handleClick:

```js
it('doing something', () => {
  const initialState = {...};

  const { MockProviders, store } = createMockProviders({ store: initialState });

  store.dispatch = spy();

  const wrapper = Enzyme.mount(
    <MockProviders>
      <PaymentPage location={{ search: '?tariffId=2' }} />
    </MockProviders>
  );

  wrapper.find(Button).simulate('click');

  expect(store.dispatch.args.pop()).toEqual([
    postPay.request({
      companyId: 1,
      tariffId: 2,
      type: PAY_TYPE_ACQUIRING,
      url: `${apiConfig.baseUrl}${PAY_RESULT_URL}`
    })
  ])

})
```


### Тестируем стили для styled-components

Используем [jest-styled-components](https://github.com/styled-components/jest-styled-components)
Прекрасная библиотека с понятной документацией.

```js
it('text-align left for sender "Agent" and ltr direction', () => {
  const wrapper = Enzyme.mount(<StyledArticle sender={TICKET_SENDER_AGENT} data-rtl={'ltr'} />);
  expect(wrapper).toHaveStyleRule('text-align', 'left');
});
```

### Проверяем рендер текста в определнном блоке

```js
it('render date with format YYYY/MM/DD', () => {
  const article = {
    created_at: '2018-04-06T13:43:03.414Z'
  };

  const wrapper = Enzyme.shallow(<Article article={article} userEmail={'test@example.com'} />);

  expect(wrapper.find('.article-date').text()).toEqual('2018/04/06');
});
```

### Проверяем что строка рендерит html

```js
it('render html from string', () => {
  const article = {
    body: 'Some string with <h1>H1 tag</h1>'
  };

  const wrapper = Enzyme.shallow(<Article article={article} userEmail={'test@example.com'} />);

  expect(wrapper.find('.article-body').html()).toContain(`h1`);
});
```
