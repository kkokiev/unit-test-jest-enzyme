# mapStateToProps & mapDispatchToProps

[Большая статья на тему](https://jsramblings.com/2018/01/15/3-ways-to-test-mapStateToProps-and-mapDispatchToProps.html)

Автор склоняется к третьему варианту.
- Каждое свойство `mapStateToProps` должно представлять собой отдельный селектор в отдельном файле
- Каждое свойство `mapDispatchToProps` должно представлять собой отдельную функцию в отдельном файле
- Тесты пишутся для selectors и action creators

`selectors.js`
```js
export const getLastRolledNumber = (state) => (state.lastRolledNumber);
```

`selectors.test.js`
```js
import { getLastRolledNumber } from './selectors';

describe('Dice selectors', () => {
    it('should select last rolled number from state', () => {
        const state = { lastRolledNumber: 7 };
        expect(getLastRolledNumber(state)).toBe(7);
    });

});
```

`actionCreators.js`
```js
export const rollDice = () => {
    return { type: 'ROLL_DICE' };
};
```

`actions.test.js`
```js
import { rollDice } from './actionCreators';

describe('Dice actions', () => {
    it('should dispatch ROLL_DICE action', () => {
        expect(rollDice()).toEqual({ type: 'ROLL_DICE'});
    })
});
```
