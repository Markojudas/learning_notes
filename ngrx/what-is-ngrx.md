# What is NgRx?

NgRx is a framework for building reactive applications in Angular. NgRX provides libraries for:

* Managing global and local state.
* Isolation of side effects to promote a cleaner component architecture.
* Entity collection management.
* Integration with the Angular Router.
* Developer tooling that enhances developer experience when building many different types of applications.

## Packages

NgRX packages are divided into a few main categories.

## State

* Store - RxJs powered global state management for Angular apps, inspired by Redux.
* Effects - Side effect model for @ngrx/store.
* Router Store - Bindings to connect the Angular Router to @ngrx/store.
* Entity - Entity State adapter for managing record collections.

## Data

* Data - Extension for simplified entity data management.

## View

* Component - Extension for building reactive Angular templates.

## Developer Tools

* Store Devtools - Instrumentation for @ngrx/store that enables visual tracking of state and time-travel debugging.
* Schematics - Scaffolding library for Angular applications using NgRx libraries.
* ESlint Plugin - ESLint rules to warn against bad practices. It also contains few automatic fixes to enforce a consistent style, and to promote best practice.

## Actions

Actions are one of the main building blocks in NgRx. Actions express *unique events* that happen throughout your application. From user interaction with the page, external interaction through network requests, and direct interaction with device APIs, these and more events are described with actions.\

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

* Upfront - write actions before developing features to understand and gain a shared knowledge of the feature being implemented.
* Divide - categorize actions based on the event source.
* Many - actions are inexpensive to write, so the more actions you write, the better you express flows in your application.
* Event-Driven - capture events **not** commands as you are separating the description of an event and the handling of that event.
* Descriptive - provide context that are targeted to a unique event with more detailed information you can use to aid in debugging with the developer tools.

Example action of initiating a login request:

```JS
//login-page.actions.ts
import {} from '@ngrx/store';

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

* The category of the action is captured within the square brackets `[]`.
* The category is used to group actions for a particular area, whether it be a component page, backend API, or browser API.
* The `Login` text after the category is a description about what event occurred from this action. In this case, the user clicked a login button from the login page to attempt to authenticate with a username and password.

Actions only responsibility are to express unique events and intents.

## Reducers

Reducers in NgRx are responsible for handling transitions from one state to the next state in your application. Reducer functions handle these transitions by determining which actions to handle based on the action's type.

## Reducers. Introduction

Reducers are pure functions in that they produce the same output for a given input. They are without side effects and handle each state transition synchronously. Each reducer function takes the latest `Action` dispatched, the current state, and determines whether to return a newly modified state or the original state.

## Reducers. The reducer function

There are a few consistent parts of every piece of state managed by a reducer.

* An interface or type that defines the shape of the state.
* The arguments including the initial state or current state and the current action.
* The functions that handle state changes for the associated actions(s).

Example of a set of actions that handle the state of a scoreboard, and the associated reducer function.

First, define some actions for interacting with a piece of state:

```JS
//scoreboard-page.actions.ts

import {} from '@ngrx/store';

export const homeScore = createAction('[Scoreboard Page] Home Score');
export const awayScore = createAction('[Scoreboard Page] Away Score');
export const resetScore = createAction('[Scoreboard Page] Score Reset');
export const setScores = createAction('[Scoreboard Page] Set Scores', props < {
    game: Game
} > ());
```
