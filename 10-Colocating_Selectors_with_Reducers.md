# 10. Selector와 리듀서 같은 곳에 놓기
[비디오 링크](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux)

`VisibleTodoList` 안에, `mapStateToProps` 함수는 `getVisibleTodos` 함수를 사용해서, `todos`에 해당하는 `state`의 일부를 전달한다. 하지만, 만일 state의 구조를 변경한다면, 이 모든 갱신 사항을 기억해야 한다.

이것을 명확히 하기 위해, `getVisibleTodos` 함수를 뷰 레이어 밖으로 이동시켜서 `todos` 리듀서가 포함된 파일에 넣을 수 있다. `todos`리듀서는 `todos` 상태의 내부구조에 대해 가장 잘 알고 있기 때문에 이렇게 할 것이다.

#### `VisibleTodoList` 이전
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all':
      return todos;
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};

const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all'),
});
```

### Reducer 수정

`getVisibleTodos` 구현을 리듀서와 같은 파일로 옮기고, 이름을 지어서 export 할 것 이다.

규칙은 간단하다. 기본 export는 항상 리듀서 함수이지만, UI에 의해 표시되는 데이터를 준비하는 함수는 `'get'`으로 이름을 시작한다. 현재 상태에서 어떤 것을 선택하기 때문에, 이런 함수들은 일반적으로 _selectors_ 라고 부른다.

리듀서들 안에, `state` 인자는 이 특정 리듀서의 상태와 일치하기 때문에, selectors 를 위한 같은 규칙을 따를 것이다. `state` 인자는 이 파일에서 exoprt 되는 리듀서의 상태와 일치한다.

#### Inside `src/reducers/todos.js`
```javascript
export const getVisibleTodos = (state, filter) => {
  switch (filter) {
    case 'all':
      return state;
    case 'completed':
      return state.filter(t => t.completed);
    case 'active':
      return state.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

### Root Reducer 수정

`VisibleTodoList`의 안은 여전히 state 구조에 의존적이다. 왜냐하면 state로 부터 `todos`를 읽지만 실제 `todos`를 읽어들이는 방법은 나중에 바뀔 것이기 때문이다.

이런 점을 염두해 두고, 이름 지어진 selector를 export하는 루트 리듀서를 수정할 것 이다. `getVisibleTodos`라 불릴 것이고, 이전 처럼 `state`와 `filter`를 받아들일 것 이다. 하지만, 이 경우 `state`는 결합된 리듀서의 상태와 같을 것 이다.

이제, 리듀서와 같이 `todos` 파일안에 정의된 `getVisibleTodos` 함수라고 부를 수 있지만, 영역안에 함수이름과 정확히 같은 이름으로 import할 수 있기 때문에 그 이름은 사용할 수 없다.

이를 피하기 위해, name space를 import하는 문법을 사용해서 모든 exprot를 객체와 함께 표시한다(이경우 `fromTodos`).

이제 다른 파일에 있는 정의된 함수를 호출 하기 위해 `fromTodos.getVisibleTodos()`를 사용할 수 있고, `todos`에 `state`에 해당하는 부분을 전달 할 수 있다.

#### Root Reducer 수정 (`src/reducers/index.js`)
```javascript
import { combineReducers } from 'redux';
import todos, * as fromTodos from './todos';

const todoApp = combineReducers({
  todos,
});

export default todoApp;

export const getVisibleTodos = (state, filter) =>
  fromTodos.getVisibleTodos(state.todos, filter);
```

### `VisibleTodoList` 수정
이제 `VisibleTodoList`컴포넌트ㅇ로 돌아가서 루트 리듀서 파일 로 부터 `getVisibleTodos`를 import 할 수 있다.

`import { getVisibleTodos } from '../reducers'`

`getVisibleTodos`는 어플리케이션 상태 모습에 관한 모든 지식을 포함하고 있어서, 이것을 넘김 으로서 어플리케이션의 모든 상태를 전달 할 수 있으며, 어떻게 selector에 묘사된 로직에 따라 visible todo가 선택되는지 알게 해 준다.

[Recap at 2:51 in video](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux)
