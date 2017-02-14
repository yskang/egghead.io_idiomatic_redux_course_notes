# 15. 가져온 데이터로 액션 발행 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)

`VisibleTodoList` 안에 중단한 것을 이어서, `fetchData`라는 분리된 메소드로 라이프 사이클을 가로채는 코드 사이의 공용 코드를 추출 할 것 인데, 이때 데이터는 오직 필터에 기반한 것만 가져오기를 원한다.

초기 데이터를 가져오기 위해 `componentDidMount()` 가로채기로 부터 이 메소드를 호출한다. 또한 `filter`가 변경될 때 마다 `componentDidUpdate` 라이프 사이클 가로채기 안쪽에서 호출 할 것 이다.

#### `VisibleTodoList.js`
```javascript
class VisibleTodoList extends Component {
  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    if (this.props.filter !== prevProps.filter) {
      this.fetchData();
    }
  }

  fetchData() {
    fetchTodos(this.props.filter).then(todos =>
      console.log(this.props.filter, todos)
    );
  }
  .
  .
  .
```

### `fetchData()` 개선
Redux 저장소의 `state`의 일부분이 되도록 todo들을 가져오기 원한며, 상태로 무언가를 가져오는 유일한 방법은 액션을 발행하는 것 이다.

방금 가져온 `totos`를 가지고 `receiveTodos` 컬백 prop을 호출 할 것 이다.

```javascript
fetchData() {
  fetchTodos(this.props.filter).then(todos =>
    this.props.receiveTodos(todos)
  );
}
```

컴포넌트 안에서 사용가능 하게 하기 위해, `receiveTodos`라고 불리는 함수를 전달 할 필요가 있으며, 액션 생성자 로서 `connect`의 두번째 인자 될 것이다. 함수의 이름이 컬백 prop과 일치 하기 때문에 ES6에 있는 객체 속성 표기 방법을 이용해 좀 더 짧게 표현 할 수 있다.

또한, 이외 모든 액션 생성자들이 정의된 파일로 부터 `receiveTodos`를 import 할 것 이다.

```javascript
// `VisibleTodoList.js` 제일 위
import { toggleTodo, receiveTodos } from '../actions'

...

// `VisibleTodoList.js` 아랫 쪽
VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo, receiveTodos }
)(VisibleTodoList))
```

### `receiveTodos` 구현 하기

이제 `receiveTodos`를 실제 구현하는것이 필요하다.

액선 생성자 파일(`src/actions/index.js`) 에서 인자로 몇개의 서버 `response`를 받고 필드로 `'RECEIVE_TODOS'`의 `tpye` 과 `response`를 가지는 객체를 반환하는 새로운 `receiveTodos` 함수를 만들어서 export 한다.

#### `src/actions/index.js` 내부
```javascript
export const receiveTodos = (response) => ({
  type: 'RECEIVE_TODOS',
  response,
});
```

리듀서들이 이 액션을 다루는 것은 응답하는 필터가 어떤 것인지 알 필요가 생길 것 인데, 그래서 `receiveTodos` 액션 생성자에 인자로 `filter`를 추가 할 것 이고 액션 객체의 일부로서 전달 할 것 이다.

```javascript
export const receiveTodos = (filter, response) => ({
  type: 'RECEIVE_TODOS',
  filter,
  response,
});
```

### `filter`로 `VisibleTodoList` 컴포넌트를 갱신하기

`VisibleTodoList`로 돌아와서, 액션 생성자를 통해 필터를 전달하기 위해 `fetchData`를 갱신해야 한다.

`pros`로 부터 `filter`와 `receiveTodos`를 얻어오기 위해 ES6 구조화 문법을 사용한다. 지금 당장 `filter`를 없애는 것은 중요한데, 이유는 컬백이 동작하는 시간에 의해 사용자가 다른 곳으로 가버려서 `this.props.filter`가 변경될 수 있기 때문이다.

#### `VisibleTodoList` 내부
```javascript
fetchData() {
  const { filter, receiveTodos } = this.props;
  fetchTodos(filter).then(todos =>
    receiveTodos(filter, todos)
  );
}
```

### 더 작은 보일러플레이트 코드 작성 하기

앱을 돌아다니면, 컴포넌트는 라이프 사이클 가로채기에서 데이터를 가져오고, 데이터가 준비되면 Redux 액션을 발행한다. 이제 좀 더 작은 보일러 플레이트를 작성해 보자.

네임스페이스 import를 사용해서 import들을 변경하는 것 부터 시작할 수 있다. 즉, actions 파일에서 export된 함수는 `actions`라는 객체 안에 있고, connect에 두번째 인자로 전달 될 것이다.

```javascript
// At the top of `VisibleTodoList`

// Before: import { toggleTodo, receiveTodos } from '../actions'
import * as actions from '../actions' // After
.
.
.
// At the bottom of `VisibleTodoList`
VisibleTodoList = withRouter(connect(
  mapStateToProps,
  // Before: { onTodoClick: toggleTodo, receiveTodos}
  actions // After
)(VisibleTodoList))

```

`VisibleTodoList`의 `render()`함수 안에서, `props`를 없앨 것 인데, 왜냐하면 `toggleTodo` 액션 생성자는 `onTodoClick` prop 이름을 전달 해야 하기 때문이다. 하지만, 나머지 props는 그대로 전달 된다.

`...rest` 객채는 이제 `toggleTodo`가 아닌 모든 props를 가지고 있기 때문에, 이를 통화 시킬 것 이다. 또한, `toggleTodo`도 `onTodoClick` prop으로서 전달 할 것 인데, 왜냐하면 `TodoList` 컴포넌트가 기대하는 것 이기 때문이다.

```javascript
// Inside of `VisibleTodoList`
  render() {
    const { toggleTodo, ...rest } = this.props;
    return (
      <TodoList
        {...rest}
        onTodoClick={toggleTodo}
      />
    );
  }
}
```

[Recap at 3:30 in video](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)
