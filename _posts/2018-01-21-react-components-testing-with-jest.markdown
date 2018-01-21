---
layout: post
title:  "Testing React components with Jest"
date:   2018-01-21 21:10:57 +0200
categories: jest unit-testing
---
Hello there.

Unit testing is really important as it helps to identify errors at the really early stage of development and to save our time in a long time perspective.

And for now Jest is treated as most suitable tool for React application testing. Let's take a look at few simple cases to be tested.
Here's really simple component. It's accepts single number as prop, renders it, and increases it whenever the '+' button is clicked.

```jsx
import React, { Component } from 'react';

export default class Counter extends Component {
  constructor({number}) {
    super();
    this.state = {
      number
    };
  }

  increaseNumber() {
    this.setState({
      number: ++this.state.number
    })
  }

  render() {
    const {
      number
    } = this.state;

    return (
      <div>
        <div className="number">{number}</div>
        <button onClick={this.increaseNumber.bind(this)}>+</button>
      </div>
    )
  }
}
```
## Testing with snapshots.

The idea behind snapshot testing is creation of rendered React view in JSON format. Then this snapshot will be stored as ```__snapshots__/**.js.snap__``` file in 
test directory. Then Jest will be rendering the same component during every next test run of the fly and comparing it to a file-stored JSON.

Let's take a look how would test file look like in case of our Counter component:

```jsx
import React from 'react';
import Counter from './Counter';
import renderer from 'react-test-renderer';

describe('Counter component', () => {
  const tree = renderer
    .create(<Counter number={2} />)
    .toJSON();

  it('should be rendered correctly', () => {
    expect(tree).toMatchSnapshot();
  });
});
```
That will generate the following snapshot when we run jest command for the first time.

```
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`Counter component should be rendered correctly 1`] = `
<div>
  <div
    className="number"
  >
    2
  </div>
  <button
    onClick={[Function]}
  >
    +
  </button>
</div>
`;

```
And that is it! You just write simple unit test which is 90% similar to any other component's test except it's name and passed props. And run ```jest``` in your terminal.
Please notice that we used ```react-test-renderer``` package to avoid using real browser for component's rendering.

What would happen if we change our component slightly? Let's add ```className``` prop to our component's container :

```jsx
    ...
     number
    } = this.state;
    
    return (
      <div className="counter">
        {number}
        <button onClick={this.increaseNumber.bind(this)}>+</button>
      </div>
    )
    ...
```
And re-run ```jest``` in terminal:

```
  ● Counter component › should be rendered correctly

    expect(value).toMatchSnapshot()
    
    Received value does not match stored snapshot 1.
    
    - Snapshot
    + Received
    
    @@ -1,6 +1,8 @@
    - <div>
    + <div
    +   className="counter"
    + >
        <div
          className="number"
        >
          2
        </div>

      10 | 
      11 |   test('should be rendered correctly', () => {
    > 12 |     expect(tree).toMatchSnapshot();
      13 |   });
      14 | 
      15 |   test('number should be increased in DOM after + is clicked', () => {
      
      at Object.<anonymous> (src/Counter.test.js:12:18)

 › 1 snapshot test failed.
Snapshot Summary
 › 1 snapshot test failed in 1 test suite. Inspect your code changes or run `npm test -u` to update them.

```

So our test provided us with information that ```Counter``` component has been updated. But snapshot hasn't and we need to run our tests this way: ```jest -- -u```.
```
PASS  src/Counter.test.js
  Counter component
    ✓ should be rendered correctly (7ms)

 › 1 obsolete snapshot removed.
  - should be rendered correctly 1
Snapshot Summary
 › 1 obsolete snapshot removed.

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   1 passed, 1 total
Time:        0.9s, estimated 1s
Ran all test suites.

```
That's almost everything you will be doing while covering your components with snapshots tests. Write routine test, run ```jest```, run ```jest -- -u``` if previous has failed.
Do you think it's simple? You're right! But I have notice it's not really helpful. You're just got noticed whenever your changes update one or several components. That's it! So for my personal opinion snapshot testing is some kind of thing that doesn't save your much time (like other kinds of testing do). But doesn't cost your much time eather.
So I would suggest to cover your component with snapshot tests AND DOM testing.

## DOM Testing

DOM Testing helps us make sure that our component updates appropriately after some user's interactions. I.e. let's check if ```number``` rendered value updates on page after plus button is clicked.
We will use Enzyme testing utility.

First of all let's configure Enzyme to use React 16 adapder. It's good pracrice to place test config to a separate file. I created ```src/preRunTest.js``` file for that purpose.

```jsx
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

configure({adapter: new Adapter()});
```
  And placed jest configuration into my ```package.json``` file to point out on this file.
  
```JSON
  ...
  ,
  "jest": {
    "setupTestFrameworkScriptFile": "./src/preRunTest.js"
  }
  ...
```
And here're my updates for ```Counter.test.js```
```jsx
...
import { mount } from 'enzyme';
...

...
  test('number should be increased in DOM after + is clicked', () => {
    const counterComponent = mount(<Counter number={2} />);
    const button = counterComponent.find('button');
    const numberContainer = counterComponent.find('.number');

    expect(numberContainer.text()).toEqual('2');

    button.simulate('click');

    expect(numberContainer.text()).toEqual('3');
  });
});

```
So we mounted our Counter component with ```number={}``` propery. Checked that we have 2 string rendered in ```.number``` element, triggered click on ```button``` element and checked that rendered value got increased.

## Business logic testing.

I also prefer to cover components business logic methods with unit tests to make it possible to ensure nothing's broken at any given point of time.

So lets add some logic to our number rendering. I want to show message that prints how many times button was clicked. And i want see just zero for ```number === 0```,
```button clicked 1 time``` for ```number === 1``` case, and ```Button was clicked ${number} times!!!``` for the rest.
Now ```Counter.test.js``` should look like this:

```jsx
import React from 'react';
import Counter from './Counter';
import {mount, shallow} from 'enzyme';
import renderer from 'react-test-renderer';

describe('Counter component', () => {
  const tree = renderer
    .create(<Counter number={2} />)
    .toJSON();

  test('should be rendered correctly', () => {
    expect(tree).toMatchSnapshot();
  });

  test('number should be increased in DOM after + is clicked', () => {
    const counterComponent = mount(<Counter number={2} />);
    const button = counterComponent.find('button');
    const numberContainer = counterComponent.find('.number');

    expect(numberContainer.text()).toEqual('Button was clicked 2 times!!!');

    button.simulate('click');

    expect(numberContainer.text()).toEqual('Button was clicked 3 times!!!');
  });

  test('should return appropriate messages for different numbers', () => {
    const counter = shallow(<Counter number={0}/>).instance();

    expect(counter.getMessage(0)).toEqual('zero');
    expect(counter.getMessage(1)).toEqual('Button clicked 1 time.');
    expect(counter.getMessage(999)).toEqual('Button was clicked 999 times!!!');
  });
});
```
And don't forget to update your snapshot!

Complete example of testing of Counter component with webpack setup can be found [in this repo](https://github.com/lidak/jest-example). Thanks. Take care.
