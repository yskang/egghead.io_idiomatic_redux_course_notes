# 16. Promises 를 인식 하기위해 `dispatch()` 래핑 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)

`receiveTodos` 액션 생성자는 그 자체로는 그렇게 쓸모 있는 것은 아닌데, 이유는 호출 하면, `todos`를 가져오는 것을 먼저 해야 하기 때문이다. `fetchTodos`와 `receiveTodos`는 같은 인자들을 받아 들이기 때문에, 만일 이 두개의 코드를 하나의 액션 생성자로 묶을 수 있다면 매우 좋을 것 이다.

#### `VisibleTodoList`의 안에 있는 `fetchData()`
```javascript
fetchData() {
  const { filter, receiveTodos } = this.props;
  fetchTodos(filter).then(todos =>
    receiveTodos(filter, todos)
  );
}
```

### 액션 생성자 리팩토링

액션 생성자 파일(`src/actions/index.js`)로 가짜 API를 importing 하는 것부터 시작 한다.

`import * as api from '../api'`

이제 `fetchTodos`라 불리는 비동기 액션 생성자가 추가 될 것 이다. 인자로 `filter`를 받아서, 그 인자로 API의 `fetchTodos` 메소드를 호출 할 것 이다.

Promise 의 `then` 메소드를 이용해서 promise의 결과를 `response`에서 주어진 `filter`와 `response`에 의해 생성된 액션 객체로 변환 시킬 것 이다.

#### `src/actions/index.js` 내부
```javascript
export const fetchTodos = (filter) =>
  api.fetchTodos(filter).then(response =>
    receiveTodos(filter, response)
  );
```

`receiveTodos`는 동기적으로 액션 객체를 반환하지만, `fetchTodos`는 액션 객체를 통해 얻어지는 Promise를 반환 한다.

이제 액션 생성자들로 부터 `receiveTodos`를 내보내는 것을 중단 할 수 있는데, 이유는 `fetchTodos`를 직접 사용 하도록 컴포넌트를 변경 시킬 수 있기 때문이다.

### `VisibleTodoList` 갱신

컴포넌트 파일로 돌아와서, `connect`에 의해 주입된 `fetchTodos` prop을 사용할 수 있다. 이것은 방금 작성한 새로운 동기적 `fetchTodos` 액션 생성자에 해당된다.

`import { fetchTodos } from '../api'`를 없앨 수 있는데, 이유는 이제부터 `connect`에 의해 props에 주입된 `fetchTodos`를 이용할 것이기 때문이다.

```javascript
fetchData() {
  const { filter, fetchTodos } = this.props;
  fetchTodos(filter);
}
```

### 방금 한 것을 다시 반복...

`fetchTodos` 액션 생성자는 API로 부터 `fetchTodos` 함수를 호출하지만, 그 후 그 결과를 `receiveTodos`에 의해 생성된 Redux 액션 생성자로 변환 한다.

하지만, 기본적으로, Redux는 Promise 보다는 평범한 객체를 발행하는 것 만을 허락 한다. `configureStore.js` 안에 있는 `addLoggingToDispatch()`에서 사용되는 것과 같은 트릭을 사용하는 것으로 Promise를 알아채는 것을 알려 줄 수 있다. (`addLoggingToDispatch` 함수가 `store`에서 `dispatch`를 가져와서 모든 액션과 `state`를 기록하는 새로운 버전의 `dispatch`를 반환하는 것을 상기하라.)

### Promise 지원 추가

`configureStore.js` 안에서, `store`를 받아들이고 promise를 지원하는 `dispatch` 버전을 반환하는 `addPromiseSupport()` 함수를 생성할 것이다.

우선, `store`에 정의도어 있는 `rawDispatch` 함수를 잡아서 차후에 호출 하도록 한다. dispatch 함수로서 같은 API를 가지는 함수를 반환 할 것이다 -- 이 말은, 액션을 받는 다는 말이다.

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

`action`이 진짜 액션인지 액션의 promise인지 모르기 때문에, `then` 메소드가 있는지 확인해 봐서 있으면 함수이다. 만일 액션이 promise라면, `rawDispatch`를 통과해서 액션을 가져오기를 기다린다.

그렇지 않으면, 받은 `action`객체를 이용해 `rawDispatch`를 바로 호출하기 만을 할 것 이다.

새로운 `addPromiseSupportToDispatch` 함수는 액션과 액션을 내놓는 promise 둘 다 사용할 수 있게 해 준다.

마무리 하기 위해, 저장소로 돌아가기 전에 앱으로 가기 위해 한번 더 새로운 함수를 호출할 필요가 있다.

```javascript
// At the bottom of `configureStore.js`
const configureStore = () => {
  const store = createStore(todoApp);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.dispatch = addPromiseSupportToDispatch(store);

  return store;
};
```

만일 앱을 지금 실행 시키면, 응답이 준비되었을 때 여전히 `'RECEIVE_TODOS'`액션이 발행 되는 것을 볼 수 있을 것 이다. 하지만, 컴포넌트는 비동기적인 액션 생성자에서 비동기적인 로직을 캡슐화 하는 좀 더 편한 API를 사용한다.

### 순서 문제

`configureStore` 안에 있는 `dispatch` 함수를 오버라이드 하는 순서가 중요하다는 것을 기억해야 하다.

만일 변경해서 `addLoggingToDispatch` 이전에 `addPromiseSupportToDispatch`를 호출하면, 액션인 먼저 표시되고나서 promise가 처리 될 것 이다.

이렇게 되면 `undefined` 형식의 액션을 받게 되고 액션 대신 별로 유용하지 않은 Promise를 보게 될 것 이다.

[Recap at 4:15 in video](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)
