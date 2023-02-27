# NGRX

- [NGRX](#ngrx)
  - [What is NgRx?](#what-is-ngrx)
    - [Packages](#packages)
    - [State](#state)
    - [Data](#data)
    - [View](#view)
    - [Developer Tools](#developer-tools)
  - [Actions](#actions)
    - [Introduction](#actions-introduction)
    - [The Action interface](#actions-the-action-interface)
    - [Writing actions](#actions-writing-actions)
  - [Reducers](#reducers)
    - [Introduction](#reducers-introduction)
    - [The reducer function](#reducers-the-reducer-function)
      - [Defining the state shape](#defining-the-state-shape)
      - [Setting the initial State](#setting-the-initial-state)
      - [Creating the reducer function](#creating-the-reducer-function)
    - [Registering root state](#reducers-registering-root-state)
      - [Using the Standalone API](#root-state---using-the-standalone-api)
    - [Registering feature state](#reducers-registering-feature-state)
      - [Using the Standalone API](#feature-state---using-the-standalone-api)
  - [Effects](#effects)
    - [Introduction](#effects-introduction)
    - [Key Concepts](#effects-key-concepts)

## What is NgRx?

NgRx is a framework for building reactive applications in Angular. NgRX provides libraries for:

- Managing global and local state.
- Isolation of side effects to promote a cleaner component architecture.
- Entity collection management.
- Integration with the Angular Router.
- Developer tooling that enhances developer experience when building many different types of applications.

## Packages

NgRX packages are divided into a few main categories.

## State

- Store - RxJs powered global state management for Angular apps, inspired by Redux.
- Effects - Side effect model for @ngrx/store.
- Router Store - Bindings to connect the Angular Router to @ngrx/store.
- Entity - Entity State adapter for managing record collections.

## Data

- Data - Extension for simplified entity data management.

## View

- Component - Extension for building reactive Angular templates.

## Developer Tools

- Store Devtools - Instrumentation for @ngrx/store that enables visual tracking of state and time-travel debugging.
- Schematics - Scaffolding library for Angular applications using NgRx libraries.
- ESlint Plugin - ESLint rules to warn against bad practices. It also contains few automatic fixes to enforce a consistent style, and to promote best practice.

## Actions

Actions are one of the main building blocks in NgRx. Actions express _unique events_ that happen throughout your application. From user interaction with the page, external interaction through network requests, and direct interaction with device APIs, these and more events are described with actions.

## Actions. Introduction

Actions are used in many areas of NgRx. Actions are inputs and outputs of many systems in NgRx. Actions help you to understand how events are handled in your application.

## Actions. The Action interface

An `Action` in NgRx is made up of a simple interface:

```JS
//Action Interface
interface Action {
    type: string
}
```

The interface has a single property, the `type` , represented as a string. the `type` property is for describing the action that will be dispatched in your application. The value of the type comes in the form of `[Source] Event` and is used to provide a context of what category of action it is, and where an action was dispatched from. You add properties to an action to provide additional context or metadata for an action.

```JS
type: '[Auth API] Login Success'
```

This action describes an event triggered by a successful authentication after interacting with a backend API.

```JS
{
    type: '[Login Page] Login',
    username: string;
    password: string
}
```

This action describes an event triggered by a user clicking a login button from the login page to attempt to authenticate a user. The username and password are defined as additional metadata provided from the login page.

## Actions. Writing actions

There are a few rules to writing good actions within your application.

- Upfront - write actions before developing features to understand and gain a shared knowledge of the feature being implemented.
- Divide - categorize actions based on the event source.
- Many - actions are inexpensive to write, so the more actions you write, the better you express flows in your application.
- Event-Driven - capture events **not** commands as you are separating the description of an event and the handling of that event.
- Descriptive - provide context that are targeted to a unique event with more detailed information you can use to aid in debugging with the developer tools.

Example action of initiating a login request:

```JS
//login-page.actions.ts
import {
    createActions,
    props
} from '@ngrx/store';

export const login = createAction(
    '[Login Page] Login',
    props < {
        username: string;password: string
    } > ();
);
```

The `createAction` function returns a function, that when called returns an object in the shape of the `Action` interface. The `props` method is used to define any additional metadata needed for the handling of the action. Action creators provide a consistent, type-safe way to construct an action that is being dispatched.

Use the action creator to return the `Action` when dispatching.

```JS
//login-page.component.ts
onSubmit(username: string, password: string) {
    store.dispatch(login({
        username: username,
        password: password
    }));
}
```

The `login` action creator receives an object of `username` and `password` and returns a plain JavaScript object with a `type` property of `[Login Page] Login` , with `username` and `password` as additional properties.

The returned action has a very specific context about where the action came from and what event happened.

- The category of the action is captured within the square brackets `[]`.
- The category is used to group actions for a particular area, whether it be a component page, backend API, or browser API.
- The `Login` text after the category is a description about what event occurred from this action. In this case, the user clicked a login button from the login page to attempt to authenticate with a username and password.

Actions only responsibility are to express unique events and intents.

## Reducers

Reducers in NgRx are responsible for handling transitions from one state to the next state in your application. Reducer functions handle these transitions by determining which actions to handle based on the action's type.

## Reducers. Introduction

Reducers are pure functions in that they produce the same output for a given input. They are without side effects and handle each state transition synchronously. Each reducer function takes the latest `Action` dispatched, the current state, and determines whether to return a newly modified state or the original state.

## Reducers. The reducer function

There are a few consistent parts of every piece of state managed by a reducer.

- An interface or type that defines the shape of the state.
- The arguments including the initial state or current state and the current action.
- The functions that handle state changes for the associated actions(s).

Example of a set of actions that handle the state of a scoreboard, and the associated reducer function.

First, define some actions for interacting with a piece of state:

```JS
//scoreboard-page.actions.ts

import {
    createAction,
    props
} from '@ngrx/store';

export const homeScore = createAction('[Scoreboard Page] Home Score');
export const awayScore = createAction('[Scoreboard Page] Away Score');
export const resetScore = createAction('[Scoreboard Page] Score Reset');
export const setScores = createAction('[Scoreboard Page] Set Scores', props < {
    game: Game
} > ());
```

Then, create a reducer file that imports the actions and define a shape for the piece of state.

### Defining the state shape

Each reducer function is a listener of actions. The scoreboard actions defined above describe the possible transitions handled by the reducer. Import multiple sets of actions to handle additional state transitions within a reducer.

```JS
//scoreboard.reducer.ts

import {
    Action,
    createReducer,
    on
} from '@ngrx/store';
import * as ScoreboardPageActions from '../actions/scoreboard-page.actions';

export interface State {
    home: number;
    away: number
}
```

You define the shape of the state according to what you are capturing, whether it be a single type such as a number, or a more complex object with multiple properties.

### Setting the initial State

The initial state gives the state an initial value, or provides a value if the current state is `undefined` . You set the initial state with defaults for your required state properties.

Create and export a variable to capture the initial state with one or more defaults values.

```JS
//scoreboard.reducer.ts

export const initialState: State = {
    home: 0,
    away: 0,
}
```

The initial values for the `home` and `away` properties of the state are 0.

### Creating the reducer function

The reducer function's responsibility is to handle the state transitions in an immutable way. Create a reducer function that handles the actions for managing the state of the scoreboard using the `createReducer` function.

```JS
//scoreboard.reducer.ts

export cost scoreboardReducer = createReducer(
    initialState,
    on(ScoreboardPageActions.homeScore, state => ({
        ...state,
        home: state.home + 1
    })),
    on(ScoreboardPageActions.awayScore, state => ({
        ...state,
        away: state.away + 1
    })),
    on(ScoreboardPageActions.resetScore, state => ({
        home: 0,
        away: 0
    })),
    on(ScoreboardPageActions.setScores, (state, {
        game
    }) => ({
        home: game.home,
        away: game.away
    }))
)
```

Above, the reducer is handling 4 actions `[Scoreboard Page] Home Score` , `[Scoreboard Page] Away Score` , `[Scoreboard Page] Score Reset` , and `[Scoreboard Page] Set Scores` . Each action is strongly-typed. Each action handles the state transition immutably. This means that the state transitions are not modifying the original state, but are returning a new state object using the spread operator. The spread syntax copies the properties from the current state into the object, creating a new reference. This ensures that a new state is produced with each change, preserving the purity of the change. This also promotes referential integrity, guaranteeing that the old reference was discarded when a state change occurred.

> **_NOTE_**:
> The [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) only does shallow copying and does not handle deeply nested objects. You need to copy each level in the object to ensure immutability. There are libraries that handle deep copying including [lodash](https://lodash.com/) and [immer](https://github.com/immerjs/immer).

When an action is dispatched, _all registered reducers_ receive the action. Whether they handle the action is determined by the `on` functions that associate one or more actions with a given state change.

## Reducers. Registering root state

The state of your application is defined as one large object. Registering reducer functions to manager parts of your state only defines keys with associated values in the object. To register the global `state` within your application, use the `StoreModule.forRoot()` method with a map of key/value pairs that define your state. The `StoreModule.forRoot()` registers the global providers for your application, including the `Store` service you inject into your components and services to dispatch actions and select pieces of state.

```JS
//app.module.ts

import {
    NgModule
} from '@angular/core';
import {
    StoreModule
} from '@ngrx/store';
import {
    scoreboardReducer
} from './reducers/scoreboard.reducer';

@NgModule({
    imports: [
        StoreModule.forRoot({
            game: scoreboardReducer
        });
    ],
})

export class AppModule {}
```

Registering states with `StoreModule.forRoot()` ensures that the states are defined upon application startup. In general, you register root states that always need to be available to all areas of your application immediately.

### Root state - Using the Standalone API

Registering the root store and state can also be done using the standalone APIs if you are bootstrapping an Angular application using standalone features.

```JS
//main.ts

import {
    bootstrapApplication
} from '@angular/platform-browser';
import {
    provideStore,
    provideState
} from '@ngrx/store';

import {
    AppComponent
} from './app.component';
import {
    scoreboardReducer
} from './reducers/scoreboard.reducer';

bootstrapApplication(AppComponent, {
    providers: [
        provideStore(),
        provideState({
            game: scoreboardReducer
        })
    ],
});
```

> **Note:**
> Although you can register reducers in the `provideStore()` function, ngrx.io recommends keeping `provideStore()` empty and using the `provideState()` function to register feature states in the root `providers` array.

## Reducers. Registering feature state

Feature states behave in the same way root states do, but allow you to define them with specific feature areas in your application. Your state is one large object, and feature states register additional keys and values in that object.

Looking at an example state object, you see how a feature state allows your state to be built up incrementally. Start with an empty state object:

```JS
//app.module.ts

import {
    NgModule
} from '@angular/core';
import {
    StoreModule
} from '@ngrx/store';

@NgModule({
    imports: [
        StoreModule.forRoot({})
    ],
})

export class AppModule {}
```

Using the Standalone API:

```JS
//main.ts

import {
    bootstrapApplication
} from '@angular/platform-browser';
import {
    provideStore
} from '@ngrx/store';

import {
    AppComponent
} from './app.component';

bootstrapApplication(AppComponent, {
    providers: [
        providerStore()
    ],
});
```

This registers your application with an empty object for the root state.

```JS
{

}
```

Now use the `scoreboard` reducer with a feature `NgModule` named `ScoreboardModule` to register additional state.

```JS
//scoreboard.reducer.ts

export const scoreboardFeatureKey = 'game';
```

```JS
//scoreboard.module.ts

import {
    NgModule
} from '@angular/core';
import {
    StoreModule
} from '@ngrx/store';
import {
    scoreboardFeatureKey,
    scoreboardReducer
} from './reducers/scoreboard.reducer';

@NgModule({
    imports: [
        StoreModule.forFeature(scoreboardFeatureKey, scoreboardReducer)
    ],
})
export class ScoreboardModule {}
```

### Feature state - Using the Standalone API

Feature states are registered in the `providers` array of the route config.

```JS
//game-routs.ts

import { Route } from '@angular/router';
import { provideState } from '@ngrx/store';

import { scoreboardFeatureKey, scoreboardReducer } from './reducers/scoreboard.reducer';

export const routes: Route[] = [
  {
    path: 'scoreboard',
    providers: [
      provideState({ [scoreboardFeatureKey]: scoreboardReducer  })
    ]
  }
];
```

> **Note:**
> It is recommended to abstract a feature key string to prevent hardcoding strings when registering feature state and calling `createFeatureSelector`. Alternatively, you can use a [Feature Creator](https://ngrx.io/guide/store/feature-creators) which automatically generates selectors for your feature state.

Add the `ScoreboardModule` to the `AppModule` to load the state eagerly.

```JS
//app.module.ts

import { NgModule } from '@angular/core';
import { StoreModule } from '@ngrx/store';
import { ScoreboardModule } from './scoreboard/scoreboard.module';

@NgModule({
  imports: [
    StoreModule.forRoot({}),
    ScoreboardModule
  ],
})
export class AppModule {}
```

Using the Standalone API, register the feature state on application bootstrap.

```JS
//main.ts

import { bootstrapApplication } from '@angular/platform-browser';
import { provideStore } from '@ngrx/store';

import { AppComponent } from './app.component';
import { scoreboardFeatureKey, scoreboardReducer } from './reducers/scoreboard.reducer';

bootstrapApplication(AppComponent, {
  providers: [
    provideStore({ [scoreboardFeatureKey]: scoreboardReducer }),
  ]
});
```

After the feature is loaded, the `game` key becomes a property in the object and is now managed in the state.

```JS
{
  game: { home: 0, away: 0}
}
```

Whether your feature states are loaded eagerly or lazily depends on the needs of your application. You use feature states to build up your state object over time and through different feature areas.

## Effects

Effects are an RxJs powered side effect model for [Store](https://ngrx.io/guide/store). Effects use streams to provide [new sources](https://martinfowler.com/eaaDev/EventSourcing.html) of actions to reduce state based on external interactions such as network requests, web sockets messages and time-based events.

### Effects. Introduction

In a service-based Angular application, components are responsible for interacting with external directly through services. Instead, effects provide a way to interact with though services and isolate them from the components. Effects are where you handle tasks such as fetching data, long-running tasks that produce multiple events, and other external interactions where your components don't need explicit knowledge of these interactions.

### Effects. Key Concepts

- Effects isolate side effects from components, allowing for more _pure_ components that select state and dispatch actions.
- Effects are long-running services that listen to an observable of _every_ action dispatched from the Store.
- Effects filter those actions based on the type of action they are interested in. This is done by using an operator.
- Effects perform tasks, which are synchronous or asynchronous and return a new action.
