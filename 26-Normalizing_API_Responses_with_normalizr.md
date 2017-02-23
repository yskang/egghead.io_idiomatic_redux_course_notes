# 26. `normalizer`로 API 응답 정규화 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)

현재 `byId` 리듀서는 서버 액션을 서로 다르게 다루고 있는데, 이유는 응답의 모양이 서로 다르기 때문이다.

예를 들어, `'FETCH_TODOS_SUCCESS'` 액션의 응답은 todo들의 배열이다. 이 배열은 반복해서 한번에 하나씩 다음 상태로 합쳐져야 한다.

`'ADD_TODO_SUCCESS'`를 위한 응답은 방금 추가된 한개의 todo이고, 이 todo는 서로 다른 방법으로 합쳐져야 한다.

모든 새로은 API 호출을 추가 하는 대신, 응답의 모양이 항상 같도록 응답을 정규화 하려고 한다.

### `normalizr` 설치 하기

`normalizr`는 API 응답을 같은 모양이 되도록 정겨호 ㅏ하는데 도움을 주는 유틸리티 라이브러리다.

`$ npm install --save normalizr`

###  `schema.js` 생성하기
`actions` 폴더에 새로운 파일 `schema.js`를 생성 할 것 이다.

`Schema` 생성자를 import 하고, `normalizr`로 부터 `arrayOf` 함수를 import 할 것 이다.

첫번 째로 export 되는 Schema는 `todo` 객체가 될 것이고, `todos`를 정규화된 응답에 있는 사전의 이름으로 지정 할 것 이다.

다음 schema는 `arrayOfTodos`로서 `todo` 객체의 배열을 가지고 있는 응답에 해당 될 것 이다.

```javascript
import { Schema, arrayOf } from 'normalizr'

export const todo = new Schema('todos');
export const arrayOfTodos = arrayOf(todo);
```

### Action 생성자 갱신하기

`actions/index.js` 에, `normalizr`로 부터 `normalize`라는 함수를 import 해서 추가 할 것 이다. 또한 schema 파일에 정의한 모든 Schema를 위한 namespace를 추가 한다.

`FETCH_TODOS_SUCCESS` 컬백 내부에, `normalized response` 로그를 추가 해서 정규화된 응답이 어떻게 보이는지 볼 수 있게 할 것 이다. 첫번째 인자로 원래 `response`를 넣고, 두번째 인자로 이에 따른 schema(이경우, `arrayOfTodos`)를 넣어서 `normalize` 함수를 호출한다.

```javascript
return api.fetchTodos(filter).then(
  response => {
    console.log(
      'normalized response',
      normalize(response, schema.arrayOfTodos)
    )
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
```

`addTodo`도 비슷한 방법으로 수정한다.
```javascript
export const addTodo = (text) => (dispatch) =>
  api.addTodo(text).then(response => {
    console.log(
      'normalized response',
      normalize(response, schema.todo)
    )
    dispatch({
      type: 'ADD_TODO_SUCCESS',
      responsed,
    });
  });
```

### 응답 비교하기

이 시점에서, 액션에서 응답은 to-do 객체들의 배열이지만, `'FETCH_TODOS_SUCCESS'`에 대한 정규화된 응답은 다음 두개의 필드를 가지는 객체이다: `entities`와 `result`

`entities`는 그 id에 의한 응답에 있는 모든 `todo`를 포함한 `todos`라는 정규화된 사전을 가진다. `normalizr`는 뒤따르는 `arrayOfTodos` schema에 의한 응답에서 이런 `todo` 객체들을 찾는다. 편리하게도, ID들로 인텍싱 되어 있어서, lookpu 테이블에 쉽게 합쳐질 수 있을 것 이다.

두번째 필드는 `result`인데, `todo` ID들의 배열이다. 원래 응답 배열에 있는 `todos`와 같은 순서로 되어 있다. 하지만, `normalizr`는 각각 `todo`를 그것의 ID로 대체 하고, 모든 todo를 `todos` 사전으로 이동 시킨다. 


### 액션 생성자 마무리 수정

액션 생성자를 변경해서 응답 필드에 원래 응답 대신 정규화된 응답을 전달 한다.

##### 이전:
```javascript
return api.fetchTodos(filter).then(
  response => {
    console.log(
      'normalized response',
      normalize(response, schema.arrayOfTodos)
    )
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
```

##### 이후:
```javascript
return api.fetchTodos(filter).then(
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response: normalize(response, schema.arrayOfTodos),
    });
  },
```

### 리듀서 수정하기

`byId` 리듀서에 있는 특별한 case를 삭제 할 수 있는데, 이유는 응답의 모양이 정규화 도었기 때문이다. 액션 종류에 의해 스위칭 되는 대신, 액션이 그에 대한 응답 객채를 가지고 있는지 확인 할 것 이다.

##### `byId` 리듀서 이전:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'FETCH_TODOS_SUCCESS': // eslint-disable-line no-case-declarations
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    case 'ADD_TODO_SUCCESS':
      return {
        ...state,
        [action.response.id]: action.response,
      };
    default:
      return state;
  }
};
```

정규화된 응답에 있는 `entities.todos`의 내부에 어떤 요소도 포함된, 현재 있는 모든 요소들을 가지고 있는 새로운 lookup 테이블 버젼을 반환 할 것 이다. 다른 액션에 대해선, 있는 그대로 lookup 테이블을 반환 할 것 이다.

##### `byId` 리듀서 이후:
```javascript
const byId = (state = {}, action) => {
  if (action.response) {
    return {
      ...state,
      ...action.response.entities.todos,
    };
  }
  return state;
};
```

이제 새로운 `action.response` 모습을 위해 `createList.js`의 내부에 있는 `ids` 리듀서를 업데이트 해야 한다.

##### `ids` 리듀서 이전:
```javascript
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

이제, 액션 응답은 `result` 필드를 가지며, 이 필드는 `id`들의 배열(이경우 `'FETCH_TODOS_SUCCESS'`), 혹은 가져온 todo(이 경우 `'ADD_TODO_SUCCESS'`)의 단일 `id` 이다.

##### `ids` 리듀서 이후
```javascript
const ids = (state = [], action) => {
   switch (action.type) {
     case 'FETCH_TODOS_SUCCESS':
       return filter === action.filter ?
         action.response.result :
         state;
     case 'ADD_TODO_SUCCESS':
       return filter !== 'completed' ?
         [...state, action.response.result] :
         state;
     default:
       return state;
   }
 };
```

[Demonstration and recap at 5:33 in video](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)
