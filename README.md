# MobX

## _Simple, scalable state management_

[![Build Status](https://travis-ci.org/mobxjs/mobx.svg?branch=master)](https://travis-ci.org/mobxjs/mobx)
[![Coverage Status](https://coveralls.io/repos/mobxjs/mobx/badge.svg?branch=master&service=github)](https://coveralls.io/github/mobxjs/mobx?branch=master)
[![Join the chat at https://gitter.im/mobxjs/mobx](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/mobxjs/mobx?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![#mobservable channel on reactiflux discord](https://img.shields.io/badge/discord-%23mobx%20%40reactiflux-blue.svg)](https://discord.gg/0ZcbPKXt5bYAa2J1)

`npm install mobx --save`

MobX is a battle tested library that makes state management simple and scalable by transparently applying functional reactive programming (TFRP).
The philosophy behind MobX is very simple:

_Everything that can be derived from the application state, should be derived_.

This includes the UI, data serialization, server communication etc.
If React allows you to declaratively define your component tree and uses the virtual DOM as black box to minify the amount of DOM mutations,
then MobX is the library that allows you to declaratively define your state model and derivations in a declarative manner, and provides the black box to make your wishes come true.

## MobX was formerly known as Mobservable.
To use install pre- 2.0 `mobx*` compatible packages, use `mobservable` instead of `mobx`.
See the [changelog](https://github.com/mobxjs/mobx/blob/master/CHANGELOG.md) for all the details.

## Core concepts

MobX has a few core concepts. The following snippets can be tried online using [JSiddle](https://jsfiddle.net/mweststrate/wv3yopo0/).

### Observable state

MobX adds observable capabilities to existing data structures like objects, arrays and class instances. This can simply be done by annotating your class properties with the [@observable](http://mobxjs.github.io/mobx/refguide/observable-decorator.html) decorator (ES2015), or by invoking the [`observable`](http://mobxjs.github.io/mobx/refguide/observable.html) or [`extendObservable`](http://mobxjs.github.io/mobx/refguide/extend-observable.html) functions (ES5).

```javascript
class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}
```

Using `@observable` is like turning a value into a spreadsheet cell. But unlike spreadsheets, these values can be not just primitive values, but references, objects and arrays as well. You can even [define your own](http://mobxjs.github.io/mobx/refguide/extending.html) observable data sources.

### Reactive derivations

With MobX you can simply define derived values that will update automatically when relevant data is modified. For example by using the [`@computed`](http://mobxjs.github.io/mobx/refguide/computed-decorator.html) decorator or by using parameterless functions as property values in `extendObservable`.

```javascript
class TodoList {

    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}
```
MobX will ensure that `unfinishedTodoCount` is updated automatically when a todo is added or when one of the `finished` properties is modified.
Computations like these can very well be compared with formulas in spreadsheet programs like MS Excel. They update automatically whenever, and only when, needed.

### Reactions

A reaction is a bit similar to a computed value, but instead of producing a new value it produces a side effect.
Reactions bridge reactive and imperative programming for things like printing to the console, making network requests, incrementally updating the React component tree to patch the DOM, etc.

Reactions can be created by using the [`autorun`](http://mobxjs.github.io/mobx/refguide/autorun.html), [`autorunAsync`](http://mobxjs.github.io/mobx/refguide/autorun-async.html) or [`when`](http://mobxjs.github.io/mobx/refguide/when.html) functions.
Or, if you are using for example ReactJS, you can turn your (stateless function) components into reactive components by simply slapping the [`@observer`](http://mobxjs.github.io/mobx/refguide/observer-component.html) decorator from the `mobx-react` package onto them.

```javascript
import React, {Component} from 'react';
import {observer} from "mobx-react";

@observer
class TodoListView extends Component {
    render() {
        return <div>
            <ul>
                {this.props.todoList.todos.map(todo =>
                    <TodoView todo={todo} key={todo.id} />
                )}
            </ul>
            Tasks left: {this.props.todoList.unfinishedTodoCount}
        </div>
    }
}

const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
);

const store = new TodoList();
React.render(<TodoListView todoList={store} />, document.getElementById('mount'));
```

`observer` turns React (function) components into derivations of the data they render.
When using MobX there are no smart or dumb components.
All components render smartly but are defined in a dumb manner. MobX will simple make sure the components are always re-rendered whenever needed,
but also no more than that.
So the `onClick` handler in the above example will force the proper `TodoView` to render,
and it will cause the `TodoListView` to render if the amount of unfinished tasks has changed.

However, if you would remove the `Tasks left` line (or put it into a separate component), the `TodoListView` will no longer re-render when ticking a box.
You can verify this yourself by changing the [JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/).

### Actions

Unlike many flux frameworks, MobX is unopinionated about how user events should be handled.
This can be done in a Flux like manner.
Or by processing events using RxJS.
Or by simply handling events in the most straightforward way possible, as demonstrated in the above `onClick` handler.
In the end it all boils down to: Somehow the state should be updated.
After updating the state, `MobX` will take care of the rest, in an efficient, glitch-free manner.
So simple statements like below are enough to automatically update the user interface.
There is no technical need for firing events, calling dispatcher or what more.
A React component is in the end nothing more than a fancy representation of your state.
A derivation that will be managed by MobX.

```javascript
store.todos.push(
    new Todo("Get Coffee"),
    new Todo("Write simpler code")
);
store.todos[0].finished = true;
```

## MobX: Simple and scalable

MobX is one of the least obtrusive libraries you can use for state management. That makes the `MobX` approach not just simple, but very scalable as well:

### Using classes and real references
With MobX you don't need to normalize your data. This makes the library very suitable for very complex domain models (At Mendix for example ~500 different domain classes in a single application).

### Referential integrity is guaranteed
Since data doesn't need to be normalized, and MobX automatically tracks the relations between state and derivations, you get referential integrity for free. Rendering something that is accessed through three levels of indirection? No problem, MobX will track them and re-render whenever one of the references changes. As a result staleness bugs are a thing of the past. As a programmer you might forget that changing some data might influence an seemingly unrelated component in a corner case. MobX won't forget.

### Simpler actions are easier to maintain
As demonstrated above, modifying state when using MobX is very straightforward. You simply write down your intentions. MobX will take care of the rest.

### Fine grained observability is efficient
MobX builds a graph of all the derivations in your application to find the least amount of re-computations that is needed to prevent staleness. "Derive everything" might sound expensive, MobX builds a virtual derivation graph to minimize the amount re-computations need to keep derivations in sync with the state. In fact, when testing MobX at Mendix we found out that using this library to track the relations in our code is often a lot more efficient then pushing changes through our application by using handwritten events or "smart" selector based container components.
The simple reason is that MobX will establish far more fine grained 'listeners' on your data then you would do as a programmer.
Secondly MobX sees the causality between derivations so it can order them in such a way that no derivation has to run twice or introduces a glitch.
How that works? See this [in-depth explanation of MobX](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254#.a2j1rww8g).

## Credits

MobX is inspired by reactive programming principles as found in spreadsheets. It is inspired by MVVM frameworks like in MeteorJS tracker, knockout and Vue.js. But MobX brings Transparent Functional Reactive Programming to the next level and provides a stand alone implementation. It implements TFRP in a glitch-free, synchronous, predicatable and efficient manner.

And a lot of credits for [Mendix](https://github.com/mendix), for providing the flexibility and support to maintain MobX and the chance to proof the philosophy of MobX in a real, complex, performance critical applications.

## Further examples and documentation

* [Full API documentation](http://mobxjs.github.io/mobx/)
* Boilerplate and example projects for ES5, Babel, Typescript can be found in the [MobX](https://github.com/mobxjs) organization on github.

Blogs:
* [Making React reactive: the pursuit of high performing, easily maintainable React apps](https://www.mendix.com/tech-blog/making-react-reactive-pursuit-high-performing-easily-maintainable-react-apps/)
* [Becoming fully reactive: an in-depth explanation of Mobservable](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254#.a2j1rww8g)
* [Pure rendering in the light of time and state](https://medium.com/@mweststrate/pure-rendering-in-the-light-of-time-and-state-4b537d8d40b1)
* [SurviveJS interview on MobX, React and Flux](http://survivejs.com/blog/mobservable-interview/)

## What others are saying...

> _Elegant! I love it!_
> &dash; Johan den Haan, CTO of Mendix

> _We ported the book Notes and Kanban examples to Mobservable. Check out [the source](https://github.com/survivejs/mobservable-demo) to see how this worked out. Compared to the original I was definitely positively surprised. MobX seems like a good fit for these problems._
> &dash; Juho Vepsäläinen, author of "SurviveJS - Webpack and React" and jster.net curator

> _Great job with MobX! Really gives current conventions and libraries a run for their money._
> &dash; Daniel Dunderfelt

> _I was reluctant to abandon immutable data and the PureRenderMixin, but I no longer have any reservations. I can't think of any reason not to do things the simple, elegant way you have demonstrated._
> &dash;David Schalk, fpcomplete.com

## Contributing

* Feel free to send pr requests.
* Use `npm test` to run the basic test suite, `npm run coverage` for the test suite with coverage and `npm run perf` for the performance tests.
