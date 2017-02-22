# 03. 내부 저장소에 상태 저장 해 놓기
[비디오 링크](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage)

이번 예제에서는 브라우저의 `localStorage` API를 이용해 어플리케이션의 상태를 지속시킬 수 있게 해볼 수 있다.

함수 `loadState()`와 모델 `localStorage.js`을 작성한다.

`loadState` 함수는 키를 이용해 `localStorage`에 들어가서, 문자열을 검색하고, JSON 포멧으로 파싱할 것이다. 이 코드는 사용자의 브라우저가 `localStorage` API의 사용을 불허 할 수 있기 때문에 `try/catch` 로 감쌀 필요가 있다.

####`localStorage.js`
```javascript
export const loadState = () => {
  try {
    const serializedState = localStorage.getItem('state');
    if (serializedState === null) {
      return undefined;
    }
    return JSON.parse(serializedState);
  } catch (err) {
    return undefined;
  }
};
```

`loadState` 함수에서 만일 `serializedState` 가 `null` 이라면 키가 존재하지 않는 것을 의미하기 때문에, 리듀서들이 상태값 대신 설정 하도록 `undefined`를 반환 해야 한다.

하지만, 만일 `serializedState` 문자열이 존재하면, `state` 객체로 반환해야 하기 때문에 `JSON.parse(serializedState)`를 사용한다.

상태를 읽기위한 함수를 가지고 있으니까, localStorage에 상태를 하나 저장해 보자:

```javascript
export const saveState = (state) => {
  try {
    const serializedState = JSON.stringify(state);
    localStorage.setItem('state', serializedState);
  } catch (err) {
    // Ignore write errors.
  }
};
```

`saveState`함수는 매개변수로서 `state`를 받고, `loadState`함수는 정반대로 동작 한다.

우선, `JSON.stringify(state)`를 사용해 상태를 문자열로 직렬화 시킨다. 상태가 직렬화 되어 있는 경우에만 동작하게 되는데, 만일 권장되는 리덕스 사용법을 따른다면 잘 동작할 것 이다.

`JSON.stringify()` 혹은 `localStorage.setItem()`은 실패 할 수 있는 동작이므로, 에러를 catch해야 한다.


### `localStorage.js` 사용
`index.js` 파일로 돌아와서, 방금 작성한 함수들을 import 한다.
```javascript
import { loadState, saveState } from './localStorage'
```

저장소가 변경될 때 마다 상태를 저장하기 위해서, 상태의 변화를 감지해서 `saveState` 함수로 저장소의 현재 상태를 전달하는 동작을 하는 리스너를 추가하는데 `store`의 `subscribe()` 메소드를 사용할 것 이다.

```javascript
// Inside of index.js ...
store.subscribe(() => {
  saveState(store.getState())
})
```

이제 다시 로딩 될때마다 상태가 저장될 것 이다. 하지만, 이건 완료 혹은 미완료 Todo 아이템만 추적되는 것이 아니다. 대부분의 경우 데이터는 저장하기를 원하지만 UI까지 원하지는 않기 때문에 이렇게 하는 것은 이상적이지 않다.

이런 점을 수정하기 위해, `store.subscribe()`를 고칠 것이다.:
```javascript
store.subscribe(() => {
  saveState({
    todos: store.getState().todos
  })
})
```
이제 상태에서 `todos` 부분만 저장 한다. 페이지를 새로고칠 때, `visibilityFilter`는 리듀서에 의해 기본값인 `'SHOW_ALL'` 이 설정 될 것이다.

### 하지만 버그가 있다...
이 코드에 작성된 방법으로 새로운 Todo를 추가 하려고 할 때, React는 `Encountered two children with the same key, 0` 라는 에러를 낼 것이다.

이것은 `actions.js` 파일 안에있는 `addTodo` 액션 생성자에 정의되어 있는 `TodoList` 컴포넌트에 키로 `todo.id`를 사용하고 있기 때문이다. `addTodo` 액션 생성자는 로컬 변수 `nextTodoId`를 사용하며 기본값으로 `0`이 지정된다.

##### `addTodo` 이전:
```javascript
let nextTodoId = 0

export const addTodo = (text) => ({
  type: 'ADD_TODO',
  id: (nextTodoId++).toString(),
  text
})
```

이것을 회피하기 위해, `node-uuid` 라는 npm 모듈을 설치한다:

`$ npm install --save node-uuid`

모듈을 사용하기 위해, `node-uuid` 로 부터 `v4`를 import 하고 `(nextTodoId++)` 대신 호출한다. _노트: `v4`는 표준의 이름일 뿐이다._

##### `addTodo` 이후:
```javascript
import { v4 } from 'node-uuid'

export const addTodo = (text) => ({
  type: 'ADD_TODO',
  id: v4(),
  text
})
```
이제 모든 Todo 아이템들은 다시 로딩을 해도 보존 된다.

### Throttling `saveState()`
현재 구독하는 리스너의 내부에서 `saveState()`를 호출하기 때문에 저장소 상태가 변경될 때마다 호출 된다. 자원이 많이 소모되는 `stringify` 동작이 사용되기 때문에 너무 잦은 호출은 피하고 싶다.

이 문제를 수정하기 위해 `throttle` 기능이 포함된 `lodash`라는 npm 모듈을 사용할 것이다.

`$ npm install --save lodash`


#### `store.subscribe()` 이전:
```javascript
store.subscribe(() => {
  saveState({
    todos: store.getState().todos
  })
})
```

컬백을 `throttle`로 감싸서, 밀리세컨드 내에 특정 회숫 이상 호출되지 않는 것을 보장 받을 수 있다.

한개 함수를 사용하기 위해 전체 라이브러리를 가져오는 대신, `lodash` 로 부터 직접 `throttle`을 import 한다.

#### `store.subscribe()` 이후:
```javascript
// index.js 제일 위쪽
import throttle from 'lodash/throttle'
.
.
.
store.subscribe(throttle(() => {
  saveState({
    todos: store.getState().todos
  })
}, 1000))
```

이제, 저장소가 진짜 빠르게 업데이트 된다 해도, `localStorage`에 적는 건 최대 1초에 한번을 보장 할 수 있다.

#### [Recap at 6:05 in video](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage#/tab-transcript)
