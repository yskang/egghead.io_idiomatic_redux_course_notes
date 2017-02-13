# 09. `mapDispatchToProps()` 간결한 형식 사용하기
[Video Link](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)

`mapDispatchToProps` 함수는 React 컴포넌트에 어떤 props을 주입 시켜서 액션을 발행 할 수 있게 해준다. 예를 들어, `TodoList` 컴포넌트는 `todo`의 `id`릉 가지고 `onTodoClick` 컬백 prop을 호출한다.

#### TodoList` 내부
```javascript
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
```

`VisibleTodoList`컴포넌트에 있는 `mapDispatchToProps` 안에 `id`로 `onTodoClick()`이 호출 되면 이 `id`로 `toggoleTodo` 액션이 발행 되도록 정의 해 놓았다. `toggleTodo` 액션 생성자는 이 `id`를 사용해서 발행될 액선 객채를 생성한다.

#### `VisibleTodoList` `mapDispatchToProps`
```javascript
const mapDispatchToProps = (dispatch) => ({
  onTodoClick(id) {
    dispatch(toggleTodo(id));
  },
});

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

컬백 prop의 인자가 액선 생성자의 인자와 정확하게 일치 할 때, `mapDispatchToProps`를 간결하게 할 방법이 있다.

함수를 전달하는 대신 주입 하려는 컬백 props의 이름과 일치하는 액션을 생성하는 액션 생성자 함수의 객체 매핑을 전달 할 수 있다.

일반적으로 `mapDispatchToProps`를 자주 사용할 필요가 없고, 대신 객체 맵핑을 전달 할 수 있다.

#### `VisibleTodoList` 이후:
```javascript
const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo }
)(TodoList));
```

[Recap at 1:13 in video](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)
