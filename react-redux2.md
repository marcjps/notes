# Redux

#### Concepts

* The entire application state is a single object
* A reducer function takes current state and action and returns a new state

#### Benefits

* Many components can use the same state easily
* No need to pass state changes to parent component using callbacks
* Code which changes state is cleanly separated in the reducers


#### Reducer

This object will accept our actions and state, and provide a new state based on the result of the action.  

We set the initial state during the first call, when no state parameter is supplied.

/reducers/WidgetReducer.js

```javascript
const initialState = { WidgetColour: 'blue' };

export default (state = initialState, action) => {
    console.log(action);
    switch (action.type) {
        case "SET_WIDGET_COLOUR":
            // create a new state.  do not mutate the state.
            return {
                ...state,
                WidgetColour: action.newColour
            };
        default:
            return state;
    }
    //return state;
};
```

#### Store

This is our store.  We pass in the reducer so the store API can handle our actions.

/store/index.js

```javascript
import { createStore } from "redux";
import reducer from "../reducers";
const store = createStore(reducer);
export default store;
```

#### Subscribing to store changes

This code in the root of our application calls render when the state changes.  

/index.js

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import store from './store';

const render = () => {
    return ReactDOM.render(<App />, document.getElementById("root"));
};
render();
store.subscribe(render);
serviceWorker.unregister();
```

#### Sending an action

We can import the store and send an action to change the state.  We send an action type and any other parameters we want, to describe the action.

/App.js

```javascript
changeColourRed = () => {
	store.dispatch({
		type: "SET_WIDGET_COLOUR",
		newColour: "red"
	});
};
```

In normal react code we make callback (using a prop which calls a function on the parent) to pass information back from child components to parent components.  That can get messy.  Using redux means we don't need to make callbacks, we change the global state in the store, and then any other component that is using the store will get updated.

#### Retrieving values from the store

We can retrieve values from the store and pass them to components as props.

/App.js

```javascript
<Widget colour={store.getState().WidgetColour}/>
```

#### Structuring the state

We need to optimise the structure of the state for the retrieval needs of the front-end application.  

It's possible that it looks something like the database table structure, various top level objects with properties underneath.

```json
{
    Widget: {
        Colour: "blue"
    },
    Wotsit: {
        Size: 16
    }
}
```

#### Splitting reducers

To keep the code clean, we need to split the reducer logic up sensibly, perhaps by the part of the state that it is dealing with.

Redux provides a helper called combineReducers which will merge separate reducers back into a single one.

The main reducer looks like this:

/reducers/index.js

```javascript
import { combineReducers } from "redux";
import Widget from './WidgetReducer'
import Wotsit from './WotsitReducer'
export default combineReducers({
    Widget,
    Wotsit,
});
```

Each individual reducer looks the same as the reducer example at the top of the page.  They implement only the actions for their piece of the state, and return only their piece of the state.

The state object then ends up componsed of these separate objects, which are accessed like this:

/App.js

```javascript
store.getState().Widget.Colour
store.getState().Wotsit.Size
```

#### Constant action names

To avoid us mistyping action name strings we can define them as const variables in a shared file.

/constants/action_types.js

```javascript
export const SET_WIDGET_COLOUR = "SET_WIDGET_COLOUR";
```

Then we can refer to this variable when sending the action, and in the reducer, instead of coding separate copies of the string in both places.

#### Move actions to action creators

To further organise the code we can create a separate JavaScript file that contains functions to create the actions.  Then from the application components can import the appropriate file and call the actions they require.

Perhaps we might also split the actions into separate files, mirroring the splitting of the reducers.

Here's an action creator:

/actions/index.js

```javascript
import {SET_WIDGET_COLOUR} from "../constants/action_types";

export const changeWidgetColour = (colour) => ({
    type: SET_WIDGET_COLOUR,
    newColour: colour
});
```

Then in our component code we can import the actions:

/App.js

```javascript
import {changeWidgetColour} from '../actions'
```

..and call them.

```javascript
handleChangeWidgetColourRed = () => {
    store.dispatch(changeWidgetColour('red'));
};

handleChangeWidgetColourGreen = () => {
    store.dispatch(changeWidgetColour('green'));
};

render = () => {
    return (
        <div>
                <span>Widget colours: </span>
                <button onClick={this.handleChangeWidgetColourRed}>Red</button>
                <span> </span>
                <button onClick={this.handleChangeWidgetColourGreen}>Green</button>
        </div>
    )
}
```

Note that in the examples, the action creator doesn't call the store.  It just returns the action and the calling component sends the action to the store.   I'm not quite sure why we would prefer that structure.

#### Selectors

To decouple components that are reading directly from the state, we can use selector functions to return items from the state.  Selectors take the current state as a parameter and return selected data.

In the relevent reducer we add the required selector functions:

/reducers/WidgetReducer.js

```javascript
export function getColour(state) {
    return state.Colour;
};
```

In the main reducer, near the combineReducers function, we need to expose all the individual reducers.  The main reducer is the only one that knows the full structure of the state so it has to send the appropriate section of state into the selectors.

/reducers/index.js

```javascript
import { combineReducers } from "redux";
import * as fromWidget from './WidgetReducer'
import * as fromWotsit from './WotsitReducer'

export default combineReducers({
    Widget: fromWidget.default,
    Wotsit: fromWotsit.default
});

// public selectors

export function getWidgetColour(store) {
    return fromWidget.getColour(store.Widget);
}

export function getWotsitSize(store) {
    return fromWotsit.getSize(store.Wotsit);
}
```

Now in the component we can import the selectors from the main reducer.

/App.js

```javascript
import {getWidgetColour, getWotsitSize} from './reducers'
```

...and call them.

```javascript
<Widget colour={getWidgetColour(store.getState())}/>
<Wotsit size={getWotsitSize(store.getState())}/>
```

### react-redux

react-redux is a helper library for binding our components to the store (via selectors and actions).

src/index.js

```javascript
import { Provider } from 'react-redux'
import store from './store'

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
);
```

Turn selectors into props using mapStateToProps:

```javascript
import * as selectors from '../reducers'

// the state here is the state from the store.  connect passes it to mapStateToProps and we pass it to the selector.
// this code calls the selectors and assigns the output values to props of Basket
const mapStateToProps = (state) => ({
    basketTotalPrice: selectors.getBasketTotalPrice(state),
    basketItems: selectors.getBasketItems(state)
});

export default connect(
    mapStateToProps,
    null
)(Basket)
```

Then use them in the usual way: this.props.basketTotalPrice

Turn actions into props using mapDispatchToProps:

```javascript
import {basketRemove, basketAdd} from "../actions";

const mapDispatchToProps = {
    basketAdd,
    basketRemove
};

// this code is turns each action creator into a function prop on Item.
// when you call the prop, react-redux calls dispatch for you, dispatching the action to the store
// (and then calling the reducer).
export default connect(
    null,
    mapDispatchToProps
)(Item)

```

Then call them in the usual way:

```javascript
this.props.basketRemove({
    id: 1,
    name: 'foo',
    price: 1.50
});
```

### Integrating a server side API

I haven't tried doing this yet.  This looks like an interesting approach: https://www.sohamkamani.com/blog/2016/06/05/redux-apis/

Or we might need this: https://github.com/reduxjs/redux-thunk

### Unit testing

The code created using the redux patterns above is highly unit testable since it mostly uses pure classes.  (classes with no state)

Install enzyme which allows us to test react components and subcomponents.

```text
npm install --save-dev enzyme enzyme-adapter-react-16
```

Create one *.test.js for each component.  At the top of each file, initialise enzyme:

```javascript
import Adapter from 'enzyme-adapter-react-16';
import {configure, shallow} from 'enzyme';

configure({ adapter: new Adapter() });
```


#### Testing components 

We do not want components to try to connect to the store.  The unit test aims to test the components only, not the store or the overall functionality.  In the unit test there is no store.

To avoid this, we:

* In each component, as well as having the default export (which uses the connect() HOC), also export the class (unconnected)
* Import the unconnected class into the unit test for testing.

The connected class may receive selector props from mapStateToProps.  In the unit test, we simply pass those props in directly, so we know exactly what "store" input the component has received and can test against it.

```javascript
import { Basket } from './Basket'    // import the unconnected class not the connected one.  The use of { Basket } specifies the unconnected export instead of the default one.

it('shows the correct total price when the value is 1', () => {
    const wrapper = shallow(<Basket
        basketTotalPrice={1}
        basketItems={ [ {
            id: 1,
            name: 'Test item',
            price: 1
        } ] }
    />);
    expect(wrapper.text().indexOf("Total price: £1.00") !== -1).toBeTruthy();    // text anywhere in the component
    expect(wrapper.find("p").at(1).text()).toEqual("Total price: £1.00"); // text is in the second p element
});
```
#### Writing assertions

The format for the assertions is:

expect ( the value from the component ).toPassSomeTest(...);

When developing assertions it is best to test that they do fail when given the wrong data.  It's very easy to watch an assertion pass and assume it is working correctly, and then later realise that the assertion doesn't fail when it is supposed to, so doesn't actually work at all.


# Testing for the presence of subcomponents

When we mount a component it will also mount any subcomponents it creates.  Because our subcomponents may be connected components, we need to use enzyme's shallow mount function.  This will mount the parent component only and not mount any of the child components, so they will not try to call connect().  Enzyme still returns a wrapper for the child component that we can write assertions against.  From that wrapper we can see the component type and the props, but not what it rendered (as it is not mounted it won't render anything).

Because the component returns the (unmounted) connected components, it is worth remembering to import the connected version of the sub components to write tests against (not the unconnected version, which is not equal to the connected one).

```javascript

import { Basket } from './Basket'   // because we are going to mount it, we need to import the unconnected class not the connected one.  The use of { Basket } specifies the unconnected export instead of the default export.
import Item from './Item' // because Item is imported by Basket, it is the connected class not the unconnected one.

it('contains 1 basket item with the correct props', () => {

    // the two props we pass in are normally created by mapStateToProps in the connected component.
    const wrapper = shallow(<Basket
        basketTotalPrice={1}
        basketItems={ [ {
            id: 1,
            name: 'Test item',
            price: 1
        } ] }
    />);

    const item = wrapper.find(Item);
    expect(item.prop("id")).toEqual(1);
    expect(item.prop("name")).toEqual("Test item");
    expect(item.prop("price")).toEqual(1);
});
```

#### Testing for actions called by a component

Our components may send actions to the store using mapDispatchToProps.

Since we have not connected the component, we can pass different props into the component in place of the action creator props that mapDispatchToStore would have passed.  Jest allows us to mock the function props and then we can subsequently test if they were called, and how they were called.

```javascript
it('has working button to add another item', () => {

    // we'll create dummy functions for the props that mapDispatchToProps would normally create
    const basketAdd = jest.fn();
    const basketRemove = jest.fn();

    const wrapper = shallow(<Item
        id={1}
        name="Test item"
        price={1}
        quantity={1}
        basketAdd={basketAdd}
        basketRemove={basketRemove}
    />);

    wrapper.find("button.add").simulate('click');  // find the add button using css class name selector
    expect(basketAdd).toHaveBeenCalled();

    let callParameters = basketAdd.mock.calls[0]; // get information about the first call to the function
    let itemToAdd = callParameters[0]; // get the first parameter passed to the function

    // check that the details passed to basketAdd match the details of the item we created
    expect(itemToAdd.id).toEqual(1);
    expect(itemToAdd.name).toEqual("Test item");
});
```

#### When using Enzyme to test components, debug() is your best friend

The wrappers that enzyme returns from its methods have a helper method called debug() which dumps the container of the wrapper into the console.  This is useful when developing unit tests, so you can see what is being returned.  (if you don't have a debugger)

#### Testing the reducer

Because it is a pure component, the reducer is extremely easy to test.

The reducer takes an existing state and an action as parameters, and then returns the new state.  In each unit test we can pass those in and test the result.

```javascript
it('should increase the quantity in the basket', () => {
    let newState = reducer( { items: [ {
            id: 1,
            name: "Fish",
            price: 1,
            quantity: 1
        }] }, {
        type: BASKET_ADD,
        item: {
            id: 1,
            name: "Fish",
            price: 1
        }
    });

    //console.log(newState);
    // there's a quantity of 2 now as 1 was already there and we added another
    expect(newState.items[0].quantity).toEqual(2);
});
```

#### Testing selectors

Selectors receive the state as an input and return a selected (or calculated) output, which also makes them easy to test.

```javascript
it('should calculate the total price correctly when there are two items', () => {

    let state = {
        items: [ {
            id: 1,
            name: "Fish",
            price: 1,
            quantity: 1
        },{
            id: 1,
            name: "Fish",
            price: 1,
            quantity: 1
        } ]
    };
    expect(getTotalPrice(state)).toEqual(2);
});
```


#### References

* https://medium.freecodecamp.org/understanding-redux-the-worlds-easiest-guide-to-beginning-redux-c695f45546f6
* https://hackernoon.com/selector-pattern-painless-redux-store-destructuring-bfc26b72b9ae
* https://react-redux.js.org/introduction/quick-start
* https://redux.js.org/recipes/writing-tests







