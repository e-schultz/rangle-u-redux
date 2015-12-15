# Rangle-U Redux

# What is Redux?

Redux is a predictable state container for JavaScript apps. It is a framework agnostic library that. It can be used with [Angular with ng-redux](https://github.com/wbuchwalter/ng-redux), [Angular 2 with ng2-redux](https://github.com/wbuchwalter/ng2-redux), [React with react-redux](https://github.com/rackt/react-redux).

One of the reasons why Rangle.io is starting to adopt Redux on most of our applications, is because we are starting to get more React projects - and wanting to have a core piece of our stack be useable between Angular, Angular 2 and React.

When Redux is used properly - a large part of your application code becomes framework agonostic - and is just pure JavaScript, with the framework specific aspects of your code-base being pretty much the view layer.

# Some Resources

* [Managing State with Redux and Angular](http://blog.rangle.io/managing-state-redux-angular/) - Blog post I did for Rangle
* [ng-sumit-redux](https://github.com/e-schultz/ng-summit-redux) - Sample application built for Angular Summit
* [ng-summit-slides](https://github.com/e-schultz/ng-summit-slides) - Slides from Angular Summit on Redux + Angular
* [Redux Documentaton](http://redux.js.org/)
* [Dan Abramov Egghead.io Redux Course](https://egghead.io/lessons/javascript-redux-the-single-immutable-state-tree)
* [Awsome Redux Resources](https://github.com/xgrommx/awesome-redux)
* [ng-redux](https://github.com/wbuchwalter/ng-redux) - Angular bindings for Redux
* [ng2-redux](https://github.com/wbuchwalter/ng2-redux) - Angular2 bindings for Redux
* [react-redux](https://github.com/rackt/react-redux) - React bindings for Redux
* [Angular Redux Starter](https://github.com/rangle/angular-redux-starter) - Our starter project for Angular + Redux
* [React Redux Starter](https://github.com/rangle/react-redux-starter) - Our starter project for React + Redux

# The Agenda 

* Global Application State
* [Reducers 101](#reducers-101)
* [Reducers with Redux](#reducers-with-redux)
* [Unit testing Reducers](#unit-testing-reducers)
* [Actions with Redux](#actions-with-redux)
* [Unit testing actions](#unit-testing-an-action)
* [Async actions with Redux](#async-actions)
* [ng-redux](#ng-redux)
  * [Configuring $ngReduxProvider](#configuring-ngreduxprovider)
  * [$ngRedux.connect](#ngreduxconnect)
* Components and Containers

# Reducers 101

At the core of redux, is the idea of using Reducers to manage application state. So, lets just have a brief recape of what a reducer is.

A reducer is simply a function that itterates over a collection of items, and returns a single result. The reducer function takes in an accumulator - the final value that you want, a value - the current item in the collection, and can also provide an initial value.

The classic example of a reducer is doing a sum:

```javascript
const result = [1,2,3].reduce((acc,value) => acc+value, 0);
// result is 6
console.log(result);
```
[jsbin](https://jsbin.com/waqoto/edit?js,console)

However, the result of the reducer does not need to be the same type of the items in the collection. For example:

```javascript
const result = [1,2,3].reduce((acc,value) => {
  acc.sum += value;
  acc.values.push(value);
  return acc;
}, {sum: 0, values: []});

/*
result is:
{
  sum: 6,
  values: [1, 2, 3]
}
*/
console.log(result);
```
[jsbin](https://jsbin.com/wuralew/edit?js,console)

# Reducers with Redux

Redux uses reducers to manage your application state, and expects a function like:

```javascript
let todoState = (state = [], action = {}) => {
  switch (action.type) {
  case 'TODO_ADDED':

    return [action.payload, ...state];
  case 'TODO_COMPLETED':

    return state.map(todo => 
        todo.id === action.payload.id ?
        Object.assign({}, todo, { completed: !todo.completed }) : todo
        );
  default:
    return [...state];
  }
}
```
[jsbin](https://jsbin.com/gafufi/edit?js,console)

One of the key things to keep in mind with Reducers in Redux - is that they should be pure functions, have no side effects, and not mutate the state being passed in.

If you mutate the state object, this can cause issues with change detection, and being able to check the equality of objects, and lead to some hard to debug issues later on down the line.

There are various ways to avoid mutating state, and many applications are adopting [Immutable](https://facebook.github.io/immutable-js/) to ensure this.

However, it could also be worth watching 

* [Avoiding Array Mutations with concat, slice and ...spread](https://egghead.io/lessons/javascript-redux-avoiding-array-mutations-with-concat-slice-and-spread)
* [Avoiding Object Mutations with Object.assign and ...spread](https://egghead.io/lessons/javascript-redux-avoiding-object-mutations-with-object-assign-and-spread)

# Unit Testing Reducers 

Since reducers are pure functions, you can easily setup an initial state and pass in an action. This makes creating unit tests for your reducers easy - and often you do not need to be concerned with Angular, React or Redux in creating your tests.

```javascript
it('should allow parties to join the lineup', () => {
    const initialState = lineup();
    const expectedState = [{
      partyId: 1,
      numberOfPeople: 2
      }];

    const partyJoined = {
      type: PARTY_JOINED,
      payload: {
        partyId: 1,
        numberOfPeople: 2
      }
    };

    const nextState = lineup(initialState, partyJoined);
    expect(nextState).to.deep.equal(expectedState);

  });
```

# Actions with Redux

* Should return plain JSON objects
* .....unless using middleware
* Are where your side effects happen
* Are where you deal with async

Actions in Redux should return plain JSON objects that represent something that has happened in the system. When using middleware (which will be covered later), you can have actions that return promises/etc to deal with Async behavior - however even then, the result of the promise should be a plain JSON object.

This is because we want the actions to be replayable and seralizable. 

I like to think of actions as being broken into two parts:

* Action Creators - or Commands, the request to do something.
* Events - the result of what was done.

The action creator is a wrapper functiont that takes in some paramaters, does a bit of logic - and the resulting object is a result of what happened.

If you view the result as an event that happened, what Redux does is then re-plays these events over the reducers to be able to form your application state.


```javascript
const generateId = () => Math.floor((1 + Math.random()) * 0x10000).toString(16).substring(1)

export function joinLine(numberOfPeople) {
  return {
    type: PARTY_JOINED,
    payload: {
      partyId: generateId(), // <-- side effect/impure 
      numberOfPeople: parseInt(numberOfPeople, 10)
    }
  };

}
```

We do not want to have the generation of IDs/etc being handled in the reducer. We want to be able to replay the actions and end up in the same application state in a predictable way. If the generateId() was handled in the reducer, this would not be possible.

# Unit Testing an Action

When your actions are returning plain JavaScript objects, testing the logic in them is very simple. If you need to have async actions, or actions that require access to the state - the complexity will increase slightly and we will cover that in another section.

```javascript
it('should create an action for joining the line', () => {
  const action = lineupActions.joinLine(4);
  expect(action.payload.numberOfPeople).to.equal(4);
});
```

# Async Actions

* Need to use a middleware, such as [redux-thunk], or [redux-promise], that allows you to turn something other than a plain javascript object.

* Will give you access to dispatch, and getState

```javascript
import * as types from '../constants/ActionTypes';

function selectReddit(reddit) {
  return {
    type: types.SELECT_REDDIT,
    reddit
  };
}

function invalidateReddit(reddit) {
  return {
    type: types.INVALIDATE_REDDIT,
    reddit
  };
}

function requestPosts(reddit) {
  return {
    type: types.REQUEST_POSTS,
    reddit
  };
}

function receivePosts(reddit, json) {
  return {
    type: types.RECEIVE_POSTS,
    reddit: reddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  };
}

export default function asyncService($http) {
  function fetchPosts(reddit) {
    return dispatch => {
      dispatch(requestPosts(reddit));
      return $http.get(`http://www.reddit.com/r/${reddit}.json`)
        .then(response => response.data)
        .then(json => dispatch(receivePosts(reddit, json)));
    };
  }

  function shouldFetchPosts(state, reddit) {
    const posts = state.postsByReddit[reddit];
    if (!posts) {
      return true;
    }
    if (posts.isFetching) {
      return false;
    }
    return posts.didInvalidate;
  }

  function fetchPostsIfNeeded(reddit) {
    return (dispatch, getState) => {
      if (shouldFetchPosts(getState(), reddit)) {
        return dispatch(fetchPosts(reddit));
      }
    };
  }

  return {
    selectReddit,
    invalidateReddit,
    fetchPostsIfNeeded
  };
}
```
[async service example from ng-redux](https://github.com/wbuchwalter/ng-redux/blob/master/examples/async/actions/asyncService.js)
# Unit Testing Async Actions/actions that need dispatch

```javascript
function someAction(data) {
  return function(dispatch) {
    const promise = new Promise((resolve, reject) => {
      resolve({
        type: 'ACTION',
        payload: {
          data
        }
      });
    });

    return promise.then(result => dispatch(result))
  }
}


describe('some async action', () => {
  it('should check things', done => {
    let test = result => {
      expect(result.payload.data).to.be.equal('hello');
      done();
    }
    someAction('hello')(test);
  });
})
```
Or, use [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store)

```javascript
import configureStore from 'redux-mock-store';

const middlewares = []; // add your middlewares like `redux-thunk`
const mockStore = configureStore(middlewares);

// Test in mocha
it('should dispatch action', (done) => {
  const getState = {}; // initial state of the store
  const action = { type: 'ADD_TODO' };
  const expectedActions = [action];

  const store = mockStore(getState, expectedActions, done);
  store.dispatch(action);
})
```

# Middleware

Out of scope for today, but a good [blog post on middleware explained](https://medium.com/@meagle/understanding-87566abcfb7a#.mlmw2ggnx).

But for interest sake, a basic logging middleware:

```javascript
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

# ng-redux

## Configuring $ngReduxProvider

```javascript
import reducers from './reducers';
import createLogger from 'redux-logger';

const logger = createLogger({ level: 'info', collapsed: true });

export default angular
.module('app', [ngRedux, ngUiRouter])
.config(($ngReduxProvider) => {
  $ngReduxProvider
    .createStoreWith(reducers       // our application state
    , ['ngUiRouterMiddleware',      // middleware - that supports DI
    logger                          // middleware - that doesn't need DI
    ]);
});
```

## $ngRedux.connect

[ng-redux docs on connect](https://github.com/wbuchwalter/ng-redux#connectmapstatetotarget-mapdispatchtotargettarget)


### The API

```javascript
$ngRedux.connect(mapStateToTarget, [mapDispatchToTarget])(target)
```



### In practice

```javascript
import lineupActions from '../../actions/lineup-actions';

export default class LineupController {
  constructor($ngRedux, $scope) {

    let disconnect = $ngRedux.connect(
        state => this.onUpdate(state),  // What we want to map to our target
        lineupActions                   // Actions we want to map to our target
        )(this);                        // Our target
    
    $scope.$on('$destroy', disconnect); // Cleaning house
  }

  onUpdate(state) {
    return {
      parties: state.lineup
    };
  }
};
```

onUpdate
----

Whenever an action gets dispatched in redux, and there is a change in the application state - ngRedux will execute the function that you provided for the mapStateToTarget. 

This expects a plain javascript object to be returned. ng-redux will do a shallow check to see if the result of this function has changed since the last time it was called, and if so - will push the object onto your target.

In this case, the `state.lineup` property will be mapped onto `this.parties`. If you try to return a non-plain object, such as a class, or an immutable object - ngRedux will throw an error.

To access this in your template:

```html
<tr ng-repeat="party in lineup.parties">
  <td>
    {{party.partyId}}
  </td>
  <td>
    {{party.numberOfPeople}}
  </td>
</tr>
```

actions 
-----

ng-redux will also take care of mapping your actions to your target. In this example, the lineup actions exported are:

```javascript
// from lineup-actions.js
export default {
  joinLine, leaveLine
};
```

This means that on our target, we will be able to access `this.joinLine`, and `this.leaveLine`, for example:

```html
<button type="button" ng-click="lineup.joinLine(lineup.numberOfPeople)">New Party</button>
```

# Selectors

* [reselect](https://github.com/rackt/reselect), From their docs...

> * Selectors can compute derived data, allowing Redux to store the minimal possible state.
> * Selectors are efficient. A selector is not recomputed unless one of its arguments change.
> * Selectors are composable. They can be used as input to other selectors.

Selectors can also act as a way to help decouple knowledge about the state tree, from the component using it. 

For example, if your component(s) require data in a certian format - the selector can take care of that transformation for you. If you need to change the structure of your state tree, it can be easyier to update your selector to do that transformation, and have unit tests ensuring that the resulting data is the same instead of needing to re-work the entire component to understand the new structure.

Example selector from sad-ui,

```javascript
import {createSelectorCreator} from 'reselect';
import {is, List, fromJS} from 'immutable';

const immutableCreateSelector = createSelectorCreator(is);

export const filterSelector = state => state.filter.get('ip');
export const dataIpSelector = state => state.data.getIn(['ip', 'data'], List());

export const ipGraphSelector = immutableCreateSelector(
  [
    filterSelector,
    dataIpSelector
  ],
  (filters, dataset) => {
    return fromJS({
      activeFilters: filters,
      dataset: dataset
        .reduce((acc, i) => {
          return acc.push(List([
            i.get('count'),
            i.get('ip'),
            i.get('country')
          ]));

        }, List())
    });
  }
);
```

...and unit testing

```javascript
import {fromJS, List, Map} from 'immutable';
import {ipGraphSelector} from './ip-graph-selector';

var createState = (state) => ipGraphSelector(state);

describe('ipGraphSelector', () => {
  it('should return a list of lists in the format of count, ip, country', () => {
    var state = createState({
      data: fromJS({
        'ip': {
          data: [{
            ip: '123.456.789.10',
            country: 'CA',
            count: 100
          }, {
            country: 'US',
            ip: '123.456.789.9',
            count: 99
          }]
        }
      }),
      filter: fromJS({
        filterSource: 'ipGraph',
        id: 'ip',
        type: 'mustContain',
        values: {}
      })
    });

    expect(state.getIn(['dataset', 0, 0])).to.equal(100);
    expect(state.getIn(['dataset', 0, 1])).to.equal('123.456.789.10');
    expect(state.getIn(['dataset', 0, 2])).to.equal('CA');
    expect(state.getIn(['dataset', 1, 0])).to.equal(99);
    expect(state.getIn(['dataset', 1, 1])).to.equal('123.456.789.9');
    expect(state.getIn(['dataset', 1, 2])).to.equal('US');
  });
});
```

# Smart/Dumb Components, Containers

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">Container Components</th>
            <th scope="col" style="text-align:left">Presentational Components</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">Location</th>
          <td>Top level, route handlers</td>
          <td>Middle and leaf components</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Aware of Redux</th>
          <td>Yes</th>
          <td>No</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">To read data</th>
          <td>Subscribe to Redux state</td>
          <td>Read data from props</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">To change data</th>
          <td>Dispatch Redux actions</td>
          <td>Invoke callbacks from props</td>
        </tr>
    </tbody>
</table>

[redux docs - Usage With React](http://redux.js.org/docs/basics/UsageWithReact.html)

Keeping with this separation seems to be a little more natural/easy with React, but the same concepts can apply to Angular. 

* Smart containers that are aware of redux
* Pass down the data and actions that need to be used
* The dumb components are responsible for just displaying the data

An example from SAD-UI:

Smart Componet - the IP Graph:

```javascript
/* beautify preserve:start */
import {List} from 'immutable';
import {ipGraphSelector} from '../../selectors/ip-graph/ip-graph-selector';
import filterActions from '../../actions/filter/filter-actions';
/* beautify preserve:end */

export default class IpGraphController {
  constructor($ngRedux, $scope) {

    let _onChange = (state) => {
      const data = ipGraphSelector(state);

      return {
        activeFilters: data.get('activeFilters'),
        dataset: data.get('dataset')
      };
    };

    let disconnect = $ngRedux.connect(_onChange, filterActions)(this);

    $scope.$on('$destroy', () => disconnect());

    this.columnHeaders = List([
      'IP_GRAPH.HEADERS.RANK',
      'IP_GRAPH.HEADERS.ATTEMPTS',
      'IP_GRAPH.HEADERS.IP',
      'IP_GRAPH.HEADERS.COUNTY'
    ]);

    this.fnColumnCallbacks = [
      angular.noop,
      (value) => this.toggleFilter('ip', value),
      (value) => this.toggleFilter('country', value)
    ];

    this.classList = List([
      null,
      'list-graph__list--hover',
      'list-graph__list--hover'
    ]);
  }

  isSelectedOrEmpty(obj) {
    return this.activeFilters.getIn(['values', obj.get(1)]) ||
           this.activeFilters.get('values').size <= 0;
  }
}


```

```html
<list-graph
    column-headers="ipGraph.columnHeaders"
    dataset="ipGraph.dataset"
    active-filters="ipGraph.activeFilters"
    fn-column-callbacks="ipGraph.fnColumnCallbacks"
    class-list="ipGraph.classList"
    fn-check-selected="ipGraph.isSelectedOrEmpty">
</list-graph>
```

The list-graph dumb component:

```javascript
import angular from 'angular';
import invariant from 'invariant';

export default class ListGraphController {
  constructor() {

    invariant(
      !(angular.isDefined(this.fnRowCallback) && angular.isDefined(this.fnColumnCallbacks)),
      'Inside the List Graph component, both fnRowCallback and fnColumnCallbacks are defined, only one is allowed.',
      this
    );

    invariant(
      !(angular.isDefined(this.classList) && angular.isDefined(this.rowClass)),
      'Inside the List Graph component, both classList and rowClass are defined, only one is allowed.',
      this
    );
  }
}

ListGraphController.$inject = [];

```

```html
<table class="list-graph__content">
  <thead>
    <tr class="list-graph__list list-graph__list--title">
      <th
        ng-repeat="header in listGraph.columnHeaders | mutable"
        class="list-graph__list--entry-title"
        ng-class="{'list-graph__list--num-title': $index === 0}">
        {{ header | translate }}
      </th>
    </tr>
  </thead>
  <tbody>
    <tr
      class="list-graph__list {{ listGraph.rowClass }}"
      ng-class="{'list-graph__list--selected': listGraph.fnCheckSelected(data)}"
      ng-repeat="data in listGraph.dataset | mutable"
      ng-click="listGraph.fnRowCallback(data)">
      <td class="list-graph__list--num">
        {{ $index + 1 }}
      </td>
      <td
        class="list-graph__list--entry {{ listGraph.classList.get($index) }}"
        ng-click="listGraph.fnColumnCallbacks[$index](item)"
        ng-repeat="item in data | mutable track by $index"
        title="{{ item }}">
        {{ item }}
      </td>
    </tr>
  </tbody>
</table>

```


# What logic goes where?



