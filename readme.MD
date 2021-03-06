# Polymer Redux Example

## Installation

```bash
npm install --save redux
bower install --save polymer-redux
```

## Usage

### Boilerplate

Before importing Polymer Redux you must first include Redux to the applications
document.

```html
<html>
    <head>
    	<link rel="import" href="./bower_components/polymer/polymer.html">
        <script src="./bower_components/webcomponentsjs/webcomponents.js"></script>
        <script src="./node_modules/redux/dist/redux.js"></script>
        <script src="./bower_components/polymer-redux/polymer-redux.js"></script>        
    </head>
    <body>
        <!-- app -->
    </body>
</html>
```

### Setup

To bind Polymer components with Redux you must first create a ReduxBehavior
which wraps your application's store and decorates your elements. Simply set up
your Redux store as usual and then create the behavior with the `PolymerRedux`
constructor passing the store.

```javascript
initialState = {customer: {age: 30, name: 'Polymer Redux'}}
store = Redux.createStore(appReducer)
var ReduxBehavior = PolymerRedux(store)

function appReducer(state, action) {
    state = state || initialState
    var customer =
    {
        name: nameReducer(state.customer.name, action),
        age: ageReducer(state.customer.age, action)
    }
    return {customer: customer}
}

function nameReducer(state, action) {
    switch (action.type) {
        case 'update':
            return action.value || state
        default:
            return state;
    }
}

function ageReducer(state, action) {
    switch(action.type) {
        case 'increase':
            return state + 1
        case 'decrease':
            return state - 1
        default:
            return state
    }
}
```


### Binding Properties

Polymer Redux binds state to the components properties. This binding happens on
the `created` callback. To bind a property to a value in the state set the 
`statePath` key when defining properties in Polymer.

```javascript
Polymer({
    is: 'show-simplecustomer',
    behaviors: [ ReduxBehavior ],
    properties: {
        message: {
            type: Object,
            statePath: 'customer'
        }
    }
});
```


#### Dot Notation

Binding properties this way makes use of [`Polymer.Base.get()`](http://polymer.github.io/polymer/) method, so you can use dot notation paths like so: `'customer.name'`.

#### Two-way Bindings

Principle #2 of Redux's [Three Principles](http://redux.js.org/docs/introduction/ThreePrinciples.html),
says that state is read-only. Polymer however allows components to have two-way
binding via the `notify` flag. If the properties flagged with `notify` and have
`statePath` set, you will recieve a warning in your application runtime.

### Dispatching Actions

For an easier and semanatic way to dispatch actions against the store, is to create a list of actions the component can trigger. Adding a list of functions to the `actions` property, exposes them to the `dispatch()` method of the element. Or you can just simply `dispatch` an action object with `type` name.

Add 2 buttons to increase/decrease the `customer.age`, and add one input field and button to update `customer.name`.

```html
<dom-module id="show-customer">
    <template>
        <h1>Hello, <span>[[customer.name]]</span></h1>
        <h2>Age: <span>[[customer.age]]</span></h2>
        <div>
            <button id="increaseButton">+</button>
            <button id="decreaseButton">-</button>
        </div>
        <br/>
        <input id="nameTextField" placeHolder="new name"></input>
        <button id="updateButton">update</button>
    </template>
</dom-module>
```

Add event handlers for buttons and input field.

```javascript
Polymer({
    is: 'show-customer',
    behaviors: [ ReduxBehavior ],
    properties: {
        customer: {
            type: Object,
            statePath: 'customer'
        }
    },
    listeners: {
        'increaseButton.click': '_handleIncrease',
        'decreaseButton.click': '_handleDecrease',
        'updateButton.click': '_handleUpdate',
        'nameTextField.keypress': '_handleKeypress'
    },
    _handleIncrease: function() {
        this.dispatch({type: 'increase'})
    },
    _handleDecrease: function() {
        this.dispatch({type: 'decrease'})
    },
    _handleUpdate: function() {
        this.dispatch({type: 'update', value: this.$.nameTextField.value})
        this.$.nameTextField.value = ''
    },
    _handleKeypress: function(e) {
        if(e.which === 13 && !!e.currentTarget.value.trim()) {
            this._handleUpdate();
        }
    }
});
```

`dispatch()` also takes a function that returns a action object.

```javascript
Polymer({
//...
    handleClick: function() {
        this.dispatch(function() {
            return {
               type: 'ACTION'
            };
        });
    }
});
```

### Make it Work
Just simply declare the custom `Polymer` element as below.

```html
<show-customer></show-customer>
```

