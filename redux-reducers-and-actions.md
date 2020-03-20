At this point, we have a clear understanding of why we might want to set up a centralized state in our application. We also know that Redux gives us clear rules regarding how this state will need to be updated, so that we can very confidently track how and why our state got updated, and what the history of that state is. But how do we set this state up, and what are the tools that we need in order to update it?

## Learning Objectives

- Illustrate what reducers are conceptually and how they update our store
- Describe the responsibilities of actions and action creators within our Redux apps
- Compare and contrast pure vs. impure functions
- Illustrate how reducers, actions, and action creators interact within our app
- Build a simple Redux app that updates our store with state

## Updating our Redux Store

Before we get started, let's first get a quick overview of Redux. Redux is composed of three main entities: a store, actions, and reducers.

* The **store** is an object that holds all your application state, and your application can read the application state from it.
* **Actions** are objects that alert our application that something happened in the app (a user clicked something, submitted a form, etc), and detail how the state should change. They often contain new data.
* **Reducers** are functions which receive the current state and an action and create a new state based on the received action.

This flow of information to update our state and, ultimately, our webpage, is represented in the below diagram. We'll be digging further into each of those pieces throughout this article.

![redux-machine-diagram][redux-machine-diagram]

#### Getting Set Up

The Launch crew loves to have shuffleboard competitions, but all that team rivalry can mean it's important to have a trustworthy score-keeping app. Throughout this article, we'll be building a Shuffleboard score-keeping app!

To code along with the latter part of this article, run the following commands:

```no-highlight
et get redux-reducers-and-actions
open index.html
```

The webpage that opens in your browser should look like the below. You should have no errors in your console, and clicking on the "Add point" buttons should not change your page at all.

![Scorekeeper Initial App][scorekeeper-initial-app]

### Redux is built upon predictable behavior

One of the most fundamental pillars of the Redux library is that it is built upon predictable behavior: that is, we can rest easy in knowing what our initial state will be, how it can be updated, and that we will ALWAYS get in terms of expected results. Redux allows us to trust that there will be no side effects based on race conditions caused by the asynchronicity of our application (and we know that our JavaScript applications contain a whole lot of asynchronous code!). We are able to do this by relying on very simple JavaScript concepts. The data held in our store is just plain data, in the form of one large JavaScript object. Our changes that we make to this data will be passed in as plain JavaScript objects as well. And we will pass both of these things, our prior state and the changes we want to make to it, into one pure JavaScript function in order to be given back a new object with our updated data.

#### Pure vs Impure JavaScript functions

As a review, let's talk a little bit about pure vs. impure JavaScript functions.

A "pure" JS function is a function that is given arguments, and can always be expected to output the same information when given the same arguments. No side effects and no mutations. In fact, in a React context, we have seen this with functional `const` components! When given the same props, we expect these components to output the same exact thing on our page. The same applies to any pure function: we expect them to simply spit out an output based on the arguments provided, without any side effects.

A good example of a pure JavaScript function is one which adds 1 to a number, as follows:

```javascript
const plusOne = (number) => {
  return (number + 1)
}
```

No matter what, if we hand in an argument of `3` to the above function, it will return a value of `4`. Likewise, `plusOne(5)` will _always_ return a value of `6`. There are no external factors or side effects of executing this function.

On the other hand, an impure function is one that may have side effects or alter the output based on external feedback. Additionally, it may mutate the value of our original arguments. For example, we can write the below function `addOneToAll` which takes an array of numbers as its argument:

```javascript
const addOneToAll = (numbers) => {
  numbers.forEach((number, index) => {
    numbers[index] = number + 1
  })
}
```

This function is taking our original argument, `numbers`, and actually changing that array to add 1 to each number, so that we would get the following output:

```javascript
const addOneToAll = (numbers) => {
  console.log(numbers); // Logs [1, 2, 3]
  numbers.forEach((number, index) => {
    numbers[index] = number + 1
  })
  console.log(numbers); // Logs [2, 3, 4]
}
```

Mutating the original array makes this function impure, and can result in unforeseen, additional side effects!

Another type of impure function is one which alters the output based on external feedback. For example, if we have a `handleSubmit` function that runs a `'POST'` fetch call, the output of the function will be dependent upon the success of the fetch call, so we can no longer consider this a pure function.

## Reducers and Actions

While our store is where our state lives, a `reducer` is the function that is responsible for declaring our initial state, and handling any updates to the state in our store. It does so by consuming an `action`, which is what we call the simple object that gets passed into our reducer as an argument.

### Setting up our Reducer

Since we want our store to have a trackable, predictable state, we need to make sure we are using pure functions to make any necessary updates to our store. We will call these pure functions `reducers`. We will need to have a reducer defined in order to create our store.

Reducers are simply pure functions that take in two arguments: the current state, and an object, which we will call an _action_.

The first thing we will want to do is define what our initial state should be. We can do that by adding the following code to your `main.js` file:

```javascript
const initialState = {
  playerOneScore: 0,
  playerTwoScore: 0
}
```

As we can see, we're setting up our initial state just like we might define `this.state` in a React component: as a JavaScript object with two key-value pairs.

Once we define what we want our initial state to be, we can start to set up our reducer. Remember that our reducer is just a pure function, which we'll call `scoreReducer`. We're going to pass in the current state and have it default to our `initialState` upon initial load, and then pass in our `action` as our second argument.

```javascript
const scoreReducer = (state = initialState, action) => {
  // ...the contents of our reducer function will go here
}
```

Before we move forward with setting up our reducer, let's jump over to defining an action so that we can affect change on our state.

### Creating our Actions

As we already mentioned, our actions are going to be the second argument that we pass in to our reducer. They are simply going to be a JavaScript object that tells our reducer how our state needs to be updated. We will set up a different action for each use case in which we need to update our state!

There are two key-value pairs that our reducer expects to get inside of our action: what _type_ of action it is, and any necessary _data_, or _payload_. For example, if we wanted to add a new grocery to our grocery list:

```javascript
{
  type: ADD_GROCERY,
  grocery: 'Corn'
}
```

The _type_ is required in every single action we use, but a _payload_ is only required if there's some additional information we need to pass to our reducer. For example, a we may supply a user's input in a form, in the above case, the grocery to be added. In the example of our simple score-keeping app, we don't need to pass in any additional information as we will just be increasing the score by 1 each time we click the button, so we simply need to set up two _types_ of actions: `ADD_PLAYER_ONE_POINT` AND `ADD_PLAYER_TWO_POINT`.

In order to get our buttons functioning, we first need to target those buttons so we can add an event listener. Add the following code into `main.js` in order to do so:

```javascript
const playerOneScoreButton = document.getElementById('add-player-one-point')
const playerTwoScoreButton = document.getElementById('add-player-two-point')
```

Now that we have found our buttons, we can add an event listener to each. The event listener for player one's "Add Point" button will add a point to our state for player one, and likewise for player two. This means that each of the buttons need to dispatch an action to our reducer, so our reducer can update the state appropriately.

In order to pass these actions through to our reducer to be processed, we need to use the `dispatch` function given to us by the Redux library. By calling `dispatch(action)`, our app knows to take that action and run it through our reducer to make any necessary updates to state.

Go ahead and add the below code to your `main.js` file:

```javascript
const ADD_PLAYER_ONE_POINT = 'ADD_PLAYER_ONE_POINT'
const ADD_PLAYER_TWO_POINT = 'ADD_PLAYER_TWO_POINT'

playerOneScoreButton.addEventListener('click', () => {
  store.dispatch({
    type: ADD_PLAYER_ONE_POINT
  })
})

playerTwoScoreButton.addEventListener('click', () => {
  store.dispatch({
    type: ADD_PLAYER_TWO_POINT
  })
})
```

We can see that `playerOneScoreButton` now dispatches an action with type `ADD_PLAYER_ONE_POINT`, and `playerTwoScoreButton` dispatches an action with type `ADD_PLAYER_TWO_POINT`. With our actions in place, we can return to our reducer and write some code around what it should do when it receives each type of action.

_A note:_ we always want to define our action types as constants, and then use that constant in our action and reducer. This has many benefits, including that it will make it easier to namespace/organize our action types, to see what action types we have in our application all in one handy place, and to update the action type if we wanted to change it throughout our codebase. It will also save some space once our code is minified. The most important benefit, however, is that it will help us write bug-free code by preventing silly typo errors! If we used strings in our actions and reducers, our code will simply error out fairly silently in the browser. However, if we instead use a constant, we will get a nice loud `ReferenceError`, and our code will fail to compile, alerting us to our typo immediately.

### Filling in our Reducer

As we originally set up, we will want our reducer to have the below structure. If you haven't already put this into your `main.js` file, you can do so now!

```javascript
const scoreReducer = (state = initialState, action) => {
  // ...the contents of our reducer function will go here
}
```

Now that our reducer is declared, we can begin to fill in what we want to happen for each action type.

The implementation of our reducer is essentially just one large `switch` statement, which checks the type of the action that was dispatched, and decides how to update state accordingly. As a reminder, a `switch` statement is essentially just a shorthand way to write out longer `if...else` conditional statements. For more information on `switch` statements, feel free to dig into the [MDN switch statement docs][mdn-switch].

We can fill in the inside of our reducer so it matches the following:

 ```javascript
 const scoreReducer = (state = initialState, action) => {
   switch(action.type) {
     case ADD_PLAYER_ONE_POINT:
       const playerOneNewScore = state.playerOneScore + 1
       return Object.assign({}, state, {
         playerOneScore: playerOneNewScore
       })
     case ADD_PLAYER_TWO_POINT:
       const playerTwoNewScore = state.playerTwoScore + 1
       return Object.assign({}, state, {
         playerTwoScore: playerTwoNewScore
       })
     default:
       return state;
   }
 }
 ```

 If we look closely at the above, we can see that our `switch` statement is looking at the `action.type` for each action it receives. If it receives an action with type `ADD_PLAYER_ONE_POINT`, it will update our state to add one to the `playerOneScore`. Likewise, if it receives an action with type `ADD_PLAYER_TWO_POINT`, it will update our state to add one to the `playerTwoScore`. Finally, if for some reason the action type does not match either of the types specified, it will simply return the state without making any changes.

 You will notice that instead of reassigning or overwriting our state using `state = ...`, we are using `Object.assign` to return an _entirely new object_. This ties back to what we said about _pure functions_ earlier in this reading: that in order for a function to be pure, it should never mutate our original argument, but rather return a new copy of the argument. Since we know our reducer is a _pure function_, we need to make sure it is not mutating the `state` that's passed in, but rather, returning a new copy of that state! Every single change we make gives us back an entirely new version of our state. The fact that _Redux state is immutable_ and we never want to overwrite it is a fundamental rule of Redux, and a huge part of why Redux provides us with so much predictability. It also provides some useful developer friendly functionalities, such as the ability to look back at previous versions of our state, which can be a huge help when debugging our application.

 We can see that we're passing three arguments into `Object.assign`: `{}`, `state`, and our new object with the state we want to update. What's happening here is that we're using the ES6 function `Object.assign` to tell our reducer to create a new object (`{}`), fill it with the existing `state`, and then add in our new object. It's important to note that the properties are overwritten in the order they appear, so if our original state has a key-value pair with a key of `playerOneScore`, it will be overwritten by our new `playerOneScore` since we provide the new score as the _last_ argument.

We have now created our actions (and hooked them up to our buttons) and our reducer. Our final step is to set up our store, and update our HTML page with our new score.

## Setting up our Store

Currently, we have a `scoreReducer` and event listeners that are dispatching actions, so we have all of the pieces we need to make *updates* to our store. Our next step is to actually set up that store so that we have a place for our application-wide state to live!

The first step to set up our store will be importing this functionality from Redux. We can do so by adding the below line at the top of our `main.js` file:

```javascript
const { createStore } = Redux;
```

Note that we need to make sure we wrap `createStore` in curly brackets when importing. We will go into *why* we are doing this a bit later in this curriculum, but for now, just know that we need to do this in order for our reference to work!

Then, we also want to use this `createStore` function to create a store using our `scoreReducer`. Note that we _always_ need to hand in a reducer as an argument to `createStore`. We can do so by adding the following to the bottom of our file:

```javascript
const store = createStore(scoreReducer);
```

Finally, let's create our `render` method below. The render method hooks our store up to that method so that it runs every time our store is updated, and runs it:

```javascript
const render = () => {
  console.log(store.getState());
}

render();
store.subscribe(render);
```

Here, we are using two methods that go along with our store. First, we are defining what should happen when `render` gets called (currently, it should `console.log` our current state). To access the state, we use the Redux function `store.getState()`. Then, we are invoking `render()` on initial load. And finally, we are using the Redux function `store.subscribe` to make it so that our `render` function re-runs each and every time our store gets updated. Subscribe adds a listener to our store, so that after any time an action is dispatched to our reducer, our `render` method will get called. You can think of it as subscribing to a newsletter: any time a new copy of the newsletter goes out, you receive a copy of that new newsletter in your inbox. Similarly, anytime our store gets updated, our `render` method will get the updated state and re-run. If you are interested in diving into more of the detail of how these functions are working behind the scenes, you can check out the Redux docs on [createStore][create-store] and [subscribe][redux-subscribe]. A wonderful example of code explaining these internals can also be found [in this Gist][redux-gist].

At this point, we should hard refresh our Scorekeeper page. When we open up our JavaScript console, we will see that our buttons are interactive: each time we click one of our buttons, the state that's being logged in our console updates the appropriate player's score!

### Making our page display our state

Of course, we don't want our users to have to open up the console in order to see their scores. The final step in making our page fully functional is to display each score in the proper place on our page!

If we look at our `index.html` file, we can see that underneath the header for each player's section, there is an empty `<p>` tag, each with an id relating to the player. We can target these `<p>` tags in order to make the scores show up on the page.

Let's add the below code into `main.js` above our `render` function to find those `<p>` tags:

```javascript
const playerOneScoreSection = document.getElementById('player-one-score')
const playerTwoScoreSection = document.getElementById('player-two-score')
```

And finally, rather than `console.log`'ing our state, let's add it into the innerHTML of our `<p>` tags by updating our `render` function as follows:

```javascript
const render = () => {
  playerOneScoreSection.innerHTML = store.getState().playerOneScore
  playerTwoScoreSection.innerHTML = store.getState().playerTwoScore
}
```

And that's it! If we hard refresh our page again, we can see that our scores show on our page, starting at 0 and incrementing as we click the appropriate "Add Point" buttons.

Your final code should look like this:
```javascript
const { createStore } = Redux;

const initialState = {
  playerOneScore: 0,
  playerTwoScore: 0
}

const scoreReducer = (state = initialState, action) => {
  switch(action.type) {
    case ADD_PLAYER_ONE_POINT:
      const playerOneNewScore = state.playerOneScore + 1
      return Object.assign({}, state, {
        playerOneScore: playerOneNewScore
      })
    case ADD_PLAYER_TWO_POINT:
      const playerTwoNewScore = state.playerTwoScore + 1
      return Object.assign({}, state, {
        playerTwoScore: playerTwoNewScore
      })
    default:
      return state;
  }
}

const playerOneScoreButton = document.getElementById('add-player-one-point')
const playerTwoScoreButton = document.getElementById('add-player-two-point')

const ADD_PLAYER_ONE_POINT = 'ADD_PLAYER_ONE_POINT'
const ADD_PLAYER_TWO_POINT = 'ADD_PLAYER_TWO_POINT'

playerOneScoreButton.addEventListener('click', () => {
  store.dispatch({
    type: ADD_PLAYER_ONE_POINT
  })
})

playerTwoScoreButton.addEventListener('click', () => {
  store.dispatch({
    type: ADD_PLAYER_TWO_POINT
  })
})

const store = createStore(scoreReducer);
const playerOneScoreSection = document.getElementById('player-one-score')
const playerTwoScoreSection = document.getElementById('player-two-score')

const render = () => {
  playerOneScoreSection.innerHTML = store.getState().playerOneScore
  playerTwoScoreSection.innerHTML = store.getState().playerTwoScore
}

render();
store.subscribe(render);
```

## Wrapping It Up

We have now implemented a working application using Redux! We set up our initial state and made rules on how to change it in our **reducer**, created our **store** using `createStore`, put together **actions** so that our reducer knows when to change our state, and used `getState()` and `.subscribe` to make sure our page is updating with the new score each time a button gets clicked.

[scorekeeper-initial-app]: https://s3.amazonaws.com/horizon-production/images/redux-scorekeeper-initial-app.png
[redux-machine-diagram]:https://s3.amazonaws.com/horizon-production/images/redux-machine-diagram.png
[mdn-switch]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
[create-store]:(https://redux.js.org/api-reference/createstore)
[redux-subscribe]:https://redux.js.org/api-reference/store#subscribe-listener
[redux-gist]:https://gist.github.com/Chiamaka/e04e8f0c701d6f03f20773b50df79888
