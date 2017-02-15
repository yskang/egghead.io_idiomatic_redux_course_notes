# 19. 가져온 데이터로 상태 갱신 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data#/tab-transcript)


`todos.js`내부에서 `getVisibleTodos`의 현재 구현에서, 메모리에 모든 todos를 저장한다. 모든 `id`의 배열을 가지고 있고 React Router로 부터 전달된 필터에 따라 필터링 할 수 있는 todos의 배열을 얻는다.

#### 현재 `getVisibleTodos`
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

하지만, 이것은 모든 서버로 부터온 모든 데이터가 클라이언트에 이미 사용가능 해야만 정상적으로 동작 하며, 무언가 가져오는 어플리케이션에서 주로 사용되는 방식은 아니다. 만일 서버에 몇천개의 todos가 있는 경우, 클라이언트에 모두 가져와서 필터링 하는 것은 실용적이지 않을 것 이다.

### `getVisibleTodos` 리팩토링

`id`들의 커다란 리스트를 한개 유지 하는 대신, 모든 필터의 탭별로 `id`들의 리스트를 나눠져서 저장되서 유지될 수 있게 하고 가져온 데이터의 액션에 따라 채워지게 할 것 이다..

`allTodos`에 접근할 필요가 없기 때문에 `getAllTodos` 선택자를 제거 할 것 이다. 또한 서버에 의해 제공되는 todo들의 리스트를 사용하기 때문에 클라이언트에 있는 필터를 더이상 필요치 않는다. 이 말은 현재 구현에서 `switch` 문을 제거 할 수 있다는 뜻 이다.

`state.allIds`로 부터 읽는 대신, `state.IdsByFilter[filter]`로 부터 ID들을 읽을 것 이다. 그러고나서 진짜 todo들을 얻어 오기 위해 `id`들을 `state.ById` lookup 테이블에 매핑 시킬 것 이다.

#### 수정된 `getVisibleTodos`
```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = state.idsByFilter[filter];
  return ids.map(id => state.byId[id]);
};
```

### `todos` 리팩토링

선택자는 `todos` 리듀서의 결합된 `state`의 일부분이 되기 위해 이제 `idsByFilter`와 `byId`를 기대 한다.

#### `todos` 리듀서 이전:
```javascript
const todos = combineReducers({
  byId,
  allIds
});
```

The `todos` reducer used to combine the lookup table and a list of `allIds`. Now, though, we'll replace the lis of `allIds` with the list of `idsByFilter`, which will be a new combined reducer.

#### `todos` 리듀서 이후:
```javascript
const todos = combineReducers({
  byId,
  idsByFilter
});
```

### `idsByFilter` 생성하기

`idsByFilter`는 모든 필터들을 위한 `id`들의 분리된 리스트를 합친다. 그래서, `all`필터를 위해서는 `allIds`, `active` 필터를 위해서는 `activeIds`, `completed` 필터를 위해서는 `completedIDs`가 된다.

```javascript
const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
});
```

### `allIds` 리듀서 수정

원래 `allIDs` 리듀서는 ID들의 배열과 `ADD_TODO`액션을 관리 했다. 이제 이런 책임을 없앨 것 인데, 왜냐하면 이제 서버로 부터 가져오는 데이터에 응답하도록 알려주기를 원하기 때문이다.

#### `allIds` 이전:
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

`ADD_TODO`를 `RECEIVE_TODOS`로 이름을 변경하는 것부터 시작 할 것이다. `RECEIVE_TODOS` 액션을 다루기 때문에, 서버 응답으로 부터 가져오는 todos의 새로운 배열을 반환하기를 원하다. 단지 `todo`로 부터 `id`를 선택 하는 함수로 이 새로운 todo들의 배열을 매핑 할 것 이다. 모든 ID들, active ID들, completed ID들을 분리해서 저장하기로 정한 것을 상기 해서, 완전히 독립적으로 가뎌오게 할 것 이다.

#### `allIds` 이후:
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

#### `activeIds` 리듀서 생성하기

또한 `activeIds` 리듀서는 `id`들의 배열을 계속 추적할 것 이지만, 이것은 오직 active 탭에 있는 `todos`를 위해서 이다. 이전에 `allIds` 리듀서와 정확히 같은 방법으로 `RECEIVE_TODOS`를 다루기를 원할 것 이다.

```javascript
const activeIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

### 정확한 배열 수정 하기

`RECEIVE_TODOS` 액션이 오면, `activeIds` 와 `allIds` 둘 다 새로운 `state`를 반환 해야 하지만, 업데이트 해야 하는 `id` 배열을 알려줄 방법이 필요하다.

만일 `RECEIVE_TODOS` 액션을 상기 한다면, 액션의 일부로서 `filter`를 전달 했다는 것을 기억 할 지도 모르겠다. 이것은 리듀서에 상응하는 `filter`와 액션 내부의 `fitler`를 비교 하게 해준다.

모든 `allIds` 리듀셔는 단지 `all` 필터를 가진 액션에만 관심이 있고, `activeIds`는 오직 `active` 필터에만 관심이 있다.

#### `activeIds` 리듀서
```javascript
const activeIds = (state = [], action) => {
  if (action.filter !== 'active') {
    return state;
  }
  // rest of code as above
```
_`allIds`를 위해 반복 하지만 `active`를 `all`로 대체 하는 것을 기억 하라_

### `completedIds` 리듀서 생성하기
이 리듀서는 다른 필터 리듀서들과 동일 하지만, `compltet` 필터에만 해당 된다.

```javascript
const completedIds = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state;
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

### `byId` 리듀서 업데이트 하기

이제 `id`들을 고나리하는 리듀서들을 가지고 있는데, 응답으로 부터 새로운 `todos`를 진짜로 다루기 위해 `byId`리듀서를 업데이트 해야 한다.

#### `byId` 이전:
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

존재하는 `cose`를 제거 하는 것으로 시작할 수 있는데 이유는 더이상 데이터가 지역적으로 존재하지 않기 때문이다. 대신, 다른 리듀서에서만 `RECEIVE_TODOS` 액선을 다룰 것 이다.

그리고 나서 `nextState`를 생성 할 것인데, lookup 테이블에 상응하는 `state` 객체의 얕은 복사본이다. `response`에 있는 모든 `todo`객체를 거쳐 반복하고 `nextState`에 집어 넣기를 원한다.

`nextState`의 `todo.id` 항목을 방금 가져온 새로운 `todo`로 변경 할 것이다.

마지막으로, 리듀서로 부터 lookup 테이블의 다음 버젼을 반환 할 것이다.

#### `byId` After:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS': // eslint-disable-line no-case-declarations
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```

**노트:** 보통은 할당 동작은 mutation 이다. 하지만, 이 경우에는 괜찮은데 이유는 `nextState` 는 얕은 복시이기 때문이며, 한 단계 깊에 할할 뿐이다. 함수는 pure 하게 남아야 하는데 이유는 어떤 원래 상태 객체도 변경 하지 않기 때문이다.

### 마무리 하기

마지막 단계로, 프로젝트에서 `todo.js`를 import 한 부분과 파일 자체도 제거 할 수 있는데, 이는 todo들을 추가 하고 토글하는 로직이 추후 강좌에서 서버에 요청하는 API로 구현될 것이기 때문이다.

[Recap at 5:22 in video](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data)
