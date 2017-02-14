# 12. Log Actions로 `dispatch()` 래핑 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)

이제 상태의 모습은 더욱 복잡해 졌으므로, `store.dispatch`함수를 `console.log()`문구를 추가해서 오버라이드 하여 함수 액션에 의해 상태가 어떻게 영향을 받는지 보려고 한다.

인자로 `store`를 받는 새로운 함수 `addLoggingToDispatch` 를 만드는 것 부터 시작한다. 이 함수는 Redux에서 제공하는 `dispatch`를 감싸서, `store.dispatch`로 부터 발급되는 것을 그대로 읽어 들일 것이다.

`addLoggingToDispatch()`는 단일 액션 인자로서 같은 표식을 가지는 또다른 함수를 반환 할 것이다. 크롬과 같은 일부 부라우저는 같은 제목하에 몇개의 로그문구를 그룹지을 수 있는 `console.group()`의 사용을 지원하고, 액션 종류 별로 몇개의 로그들을 그룹짓기 위해 `action.type`을 전달 할 것 이다.

`store.getState()`를 불러서 액션을 발행하기 이전에 이전 `state`를 로그로 남길 것 이다. 그 다음, 액선 그 자체를 로그로 남겨서 어떤 액션이 변화를 일으키는지 보려고 한다.

`dispatch`함수의 동작을 정확히 지키기 위해, `returnValue`를 선언 하고 액션으로 `rawDispatch()` 함수를 호출한다. 몇가지 정보를 추가로 로그로 저장 하는 것을 제외하면, 이제 코드를 호출 하는 것으로 지금 만든 함수와 Redux에서 제공하는 함수 사이의 구분을 할 수 없을 것이다.

다음 상태 역시 로그로 저장 할 것인데, 스토어는 dispatch가 호출된 이후 갱신되는 것이 보장 되어 있기 때문이다. dispatch이후 다음 상태를 받기 위해 `store.getState()`를 이용할 수 있다.

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

크롬은 `console.log()`의 스타일을 위한 API를 제공하기 때문에, 로그에 색상을 추가 할 수 있다. 이전 상태는 회색, 액션은 파랑, 다음 상태는 녹색으로 색칠한다.

```javascript
console.log('%c prev state', 'color: gray', store.getState());
console.log('%c action', 'color: blue', action);
const returnValue = rawDispatch(action);
console.log('%c next state', 'color: green', store.getState());
```

만일 `console.group()` API가 브라우저에서 사용할 수 없다면, 발행된 그대로 반환 하면 된다.

```javascript
// At the top of `addLoggingToDispatch()` after `rawDispatch` is declared
if (!console.group) {
  return rawDispatch
}
```

생산 버전에서 모든것을 로그로 기록하는 것은 좋은 생각이 아니기 때문에, `process.env.NODE_ENV`가 생산모드가 아니면 이 코드를 동작 하도록 알려주는 게이트를 `configureStore()` 안에 추가 할 것이다. 그렇지 않으면, 저장소를 있는 그대로 남겨둘 것이다.

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

발행 이전 이후에 액션을 로그로 저장하는 것은 실수를 찾아내기 매우 쉽게 해 줄 것이다.

[Recap at 2:18 in video](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)
