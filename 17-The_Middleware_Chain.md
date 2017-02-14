# 17. 미들웨어 체인
[비디오 링크](https://egghead.io/lessons/javascript-redux-the-middleware-chain)

지난 강좌에서, 일반적이지 않는 행동을 추가하기 위해 `dispatch`함수를 감싸는 두개의 함수를 작성 했다. 그 것들이 같이 어떻게 동작하는지 좀 더 자세히 알아 보자.

```javascript
const configureStore = () => {
  const store = createStore(todoApp);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.dispatch = addPromiseSupportToDispatch(store);

  return store;
};
```

`store`를 반환하기 전 `dispatch` 함수의 마지막 버전은 `addPromiseSupportToDispatch`를  호출한 결과다.

#### `addPromiseSupportToDispatch` 이전:
```javascript
const addPromiseSupportToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(rawDispatch);
    }
    return rawDispatch(action);
  };
};
```

`addPromiseSupportToDispatch`에 의해 반환된 함수는 일반적인 `dispatch`함수 처럼 동작하지만, Promise를 받으면 응답을 받을 때까지 기다리고나서 `rawDispatch`에 결과를 전달 한다 (`rawDispatch`는 `store.dispatch`의 이전 값 이다).

Promise가 아닌 경우에 대해서, 함수는 바로 `rawDispatch`를 호출한다. `rawDispatch`는 `addPromiseSupportToDispatch`가 호출될 당시의 store.dispatch에 해당된다.

### `dispatch` 리팩토링 - 향상된 함수들

`store.dispatch`가 일찍(`configureStore`의 내부)이 다시 할당 되었기 때문에, `addPromiseSupportToDispatch` 내부에서 `rawDispatch`로 참조 하는것은 완전히 공정하지 않다.

`rawDispatch`를 `next`로 다시 이름을 지을 것 인데, 이유는 이 함수가 체인에서 다음번 `dispatch` 함수 이기 때문이다.

#### `addPromiseSupportToDispatch` 이후:
```javascript
const addPromiseSupportToDispatch = (store) => {
  const next = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(next);
    }
    return next(action);
  };
};
```

위, `next`는 `addLoggingToDispatch()`로 부터 반환된 `store.dispatch`를 참조 한다.

`addLoggingToDispatch()` 함수 또한 원본 `dispatch` 함수와 같은 API로 함수를 반환 하지만 액션 `type`, 이전 `state`, 다음 `state`를 기록 한다는 것을 상기 해라.

`addLoggingToDispatch`가 호출될 당시에 `store.dispatch`에 해당하는 `rawDispatch`를 호출 한다. 이경우, 이것은 `configureStore`안에 있는 `createStore()`에 의해 제공되는 `store.dispatch` 다.

하지만, 로그를 기록 하는 것을 추가 하기 이전에 `dispatch` 함수를 오버라이드 하기 원할 수 있는데 생각해 볼만한 일 이다.

일관성을 위해, 여기서도 역시 `rawDispatch`를 `next`로 이름을 변경 할 것 이다. 이 특정한 경우에, `next`는 원본 `store.dispatch`를 가르킨다.

#### `addLoggingToDispatch()`
```javascript
const addLoggingToDispatch = (store) => {
  const next = store.dispatch;
  if (!console.group) {
    return next;
  }

  return (action) => {
    console.group(action.type);
    console.log('%c prev state', 'color: gray', store.getState());
    console.log('%c action', 'color: blue', action);

    const returnValue = next(action);

    console.log('%c next state', 'color: green', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  };
};
```

### 미들웨어 함수 소개

While this method of extending the store works, it's not really great that we override the public API and replace it with custom functions.

To get away from this pattern, we will declare an array of _middleware functions_, which is just a fancy name for the extra-functionality functions we wrote.

This `middlewares` array will contain functions to be applied later as a single step.

We'll push `addLoggingToDispatch` and `addPromiseSupportToDispatch` to the middleware array.

Now we create a function `wrapDispatchWithMiddlewares()` that takes the `store` as the first argument, and the array of middlewares as the second.

#### Refactoring `configureStore`
```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];

  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(logger);
  }

  wrapDispatchWithMiddlewares(store, middlewares);

  return store;
};
```

Inside of `wrapDispatchWithMiddlewares()` we're going to use `middlewares`'s `forEach` method to run some code for every middleware.

Specifically, we will override the `store.dispatch` function to point to the result of calling the middleware with the `store` as an argument.

```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store);
  );
```

Recall that inside of our middleware functions themselves, there is a certain pattern that we have to repeat. We grabbing the value of `store.dispatch` and store it in a variable called `next` that we call later.

To make it a part of the middleware contract, we can make `next` an outside argument, just like the `store` before it and the `action` after it.

#### Updating `addLoggingToDispatch()`
```javascript
const addLoggingToDispatch = (store) => {
  return (next) => {
    if (!console.group) {
      return next;
    }

    return (action) => {
      console.group(action.type);
      console.log('%c prev state', 'color: gray', store.getState());
      console.log('%c action', 'color: blue', action);

      const returnValue = next(action);

      console.log('%c next state', 'color: green', store.getState());
      console.groupEnd(action.type);
      return returnValue;
    };
  }
};
```

With this change, the middleware becomes a function that returns a function that returns a function.

This pattern is called **currying**. This is not very common in JavaScript, but is actually very common in functional programming languages.

#### Updating `addPromiseSupportToDispatch`:
```javascript
const addPromiseSupportToDispatch = (store) => {
  return (next) => {
    return (action) => {
      if (typeof action.then === 'function') {
        return action.then(next);
      }
      return next(action);
    };
  }
};
```

Again, rather than take the next middleware from the store, we will make it injectable as an argument so that the function that calls the middlewares can choose which middleware to pass.

Finally, since `store` is not the only injected argument, we also need to inject the next middleware, which is the previous value of `store.dispatch`.

#### Updating `wrapDispatchWithMiddlewares`
```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

Now that middlewares are a first-class concept, we can rename `addLoggingToDispatch` to just `logger`, and rename `addPromiseSupportToDispatch` to `promise`.


### Arrow-ifying our `promise` Middleware

The curried style of function declaration can get very hard to read. Luckily we can use arrow functions and rely on the fact that they can have expressions as their bodies.

```javascript
// Before
const addPromiseSupportToDispatch = (store) => {
  return (next) => {
    return (action) => {
      if (typeof action.then === 'function') {
        return action.then(next);
      }
      return next(action);
    };
  }
};

// After
const promise = (store) => (next) => (action) => {
  if (typeof action.then === 'function') {
    return action.then(next);
  }
  return next(action);
}
```

It is still a function that returns a function returning a function, but it's much easier to read.

The mental model you can use for this is "_this is just a function with several arguments that are applied as they become available_".

### Wrapping up Middlewares

Our middlewares are currently specified in the order in which the `dispatch` function is overridden, but it would be more natural to specify the order in which the action propagates through the middlewares.

We will change our middleware declaration to specify them in the order in which the action travels through them:

```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];
  .
  .
  .
```

We will also `wrapDispatchWithMiddlewares` from right to left by cloning the past array then reversing it.

```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

[Recap at 5:42 in video](https://egghead.io/lessons/javascript-redux-the-middleware-chain)
