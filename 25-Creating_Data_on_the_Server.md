# 25. 서버에 데이터 생성하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)

### Fake API 갱신

다음 몇개의 강의를 통해 몇가지 새로운 기능들이 fake API에 추가 될 것이다.

첫번째 새로운 fake API endpoint는 `addTodo`이다. 이 것은 네트웍 연결을 시뮬레이트 하고, 새로운 `todo` 객체를 생성한다. REST endpoint들이 일반적으로 동작 하듯이 fake database로 주어진 텍스트로된 todo를 집어 넣고, `todo`객체를 반환 한다.

#### `addTodo` Endpoint
```javascript
export const addTodo = (text) =>
  delay(500).then(() => {
    const todo = {
      id: v4(),
      text,
      completed: false,
    };
    fakeDatabase.todos.push(todo);
    return todo;
  });
```

두번째 fake API endpoint는 `toggleTodo`다. 역시 네트웍 연결을 시뮬레이트 하고, fake database에서 적절한 todo를 찾아서, `completed` 필드 값을 뒤집은 후, 역시 결과로 `todo`를 반환한다.

#### `toggleTodo` Endpoint
```javascript
export const toggleTodo = (id) =>
  delay(500).then(() => {
    const todo = fakeDatabase.todos.find(t => t.id === id);
    todo.completed = !todo.completed;
    return todo;
  });
```

### "Add Todo" 과정 갱신

이 강좌에서, `addTodo` fake API endpoint 라고 하는 add todo 버튼을 만들 것 이다.

id 생성은 이제 서버에서 이루어 지기 때문에, 액션 생성자 파일에 있는 `node-uuid`에서 가져온 `v4` 함수는 더이상 필요하지 않다.

`addTodo` 액션 생성자는 액션 생성 thunk로 변경 될 것 이기 때문에, 커링 인자로 `dispatch`를 추가 할 것 이다. thunk는 주어진 텍스트를 가지고 API의 `addTodo` endpoint를 호출하고, 응답이 오기를 기다릴 것 이다.

응답이 오면, `'ADD_TODO_SUCCESS'` 타입과 서버 응답을 포함한 액션을 발행 할 것이다.

##### 갱신된 `addTodo` 액션
```javascript
export const addTodo = (text) => (dispatch) =>
  api.addTodo(text).then(response => {
    dispatch({
      type: 'ADD_TODO_SUCCESS',
      response,
    });
  });
```

#### `byId` 리듀서 갱신하기

새롭게 추가된 todo는 서버 응답의 일부가 될 것이기 때문에, `byId` 리듀서를 관리하는 lookup 테이블에 todo를 합치도록 변경 하야 한다.

`'ADD_TODO_SUCCESS'` 액션을 위한 새로운 case를 추가 할 것이다. 객체 전개 연산자를 lookup table의 새 버젼을 생성하는데 사용 할 것이다. 액션 응답 ID 키 아래에, 액션 응답으로 부터 읽은 새로운 todo 객체가 있다.

```javascript
// Inside reducers/byId.js
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'FETCH_TODOS_SUCCESS':
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    case 'ADD_TODO_SUCCESS': // Our new case
      return {
        ...state,
        [action.response.id]: action.response,
      };
    default:
      return state;
  }
};
```

하지만, 필터에 의해 리스트가 갱신 되지 않기 때문에, 리스트에는 오직 세개의 ID들만이 있을 것 이다. 만일 다른 텝으로 간다면, 새로운 todo가 보일 것인데 이유는 ID가 이제 가져온 ID들의 리스트에 포함되어 있기 때문이고, 비슷하게, 만일 이전 텝으로 돌아 간다면, 데이터를 다시 가져왔기 때문에 이전 todo들이 보일 것 이다.

#### `createList` 갱신

각각 텝을 위한 ID들의 리스트가 `createlist.js`에 정의되어 있는 리듀서에 의해 관리되고 있기 때문에, `'ADD_TODO_SUCCESS'` 액션을 다루는 `ids` 리듀서를 갱신 해야 한다.

todo가 서버에 추가 되었다는 확인을 받았을 때, 처음부터 있었던 ID들을 가지고 만든 리스트의 제일 끝에 새로운 ID를 추가한 새로운 ID 리스트를 반환 할 수 있다.

다른 액션들과 다르게, `'ADD_TODO_SUCCESS'`는 `action` 객체에 `filter` property를 가지고 있지 않아서, 현재 `ids`안에 있는 검사하는 `if (filter !== action.filter)` 문장은 실패 할 것 이다. 이것 때문에, 기존 검사하는 로직을 각각 다른 검사 로직으로 바꿀 것 이다.

For `FETCH_TODOS_SUCCESS`, we want to replace the fetched IDs if the filter in the action matches the filter of this list.
`FETCH_TODOS_SUCCESS`의 경우, 만일 액션에 있는 필터가 리스트의 필터와 일치 한다면 가져온 IDs을 대체 할 것 이다.

하지만, `ADD_TODO_SUCCESS` 의 경우, 완료 필터를 제외하고 모든 리스트에 있는 새롭게 추가된 todo들이 나타나게 해야 하는데, 왜냐하면 새롭게 추가된 todo들은 완료 되지 않았기 때문이다.

```javascript
const createList = (filter) => {
  const ids = (state = [], action) => {
    switch (action.type) {
      case 'FETCH_TODOS_SUCCESS':
        return filter === action.filter ?
          action.response.map(todo => todo.id) :
          state;
      case 'ADD_TODO_SUCCESS':
        return filter !== 'completed' ?
          [...state, action.response.id] :
          state;
      default:
        return state;
    }
  };
```

[Demonstration and recap at 3:36 in video](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)
