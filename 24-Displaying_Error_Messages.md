# 24. 에러 메세지 표시 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-displaying-error-messages)

때때로 API 요청이 실패 하는데, 가짜 API 클라이언트 내부에 `throw`ing 해서 거부된 Promise를 반환 하게 하는 것으로 이것을 시뮬레이트 할 것 이다. 만일 앱을 실행 시키면, 로딩 인디케이터는 `isFetching` 플래그가 `true`로 세팅 되지만, 그에 해당하는 `receiveTodos` 액션이 없어서 다시 `false`로 돌아 가기 때문에 동작에 문제가 생긴다.

### 수정

액션 생성자 파일(`actions/index.js`) 내부를 일부 깨끗하게 하는 것으로 시작을 할 것 이다.

`requestTodos` 액션은 `fetchTodos` 외부에서 전혀 사용되지 않기 때문에, 그 안에 `requestTodos` 객체 리터럴을 그 안에 내장 시킬 수 있다. `receiveTodos`가 발행된 곳에서 `fetchTodos`의 내부에 복사 붙여넣기해서 `receiveTodos`와 같은 것을 할 수 있다. `Promsie.then` 메소드에 두번째 인자로 거부 핸들러를 추가 할 것 이다.

#### Inside `fetchTodos`
```javascript
return api.fetchTodos(filter).then(
  response => {
    dispatch({
      type: 'RECEIVE_TODOS',
      filter,
      response,
    });
  },
  error => {
    // To be filled in
  }
);
```

### 명확하게 하기 위해 액션들 이름 바꾸기

`fetchTodos` 액션 생성자가 몇개의 액션들을 발행 하기 때문에, 좀 더 일관되게 이름을 바꿀 것 이다:
  * `'RECEIVE_TODOS'`는 `'FETCH_TODOS_REQUEST'`로, todo들을 요청 한다는 의미로
  * `'RECEIVE_TODOS'` 는 `'FETCH_TODOS_SUCCESS'`로, todo들을 성공적으로 가조 왔다는 의미로
  * `error` 핸들러 `'FETCH_TODOS_FAILURE'` 추가, todo들을 가져오는데 실패 했을 때 사용하기 위해

Our `error` handler will also be passed two additional pieces of data: the `filter` and the `message` that can be read with `error.message` if specified. We will use `'Something went wrong.'` as a fallback.

`error` 핸들러는 데이터의 두개의 추가적인 조각을 전달 받을 것 이다: `filter`와 만일 특정 지어 졌다면 `error.messsage` 로 일혀 지는 `message`. 기본 값으로 `'Something went wrong'`을 사용할 것 이다.

#### 수정된 `fetchTodos` `return`
```javascript
return api.fetchTodos(filter).then(
  response => {
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
  error => {
    dispatch({
      type: 'FETCH_TODOS_FAILURE',
      filter,
      message: error.message || 'Something went wrong.',
    });
  }
);
```

Now our `fetchTodos` action creator handles all the cases, and we can remove the old action creators that are now inlined (`requestTodos` and `receiveTodos`).

새로운 `fetchTodos` 액션 생성자는 모든 경우를 핸들링 하고, 이제 inlined된 오래된 액션 생성자들을 제거 할 수 있다 (`requestTodos`와 `receiveTodos`).

### 리듀서들 수정 하기

액션 타입들을 변경 했기 때문에, 이제 그에 해당하는 리듀서들을 변경 해야 한다.

`ids` 리듀셔는 `RECEIVE_TODOS` 대신 `FETCH_TODOS_SUCCESS` 핸들을 필요로 한다.

`isFetching` 리듀서는 `REQUEST_TODOS` 대신 `FETCH_TODOS_REQUEST` 핸들을 필요로 하고, `RECEIVE_TODOS` 대신 `FETCH_TODOS_SUCCESS`를 필요로 한다.

또한, `false`를 반환 함 으로서 `FETCH_TODOS_FAILURE`를 다룰 수 있어서 인티케이터 동작은 더이상 문제가 생기지 않는다.

마지막으로 변경할 리듀서는 `byId` 인데, `RECEIVE_TODOS`를 `FETCH_TODOS_SUCCESS` 변경 한다.

#### `isFetching` 리듀서 수정
```javascript
const isFetching = (state = false, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'FETCH_TODOS_REQUEST':
        return true;
      case 'FETCH_TODOS_SUCCESS':
      case 'FETCH_TODOS_FAILURE':
        return false;
      default:
        return state;
    }
  };
```

With these changes, the loading indicator won't get stuck because a corresponding failure action fires, resetting `isFetching` back to `false`.

### Displaying the Error

We'll create a new file `FetchError.js` in our `components` directory.

After `import`ing `React`, we'll create a new functional stateless component `FetchError` that will take two props: a `message` string, and an `onRetry` function. This component will be the default export for this file.

The rendered `<div>` will contain an error saying that something bad happened (including the message that is passed in the props), and a button that when clicked will invoke the `onRetry` callback prop so that the user can retry fetching the data.

##### `FetchError` Component
```javascript
const FetchError = ({ message, onRetry }) => (
  <div>
    <p>Could not fetch todos. {message}</p>
    <button onClick={onRetry}>Retry</button>
  </div>
);
```

### `VisibleTodoList`에 `FetchError` 추가 하기

`VisibleTodoList`에 `import FetchError`가 필요하고 나서, `render` 메소드를 업데이트 한다.

에러 메세지를 받기 위해, `VisibleTodoList` 컴포넌트의 `props`로 부터 재구조화 할 필요가 있다.

```javascript
// Inside VisibleTodoList
render() {
  const { isFetching, errorMessage, toggleTodo, todos } = this.props;
  ...
```

`render`의 내부에 "만일 prop에 에러 메세지가 있으면, 표시할 todo가 없다"라고 말해줄 다른 상태를 추가 할 것 이고, `FetchError` 컴포넌트를 반환 할 것 이다.

`FetchError` 컴포넌트 자체는 `message` prop을 원하는데, 방금 재구조화 한 `errorMessage` prop을 전달 받을 것 이다. `onRetry` 컬백 prop을 제공 받을 것인데, 이 컬백은 데이터를 가져오는 과정을 다시 시작하기 위해 `this.fetchData`를 호출 하는 에러 함수를 전달 할 것 이다.

```javascript
// Inside VisibleTodoList's `render()`
if (errorMessage && !todos.length) {
     return (
       <FetchError
         message={errorMessage}
         onRetry={() => this.fetchData()}
       />
     );
   }
```

가능 하게 하기 위해 `errorMessage`를 `VisibleTodoList`의 `mapStateToProps`에 넣을 필요가 있다. `isFetching` 에서 사용된 것과 같은 패턴을 따라서, `getErrorMessage`라 불리는 선택자를 호출함으로서 `errorMessage` prop을 얻고 엡의 `state`와 `filter`에 전달 한다.

```javascript
// Inside VisibleTodoList
const mapStateToProps = (state, { params }) => {
  const filter = params.filter || 'all';
  return {
    isFetching: getIsFetching(state, filter),
    errorMessage: getErrorMessage(state, filter),
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

### `getErrorMessage` 구현
파일의 위쪽에 있는 `VisibleTodoList`의 리듀서의 import에 `getErrorMessage`를 추가할 필요가 있다.
`import { getVisibleTodos, getErrorMessage, getIsFetching } from '../reducers';`

이제 루트 리듀서 파일(/reducers/index.js) 의 안에 `getErrorMessage` 선택자를 복사 붙여넣기 해서 만들고, `getIsFetching` 선택자를 리팩토링 한다.

#### 선택자 만들기
```javascript
export const getErrorMessage = (state, filter) =>
  fromList.getErrorMessage(state.listByFilter[filter]);
```

#### `createList` 수정

`createList.js` 안에, 새로운 export된 선택자 `getErrorMessage`를 추가 할 것 이며, 이 선택자는 리스트의 상태를 받고, 에러 메세지라 불리는 property를 반환 한다.

```javascript
export const getErrorMessage = (state) => state.errorMessage;
```

이제 새로운 리듀서 `errorMessage`를 초기 `state`를 `null`로 해서 선언 할 것 이다. 이렇게 하는 이유는 리듀서는 undefined 초기 상태를 가질 수 없기 때문이어서, 명시적으로 없는 상태를 만들어야 한다.

이 파일에 있는 다른 리듀서들과 같이, `createList`로 전달되는 인자로서 정해진 `filter`와 맞지 않는 필터를 가진 액션은 건너 뛰기를 원한다. 필터가 _맞으면_, 몇개의 액션을 핸들 하기를 원한다:

  * 만일 실패 했으면 에러 메세지를 출력 한다.
  * 만일 사용자가 재요청을 하면 에러 메세지를 지운다.
  * 다른 액션에 대해서는 현재 상태를 반환 한다.

`errorMessage` 리듀서는 역시 `combineReducers`에 추가 되어야 한다.

##### 완성된 `errorMessage` 리듀서
```javascript
const errorMessage = (state = null, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'FETCH_TODOS_FAILURE':
        return action.message;
      case 'FETCH_TODOS_REQUEST':
      case 'FETCH_TODOS_SUCCESS':
        return null;
      default:
        return state;
    }
  };

  return combineReducers({
    ids,
    isFetching,
    errorMessage,
  });
```

### API 수정
배번 에러를 던지는 API 대신, "retry" 버튼을 실행해 볼수 있게 랜덤하게 던지는 것을 할 수도 있다.

[Demonstration and recap at 6:43 in video](https://egghead.io/lessons/javascript-redux-displaying-error-messages)
