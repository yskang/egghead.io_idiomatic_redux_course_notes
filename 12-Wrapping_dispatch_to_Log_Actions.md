# 12. Log Actions로 `dispatch()` 래핑 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)

이제 상태의 모습은 더욱 복잡해 졌으므로, `store.dispatch`함수를 `console.log()` 같은 코드를 추가한 함수로 오버라이드 하여 함수 액션에 의해 상태가 어떻게 영향을 받는지 보려고 한다.

먼저, 인자로 `store`를 받는 새로운 함수 `addLoggingToDispatch` 를 만든다. 이 함수는 Redux에서 제공하는 `dispatch`를 감싸서, `store.dispatch`로 부터 발급되는 것을 그대로 읽어 들일 것이다.

`addLoggingToDispatch()`는 기존 `store.dispatch`와 같이 액션 하나를 인자로 받는 또다른 함수를 반환 할 것이다. 크롬과 같은 일부 부라우저는 같은 제목으로 몇개의 로그문구를 그룹지을 수 있는 `console.group()`의 사용을 지원한다. 액션 종류 별로 몇개의 로그들을 그룹짓기 위해 `console.group()`에 `action.type`을 제목으로 전달할 것이다.

`store.getState()`를 불러서 액션을 발행하기 전에 이전 `state`를 로그로 남기고, 발행한 액션을 로그로 남겨서 어떤 액션이 변화를 일으키는지 보려고 한다.

원래의 `dispatch` 동작을 그대로 남기기 위해, `returnValue`를 선언 하고 액션으로 `rawDispatch()` 함수를 호출한다. 이제 코드를 사용하는 쪽을 전혀 변경하지 않고도 몇가지 추가 정보를 로그로 저장하는 동작을 추가할 수 있다.

다음 상태 역시 로그로 저장 할 것인데, 스토어는 dispatch가 호출된 이후 갱신되는 것이 보장 되어 있기 때문이다. dispatch 이후 다음 상태를 받기 위해 `store.getState()`를 이용할 수 있다.

#### `configureStore.js`
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('prev state', store.getState());
    console.log('action', action);
    const returnValue = rawDispatch(action);
    console.log('next state', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  }
}

const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(todoApp, persistedState);

  store.dispatch = addLoggingToDispatch(store);
  .
  .
  .
```


### 마무리 손질하기

`console.log()`의 스타일을 바꾸는 API(크롬에서 제공)를 이용해서 로그에 색상을 추가 할 수 있다. 이전 상태는 회색, 액션은 파랑, 다음 상태는 녹색으로 칠한다.

```javascript
console.log('%c prev state', 'color: gray', store.getState());
console.log('%c action', 'color: blue', action);
const returnValue = rawDispatch(action);
console.log('%c next state', 'color: green', store.getState());
```

만일 `console.group()` API를 브라우저에서 사용할 수 없다면, 원래의 dispatch를 그대로 반환 하면 된다.

```javascript
// At the top of `addLoggingToDispatch()` after `rawDispatch` is declared
if (!console.group) {
  return rawDispatch
}
```

배포 버전에서 모든 것을 로그로 기록하는 것은 좋은 생각이 아니기 때문에, `process.env.NODE_ENV`가 배포 모드가 아니면 로그를 남기는 기능을 동작시키고, 배포 모드이면 store를 그대로 두는 코드를 `configureStore()` 안에 추가한다. 

#### `configureStore` 내부
```javascript
const configureStore = () => {
    const persistedState = loadState();
    const store = createStore(todoApp, persistedState);

    if (process.env.NODE_ENV !== 'production') {
      store.dispatch = addLoggingToDispatch(store);
    }
    .
    .
    .
```

dispatch 전후의 액션과 상태를 로그로 저장해두면 실수를 찾아내기 매우 쉬워진다.

[Recap at 2:18 in video](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)
