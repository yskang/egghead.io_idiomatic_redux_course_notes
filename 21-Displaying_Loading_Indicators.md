# 21. 로딩 인디케이터 표시하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

데이터를 비동기적으로 가져올 때, 사용자에게 어떤 종류의 시각적 인지를 표시해 주고 싶다. `render` 함수에 데이터를 가져오는지 알려주는 조건을 추가 해서, 보여줄 todo가 없다면, `VisibleToDoList`의 `render` 함수로 부터 로딩 인디케이터를 반환 할 것 이다.


`props`로 부터 `todos` 와 `isFetching`을 얻을 것 이다. `todos`는 리스트에 전달하기를 원하는 유일한 추가 prop 이므로, 전개 연산자를 사용하는 대신, 단지 `todos`를 직접 전달 할 것 이다.

`mapStateToProps` 함수는 이미 `visibleTodos` 를 계산 하고, prop에 `todos`를 포함 한다. `isFetching`과 비슷한 무언가를 해야 한다. `getIsFetching`은 앱의 현재 `state`와, 가져온 `todos`를 위한 `filter`를 받는다. 리듀서 파일의 최고 단계에 있는 최고 단계의 다른 선택자에 따라 선언 된다.

#### `VisibleTodoList.js` 내부
```javascript
render() {
    const { isFetching, toggleTodo, todos } = this.props;
    if (isFetching && !todos.length) {
      return <p>Loading...</p>;
    }

    return (
      <TodoList
        todos={todos}
        onTodoClick={toggleTodo}
      />
    );
  }

  /// PropType Declarations...

  const mapStateToProps = (state, { params }) => {
    const filter = params.filter || 'all';
    return {
      isFetching: getIsFetching(state, filter),
      todos: getVisibleTodos(state, filter),
      filter,
    };
  };
```

### 루트 리듀서 수정

루트 리듀서 파일('src/reducers/index.js')로 돌아가서 export된 `getIsFetching`을 위한 또다른 선택자 함수를 추가 할 것이다. 그 함수는 `state`와 `filter`를 인자로 받고, 리스트가 현재 가져온 상태인지 알아보기 위해 다른 선택자에게 위임한다.

`state.listByFilter`로 부터 이 리스트의 상태로 전달할 것이지만, `getIsFetching`을 아직 작성하지 않았다.

```javascript
export const getIsFetching = (state, filter) =>
  fromList.getIsFetching(state.listByFilter[filter]);
```

### `createList.js` 수정

새로운 `getIsFetching` 선택자를 생성하기 이전에, 리스트의 상태 모댱을 수정할 필요가 있다. `state`가 `id`들의 배열 이라고 가정하기 보다, property로서 이 배열을 가지고 있는 객체라고 가정할 것 이다.

이제 `state.isFetching`을 읽는 `getIsFetching`이라고 불리는 또 다른 선택자를 추가 할 수 있다.

```javascript
export const getIds = (state) => state.ids;
export const getIsFetching = state => state.isFetching;
```

이 두 필두 모두를 추적하는 리듀서를 원하기 때문에, 이미 존재하는 `createList` 리듀서를 복잡하게 하는 대신, `ids`로 이름을 비꿀 것 인데, 이유는 `id`만을 관리하기 때문이다.

### `isFetching` 만들기

첫 번째로, 파일의 제일 위에 Redux로 부터 유틸리티 `combineRreducers`를 import 해서 추가 할 필요가 있다:

```javascript
import { combineReducers } from 'redux';
```
이제 `state`의 `isFetching` 플래그만 관리하는 `isFetching` 리듀서를 생성 할 수 있다.

이 리듀서의 초기 상태는 `false`이고, 마치 다른 리듀서 처럼 보인다. `action.type`으로 switch문을 작성해서, 만일 `REQUEST_TODOS`라면, 데이터를 가져오기 시작했으므로 `true`를 반환하게 할 것 이다.

만일 `'RECEIVE_TODOS'` 라면, 동작이 끝났기 때문에 `false`를 반환 할 것이다. 어떤 액선에 대해서도, 현재 `state`를 반환 할 것 이다.

`createList`의 인자인 `filter` 인자가 맞지 않는 필터를 가지고 어떤 액션이라도 무시하는 `ids` 리듀서와 같은 조건을 추가 할 것 이다.

#### Inside `createList.js`
```javascript
const isFetching = (state = false, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'REQUEST_TODOS':
        return true;
      case 'RECEIVE_TODOS':
        return false;
      default:
        return state;
    }
  };
```

`REQUEST_TODOS` 액선을 다루지만, 어디에도 발행하지는 않는다는 것을 알아 두기 바란다.

### 'REQUEST_TODOS' 액션 생성자 추가 하기

액션 생성자 파일(`src/actions/index.js`)에서, 해당하는 필터와 같이 동작 하는 `'RECEIVE_TODOS'`라고 불리는 새로운 export된 함수를 추가 할 것 이다.

```javascript
export const requestTodos = (filter) => ({
  type: 'REQUEST_TODOS',
  filter,
});
```

모든 export된 액션 생성자는 `VisibleToDoList` 컴포넌트의 `props`에서 사용 가능하게 될 것 이다.

### `VisibleToDoList` 내부 `fetchData` 수정
`props`로 부터 `requestTodos`를 재구조화 할 수 있고, 비동기 적인 `fetchToDos` 동작을 하기 직전에 부를 수 있다.

```javascript
fetchData() {
  const { filter, fetchTodos, requestTodos } = this.props;
  requestTodos(filter);
  fetchTodos(filter);
}
```

[Recap at 3:51 in video](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)
