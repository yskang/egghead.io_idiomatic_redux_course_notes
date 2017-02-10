# 02\. Supplying the Initial State

[비디오 링크](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)

리덕스 스토어를 생성할 때, 초기 상태는 root 리듀서에 의해 결정된다.

이경우, root 리듀서는 `todos` 와 `visibilityFilter` 에서 `combineReducers`를 호출한 결과다.

##### _src/index.js` 내부_
```javascript
const store = createStore(todoApp)
```


##### _`src/reducers/index.js` 내부_
```javascript
const todoApp = combineReducers({
  todos,
  visibilityFilter
})
```

리듀서들은 자치적이기 때문에, 각각은 자신의 고유한 초기 상태를 `state` 인자의 기본값으로서 지정한다.

`const todos = (state = [], action) ...` 문구 안의 기본 상태는 빈 배열이다.

`const visibilityFilter = (state = 'SHOW_ALL', action) ...` 안의 기본 상태값은 문자열 이다.

그래서, 결합된 리듀서의 초기 상태는 `todo` 키 아래에 빈 배열, `visibilityFilter` 키 아래에 문자열 `'SHOW_ALL'`로 구성된 object가 될 것이다.

##### 초기 상태:
```javascript
const todoApp = combineReducers({
    todos,
    visibilityFilter
})
```

하지만, 가끔 어플리케이션이 시작하기 전에 저장소에 데이터를 넣고 싶을 때가 있다.

예를 들어, 이전 세션에서 Todos가 남은 경우가 있을 것이다.

리덕스는 `persistedState`를 `createStore`함수의 두번째 인자로 넘김으로서 가능하게 해준다.
```javascript
const persistedState = {
  todos: [{
    id: 0,
    text: 'Welcome Back!',
    completed: false
  }]
}

const store = createStore(
  todoApp,
  persistedState
)
```

`persistedState`를 넘기면, 지정된 항목은 리듀서의 기본값을 덮어쓴다. 이 예제에서, `todos`는 배열로 제공하지만, `visibilityFilter`에는 특정한 값을 정하지 않기 때문에, 기본값인 `SHOW_ALL`이 사용된다.

`createStore()` 함수의 두번째 인자로 제공하는 `persistedState`는 리덕스 자체로부터 얻을 수 있기 때문에, 리듀서의 캡슐화를 깨트리지 않는다.

`persistedState` 안에 전체 저장소의 초기 상태를 제공하는 것을 하고 싶겠지만 권장하지는 않는다. 이렇게하면 나중에 reducer를 테스트하고 변경하는 것이 어려워진다.

#### [Recap at 1:42 in video](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)
