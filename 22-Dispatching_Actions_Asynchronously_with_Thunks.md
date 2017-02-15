# 22. Thunks를 사용해 비동기적으로 액션 발행하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)

현재 구현에서 로딩 인디케이터를 보여주기 위해, `fetchTodos`로 todo들을 가져오기 이전에 `requestTodos` 액션을 발행 한다. 별도로 분리해서 호출 하기 원지 않으므로 todo들을 가져올 때 자동으로 `requestTodos` 발행을 한다면 매우 좋을 것 이다.

#### `VisibleTodoList.js` 이전
```javascript
// inside of `VisibleTodoList`
fetchData() {
  const { filter, fetchTodos, requestTodos } = this.props;
  requestTodos(filter);
  fetchTodos(filter);
}
```

시작으로, 컴포넌트로 부터 `requestTodos` 발행을 명시적으로 하는 것을 제거 할 것 이다. 이제 더이상 `actions/index.js`의 내부에서 `requestTodos` 액션 생성자를 `export`할 필요가 없다 (하지만, 코드는 지우지 마라).

Our goal is to dispatch `requestTodos()` when we start fetching, and `receiveTodos()` when they finish fetching, but our `fetchTodos` action creator only resolves through the `receiveTodos` action.

데이터를 가져오기 시작할때 `requestTodos()`를 발행 하고, 데이터 가져오기를 끝냈을 때 `receiveTodos()`를 발행 하는 것이 목표지만, `fetchTodos` 액션 생성자는 `receiveTodos` 액션을 통해서만 동작 한다.

#### 현재 `fetchTodos` 액션 생성자
```javascript
export const fetchTodos = (filter) =>
  api.fetchTodos(filter).then(response =>
    receiveTodos(filter, response)
  );
```

액션 Promise는 마지막에 단일 액션을 내놓지만, 일정 시간 이상 다수개의 액션이 발행되는 것을 추상화 하고 싶다. 이것은 Promise를 반환 하는 것 보다, 발행하는 컬백 인자를 받는 함수를 반환 하는 것을 선호하는 이유다. 

이 변경은 비동기 동작 동안 어떤 시간에서도 원하는 만큼 많은 횟수로 `dispatch()`를 호출 할수 있게 해준다. 시작할때 `requestTodos` 액션을 발행 하고 나서, Promise가 동작 할 때 또 다른 `receiveTodos` 액션을 명시적으로 발행 할 수 있게 해준다.

#### `fetchTodos` 수정
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  dispatch(requestTodos(filter));

  return api.fetchTodos(filter).then(response => {
    dispatch(receiveTodos(filter, response));
  });
};
```

이것은 Promise를 반환하는 것 보다 타이핑을 많이 해야 하지만, 좀 더 유연함을 준다. Promise는 하나의 비동기적 표현을 할 수 있지만, `fetchTodos`는 이제 컬백 인자를 가지는 함수를 반환 해서 비동기 동작 동안 여러번 호출 할 수 있게 되었다.

### Thunks 소개

`fetchTodos` 에 있는 다른 함수에서 반환되는 함수들은 종종 thunks라고 불린다. 이제 코드에 thunks를 이용 할 수 있도록 미들웨어를 구현 할 것 이다.

### `configureStore.js` 수정

`configureStore.js` 내분에 `redux-promise` 미들웨어를 제거 하고 지금 작성할 `thunk` 미들웨어로 교체 한다.

`thunk` 미들웨어는 thunk들의 발행을 지원 한다. 다른 미들웨어 처럼 `store`, 다음 미들웨어 (`next`), `action` 을 인자로 가진다.

만일 `action`이 액션 대신 함수라면, 이것은 `dispatch` 함수가 주입되기를 원하는 thunk라고 가정 할 것 이다. 이 경우, `store.dispatch`로 액션을 호출 할 것 이다.

그렇지 않으면, 단지 체인에 있는 다음 미들웨어로 액션을 전달하는 것의 결과를 반환 할 것 이다.

#### `configureStore.js`의 안에 있는 `thunk` 미들웨어
```javascript
const thunk = (store) => (next) => (action) =>
  typeof action === 'function' ?
    action(store.dispatch) :
    next(action);
```

마지막 단계로, 새로운 `thunk` 미들웨어를 `configureStore`의 안에 미들웨어들의 배열에 추가 해서 저장소에 저용되도록 해야 한다.

```javascript
const configureStore = () => {
  const middlewares = [thunk];
  // rest of configureStore
```

[Recap at 2:45 in video](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)
