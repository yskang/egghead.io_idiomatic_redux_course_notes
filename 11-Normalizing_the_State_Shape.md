# 11. State shape 정규화
[비디오 링크](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)

We currently represent the `todos` in the state tree as an array of `todo` objects. However, in the real app we would probably have more than a single array, and `todos` with the same `id`s in different arrays might get out of sync.
현재 `todo` 객체들의 배열로서 `totos` 상태 트리가 표현되고 있다. 하지만, 실제 어플리케이션에서는 단순한 배열보다는 많을 것이고, 서로 다른 배열에 있는 같은 `id`를 가진 `todo`는 동기화 되지 않을 것 이다.

#### `todos.js` 이전
```javascript
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action),
      ];
    case 'TOGGLE_TODO':
      return state.map(t =>
        todo(t, action)
      );
    default:
      return state;
  }
};
```

### `todos.js` 리펙토링

상태를 데이터베이스 처럼 다루기 때문에, `id`에 의해 색인된 배열안에 `todos`를 보관하게 된다.

리듀서를 `byID`로 이름을 바꾸는 것으로 시작한다. 이제, 모든 아이템에 맵핑 하거나 끝에 새로운 아이템을 추가 하는 대신, lookup table에 값을 변경 한다.

이제 `TOGGLE_TODO`와 `ADD_TODO`는 같은 로직을 가지고 있다. `action.id` 값을 가진 새로운 lookup table은 이전에 `action.id` 값의 리듀서를 호출한 결과이며 `action`이 된다.

```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      };
    default:
      return state;
  }
};
```

여전히 리듀서의 결합이지만 배열 대신 객체로 되어 있다.

---
**노트**:


객체 전개 연산자 (`...state`)를 사용하고 있다. 이것은 ES6의 일부가 아어서, `transform-object-rest-spread` Babel plugin을 설치 해야 하며, `.babelrc` 파일에 추가해야 동작 한다.


---

`byId` 리듀서가 액션을 받으면, 현재 액션에 대해 업데이트 된`todo`를 사용하여`id`와 실제 todos 사이의 매핑 복사본을 반환 한다.

이제 모든 추가된 `id`들을 추적을 하는 리듀서를 또 추가 한다.

### `allIds` 리듀서 추가

이제 todos를`byId` 맵에 보관하고, 리듀서의 상태를 `id` 배열로 만들 것 이다.

이 리듀서는 액션의 종류에 따라 변경 될 것이고, 만일 새로운 todo가 추가 된다면, 마지막 아이템으로 새로운 `id`를 가지는 `id`들의 새로운 배열을 반환하기를 원하기 때문에 `'ADD_TODO'`만이 신경 써야 하는 액션이다.

다른 액션들에 대해서는, 현재 상태를 반환하기만 하면 된다(현재 `id`들의 배열).

```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default:
      return state;
  }
};
```

여전히 `todos.js`파일에서 한개의 리듀서를 export 할 필요가 있기 때문에, `byId`와 `allIds`리듀서들을 묶기 위해 `combineReducers()`를 다시 사용한다.

```javascript
const todos = combineReducers({
  byId,
  allIds,
});
```

---

_노트_: 리듀서를 묶는 것은 원하는 만큼 할 수 있다. 최고 레벨 리듀서만 사용할 필요는 없다. 사실, 앱이 커짐에 따라 `combineReducers`를 여러 곳에 사용하게 되는 것이 일반적이다.

---

### `getVisibleTodos` Selector 수정

이제 리듀서들에 있는 state 모양이 변경 되었고, 매우 연관있는 selectors 역시 변경할 필요가 있다.

결함된 리듀서의 `state`와 일치 하기 때문에, 이제 `getVisibleTodos`에 있는`state` 객체가 `byId`와 `allIds`를 포함하게 될 것이다.

todos의 객체를 더이상 사용하지 않기 때문에, 배열을 생성하기 위해 `getAllTodos` selector 작성 한다.

`getAllTodos`는 현재 `state`를 가지고 `allIds`를 `state`의 `byId` lookup table에 매핑함으로서 모든 `todos`를 반환 할 것 이다.

현재 파일에서만 사용되고 있기 때문에 `getAllTodos`는 export 하지 않는다.

```javascript
const getAllTodos = (state) =>
  state.allIds.map(id => state.byId[id]);
```

필터링 될 수 있는 todos의 배열을 얻기 위해 `getVisibleTodo` selector 에 있는 이 새로운 selector를 사용할 것 이다.

`allTodos` 는 컴포넌트 처럼 todos의 배열일 것 이므로, selector로 부터 반환 할 수 있고 컴포넌트 코드의 변경에 걱정할 필요가 없다.

```javascript
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

### `todo` 리듀서 추출


`todos.js` 파일은 살짝 커졌기 때문에, 분리된 파일로 한개의 todo를 관리하는 todo 리듀서를 추출하기 적절한 시점 이다.

`src/reducers` 폴더안에 `todo.js`파일을 생성하고, 구현을 복사해 넣는다. 이제 `todos.js` 파일에 import 한다.

#### `todo.js`
```javascript
const todo = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false,
      };
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {
        return state;
      }
      return {
        ...state,
        completed: !state.completed,
      };
    default:
      return state;
  }
};

export default todo;
```

#### `todos.js`
```javascript
import { combineReducers } from 'redux';
import todo from './todo';

const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      };
    default:
      return state;
  }
};

const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default:
      return state;
  }
};

const todos = combineReducers({
  byId,
  allIds,
});

export default todos;

const getAllTodos = (state) =>
  state.allIds.map(id => state.byId[id]);

export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

[Recap at 3:54 in video](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)
