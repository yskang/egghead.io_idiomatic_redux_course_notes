#14. 경로 변경에 따른 데이터 가져오기
[비디오 링크](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)

시작 하기 전에, `VisibleTodoList` 안에 있는 todos를 가져오기를 원하기 때문에, `index.js` 진입점에서 호출되는 테스트 API `fetchTodos`를 제거한다.

`VisibleTodoList.js`로 `fetchTodos`를 import 하는 것으로 시작한다.

`import { fetchTodos } from '../api';`


`VisibleTodoList` 컴포넌트는 주입된 props인 각각 생성된 중간 컴포넌트인 `connect`와 `withRouter`가 호출함 으로서 생성된다.

`fetchTodos`를 호출하는 라이프 사이클을 가로채는 `componentDidMount()`의 안쪽이 좋을 것이다. 하지만, 생성된 컴포넌트들의 사이프 사이클을 차로채서 override 할 수 없다. 이것은 새로운 React 컴포넌트를 만들어야 한다는 뜻 이다.

### 새로운 React 컴포넌트 만들기

`VisibleTodoList.js` 안에, `React`와 React의 기본 클래스인 `Component`를 import 할 것이다.

```javascript
import React, { Component } from 'react';
// other imports...

class VisibleTodoList extends Component {
  render() {
    return <TodoList {...this.props} />;
  }
}
.
.
.
```

여전히 이전과 동일하게 presentational 컴포넌트인 `TodoList`를 그리기를 원한다. 이 새로운 클래스를 추가 하는 유일한 목적은 라이프 사이클을 가로채서 거기에 무언가를 추가 하는 것 이다. `TodoList` 에 어떤 props라도 전달될 수 있게 될 것이다.

이제 `VisibleTodoList`는 위에 있는 클래스로 정의 되었고, 같은 이름으로 또다른 상수를 정의 할 수 없기 때문에, 래핑된 컴포넌트를 가르키기 위해 `VisibleTodoList`를 바인딩에 다시 지정 한다. 새로운 클래스를 래핑 하도록 `connect()`를 대신 호출하도록 변경 할 것 이다.

```javascript
VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo }
)(VisibleTodoList));

export default VisibleTodoList;
```

`connect()` 호출에 의해 생성된 컴포넌트는 정의된 `VisibleTodoList` 클래스를 그릴 것 이다. `connect`와 `withRouter`를 래핑해서 호출한 결과는 파일에서 export한 최종 `VisibleTodoList`이다.

### 라이프 사이클 가로채기 추가

컴포넌트가 올려질 때, 현재 필터로 todo들을 걸러서 가져오기를 원한다.

prop으로서 직접 사용가능한 필터를 가지고 있게 되면 매우 편리할 것 이므로, 이전에 한 것 처럼 `params`로 부터 `filter`를 계산 하도록 `mapStateToProps`를 수정 하지만, `return` 객체의 properties 중 하나로서 전달 할 것 이다.

#### `mapStateToProps` 수정
```javascript
const mapStateToProps = (state, { params }) => {
  const filter = params.filter || 'all';
  return {
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

라이프 사이클 메소드로 돌아가 보면, `componentDidMound` 안에 `this.props.filter`를 이용할 수 있다. todo들을 가져왔을 때, `fetchTodos`는 Promise를 반환 한다. 가져온 `todos`에 접근하기 위해 `then` 메소드를 이용할 수 있고, 현재 `filter`와 가짜 백앤드로 부터 유일하게 받은 `todos`를 로그로 기록한다.

#### `componentDidMount` 구현하기
```javascript
class VisibleTodoList extends Component {
  componentDidMount() {
    fetchTodos(this.props.filter).then(todos =>
      console.log(this.props.filter, todos)
    );
  }
```


현재 구현으로, 앱을 구동시키면 `all` 필터가 표시되고 그에 해당하는 `todos`가 나타 날 것 이다.

하지만, 필터가 변경 될 때 어떤 일도 일어나지 않을 것 인데, 이유는 `componentDidMount`가 한번만 실행 되기 때문이다. 이것을 수정하기 위해, `componentDidUpdate`라 불리는 두번째 라이프 사이클 가로채기를 추가 해야 한다.

#### `componentDidUpdate` 구현하기
```javascript
// inside the `VisibleTodoList` below `componentDidMount()`
componentDidUpdate(prevProps) {
   if (this.props.filter !== prevProps.filter) {
     fetchTodos(this.props.filter).then(todos =>
       console.log(this.props.filter, todos)
     );
   }
 }
```

`componentDidUpdate`는 인자로 이전 props을 받는다. 그러고 나서 필터의 현재와 과거 값을 비교한다. 만일 현재 필터가 이전 필터와 같지 않으면, 현재 필터를 인자로 `fetchTodos()`를 호출 한다.

[Recap at 3:36 in video](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)
