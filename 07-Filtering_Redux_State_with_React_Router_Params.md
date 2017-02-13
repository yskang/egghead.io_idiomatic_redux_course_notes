# 07. React Router Params로 Redux State 필터링 하기

[비디오 링크](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)

이제 React Router에서 제공하는 `Link`를 사용하고 있어서, URL 링크를 클릭하면 갱신 되게 된다. 하지만, 내용은 갱신 되지 않는데 왜냐하면 `mapStateToProps` 함수 한에 있는 보여지는 `TodoList` 컴포넌트는 여전히 URL로 부터 읽어 오는 값이 아닌 Redux 저장소에 있는 `visibilityFilter`에 의존하고 있기 때문이다.

#### `mapStateToProps` 이전:
```javascript
const mapStateToProps = (state) => ({
  todos: getVisibleTodos(
    state.todos,
    state.visibilityFilter
  )
})
```

이 것을 해결하기 위해, `ownPorps` 라는 매개변수를 추가하고, `ownProps`로 부터 `visibilityFilter`를 읽을 것이다.

#### `mapStateToProps` After:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.filter
  )
})
```

현재 규칙인 필터 props `'all'`, `'completed'`, `'after'`을 사용하기 위해 `getVisibleTodos` 함수 또한 갱신한다.

#### `getVisibleTodos` 이전:
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos;
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed);
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

#### `getVisibleTodos` 이후:
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
```

## `App.js` 수정

`VisibleTodoList` 컴포넌트는 app안에서 랜더되므로, `VisibleTodoList`의 `mapStateToProps` 함수 안에서 가능하도록 `filter` prop을 추가할 필요가 있다.

경로 설정(이전 강좌에서 설정한 `path='/(:filter)'`)에 있는 현재 `filter` 매개변수와 필터 prop이 일치하도록 해야 한다. React Router는 이 매개변수들을 `params`라 불리는 특별 prop에 있는 경로 핸들러 컴포넌트에서 사용 가능하도록 만들기 때문에, `App`에 prop으로 `params`를 추가 할 것 이다. 이제 `params.filter`로 부터 필터를 일을 수 있게 된다.

루트 경로 일 때 `filter` param은 비어 있기 때문에, `VisibleTodoList`에 `'all'`을 or 로 추가 한다.

```javascript
const App = ({ params }) => (
  <div>
    <AddTodo />
    <VisibleTodoList
      filter={params.filter || 'all'}
    />
    <Footer />
  </div>
);
```

이제 visiblity 필터는 React Router에 의해 관리 되서, 더이상 `visibilityFilter` 리듀서는 필요하지 않다. 지우고, `index.js`에 선언된 `combineReducers()` 에서 제가 할 수 있다.

[Recap at 2:20 in video](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)
