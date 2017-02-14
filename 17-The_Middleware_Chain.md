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

저장소를 확장하는 이 메소드가 동작하는 동안, 공용 API를 오버라이드 하고 사용자 정의 함수들로 교체 하는 것은 그렇게 좋은 것이 아니다.

이 패턴을 없애기 위해, 우리가 작성한 추가-기능을 가진 함수들을 위한 멋진 이름인 _미들웨어 함수들_의 배열을 선언 할 것 이다. 

이 `미들웨어들` 배열은 추후에 단일 단계로 적용되는 함수들을 포함할 것 이다.

`addLoggingToDispatch` 와 `addPromiseSupportToDispatch`를 미들웨어 배열에 넣을 것 이다.

이제 첫번째 인자로 `store`를 받고 두번째로 미들웨어의 배열을 받는 `wrapDispatchWithMiddlewares()` 함수를 만든다.

#### `configureStore` 리팩토링
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

`wrapDispatchWithMiddlewares()` 안에서 모든 미들웨어를 위한 어떤 코드를 실행 하기 위해 `middlewares`의 `forEach` 메소드를 사용할 것 이다.

특히, `store.dispatch` 함수를 인자로서 `store`를 가지고 미들웨어를 호출하는 것의 결과를 가르키도록 오버라이드 할 것 이다.

```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store);
  );
```

미들웨어 함수들 자체의 안을 상기해 보면, 반복하는 어떤 패턴이 있다. `store.dispatch`의 값을 가지고 후에 호출하는 `next`라 불리는 변수안에 저장 한다.

이것을 미들웨어 규칙의 일부로 만들기 위해, `next`를 바깥 인자로 만들 수 있는데, 이는 이전의 `store`와 이후의 `action`과 같다.

#### `addLoggingToDispatch()` 수정
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

이 변화로, 미들웨어는 함수를 반환하는 함수를 반환하는 함수가 된다.

이 패턴은 **커링**이라 불린다. 이것은 자바 스크립트에서 매우 일반적이지 않지만, 함수형 프로그래밍 언어에서는 실제로 매우 일반적 이다.

#### `addPromiseSupportToDispatch` 수정:
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

다시, 저장소로 부터 다음 미들웨어를 가져오는 것 보다, 인자로서 주입가능하게 만들어서 미들웨어를 호출한 함수가 어떤 미들웨어를 전달할지 결정할수 있게 할 것 이다.

마지막으로, `store`는 유일한 주입된 인자가 아니므로, 다음 미들웨어를 주입할 필요가 있는데, `store.dispatch`의 이전값 이다.

#### `wrapDispatchWithMiddlewares` 수정
```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

이제 미들웨어들은 first-class 개념 이므로, `addLoggingToDispatch`를 단지 `logger`로, 그리고 `addPromiseSupportToDispatch`를 `promise`로 이름을 바꿀 수 있다.

### `promise` 미들웨어를 화살표 표기법으로 하기

커리 스타일의 함수 선언은 읽기가 매우 어려울 수 있다. 다행히도 화살표 표기법을 사용할 수 있어서 함수 본체 자체가 표현식을 가질 수 있다는 사실을 이용할 수 있다.

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

여전히 함수를 반환하는 함수를 반환하는 함수지만, 읽기는 많이 쉬워 졌다.

이를 위해 사용할 수 있는 정신적 모델은 "_이것은 단지 사용가능하게 될때 적용되는 몇개의 인자를 가지는 함수 이다._"

### 미들웨어들 정리 하기

미들웨어들은 현재 `dispatch` 함수가 오버라이드 되어 있는 순서로 정해져 있지만, 액션이 미들웨어들을 지나 전달 되는 순서로 정하는 것이 좀 더 자연스러울 것 이다.

미들웨어 선언을 변경하여 액션이 어떤 것을 지나는지에 따른 순서대로 정할 것 이다.

```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];
  .
  .
  .
```

또한 `wrapDispatchWithMiddlewares` 오른쪽에서 왼쪽으로 과거 배열을 복제하고 뒤집을 것 이다.


```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

[Recap at 5:42 in video](https://egghead.io/lessons/javascript-redux-the-middleware-chain)
