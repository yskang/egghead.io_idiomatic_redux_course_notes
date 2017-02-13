# 08. `withRouter()`를 사용하여 연결된 컴포넌트들에 Params 주입하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components)

현재 `App` 컴포넌트 안의 Router에 의해 `params.filter`가 전달 되는 것을 읽어 들이고 있다. router는 경로 설정에 지정되어 있는 어떤 Route 핸들러 컴포넌트로도 `params` prop을 주입 시킬 수 있기 때문에 router 부터 접근 할 수 있다.

하지만, `App` 컴포넌트 자신은 `filter`를 사용하지 않고, `VisibleTodoList`에 전달 하기만 할 뿐이며, `VisibleTodoList`는 현재 보여지는 Todo들을 계산하기 위해 사용한다.

Route 핸들러의 최고 레벨에서 `params`를 전달 하는 것은 길고 반복적인 일이기 때문에, `filter` prop을 제거 한다. 대신, `VisibleTodoList` 자신에게서 현재 Router `params`를 읽는 방법을 찾을 것이다.

#### `App.js` 이전:
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

#### `App.js` 이후:
```javascript
const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
);
```

## `VisibleTodoList` 갱신
React Router (version 3+) 에서 `withRouter`을 가져오는 것부터 시작한다.:

`import { withRouter } from 'react-router'`

`withRouter`는 React 컴포넌트를 받아서 그 컴포넌트에 `params` 같은 걸로 라우터와 관련있는 props를 주입시킨 새오룬 React 컴포넌트를 반환한다.

`mapStateToProps` 안에서 `params`를 사용하기를 원하기 때문에, 연결된 컴포넌트가 props으로 `params`를 가지게 되도록 `connect()` 결과를 래필 해야 한다.


또한,`ownProps`에서 직접 읽은 대신 `ownProps.params`로 부터 `filter`를 읽을 수 있도록 `mapStateToProps`를 변경 해야 한다.


이전 처럼, `'all'`을 or로 추가해 준다.

#### `VisibleTodoList` 이전:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.filter
  ),
});

// mapDispatchToProps ...

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList);
```

#### `VisibleTodoList` 이후:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.params.filter || 'all'),
});

// mapDispatchToProps ...

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

ES6 구조화 문법에 나오는 인수 정의로 부터 직접 `params`를 읽게 해서 `mapStateToProps`를 좀 더 간결하게 만들 수 있다.

```javascript
const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all'),
});
```

[Recap at 1:51 in video](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components#/tab-transcript)
