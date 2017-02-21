# 23. Thunks로 경합 조건 피하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)

가짜 API의 지연을 5초로 늘렸을 때, 문제가 있음을 알게 될 것 이다. 요청을 시작하기 이전에 로딩이 되는지 체크 하지 않고, 한 무리의 `receiveTodos` 액션들이 돌아와서, 경함 조건이 될 가능성이 있게 된다.

이 문제를 고치기 위해, 주어진 필터에 대해 todo들을 이미 가져 왔다는 것을 알면 `fetchTodos` 액션 생성자로 부터 일찍 빠져 나갈 수 있다.

`fetchTodos`의 안에, 인자로 저장소 상태와 필터를 받는 `getIsFetching` 선택자를 이용해서 현재 가져오는 중인지 알아보기 위해 `if`구문을 추가 할 것 이다. 만일 `true`를 반환하면, 어떤 액션도 발행 하지 않고 thunk로 부터 일찍 빠져 나올 것 이다.

#### `fetchTodos` 수정
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  if (getIsFetching(getState(), filter)) {
    return;
  }
  /// rest of fetchTodos
```

`getIsFetching` 선택자는 리듀셔 파일의 최상위 단계에 정의 되어 있어서, `reducers`로 부터 `import`되어진 것을 import 할 필요가 있다.

```javascript
import { getIsFetching } from '../reducers';
```

또한 이 파일에는 정의 되어 있지 않는 `getState()`를 이용한다. `store` 객체에 소속 되어 있지만, 액션 생성자로 부터 직접 접근 할 필요가 없다.

### `thunk` 미들웨어 수정

`configureStore.js` 안에 있는 `thunk` 미들웨어에서 `store.dispatch()` 함수 만을 thunk 액션들 안에 주입 하지 않고 `store.getState` 함수 또한 주입 해서 해결 한다. 이 방법으로, `thunk` 액션 생성자의 내부에 `dispatch` 뒤에 두번째 인자로 잡을 수 있다.

```javascript
// Inside configureStore.js
const thunk = (store) => (next) => (action) =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action);
```

```javascript
// Add `getState` as a second parameter to `fetchTodos`
export const fetchTodos = (filter) => (dispatch, getState) => {
```

이런 수정들로, `fetchTodos`액션 생성자는 조건에 따라 액션을 발행 하게 된다. 만일 앱을 실행 시키면, 동시에 새개의 요청들 이상을 생성 하지 못 할 것 이다(각각 필터 타입 마다 한개).

`isFetching` 플래그는 상응하는 `receiveTodos` 액션들이 돌아올 때만 리셋되고, 새로운 todo들을 요청 할 수 있다. 이것은 불필요한 네트웍 동작을 피하고 있을 지 모르는 경함 상황을 피할 수 있는 좋은 방법이다.

### `fetchTodos` 수정

thunk의 반환값은 Promise 이므로, 앞의 `return`은 즉시 응답하는 Promise가 되게 변경 할 것 이다. 이것을 할 필요는 없지만, 코드를 호출 하는데 편하다.

#### `actions/index.js` 내부
```javascript
export const fetchTodos = (filter) => (dispatch, getState) => {
  if (getIsFetching(getState(), filter)) {
    return Promise.resolve();
  }
  // rest of fetchTodos
```

`thunk` 미들웨어 자체는 이 Promise를 이용하지 않지만, 이 액션 생성자를 발행하는 것의 반환값 `return`이 되서, 비동기적 액션이 완료된 이후 코드들의 동작의 순서를 정하기 위해 컴포넌트 내에서 이용 할 수 있게 된다.

#### `VisibleTodoList` 내부
```javascript
fetchData() {
  const { filter, fetchTodos } = this.props;
  fetchTodos(filter).then(() => console.log('done!'));
}
```

### `redux-thunk` 소개

`redux-thunk` 는 방금 구현한 것과 비슷한 미들웨어다. 설치 하려면 다음을 실행 하라.

`npm install --save redux-thunk`.

`redux-thunk`가 설치되면, 방금 작성한 thunk 미드루에어를 제거하고, `redux-thunk`를 `thunk`대신 import 할 수 있다.

[Recap at 2:52 in video](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)
