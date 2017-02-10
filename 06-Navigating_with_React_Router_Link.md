# 06. 리액트 라우터 사용해서 이동하기 `<Link>`
[비디오 링크](https://egghead.io/lessons/javascript-redux-navigating-with-react-router-link?series=building-react-applications-with-idiomatic-redux)

이번 강좌에서는 visiblility 필터를 컨트롤하는 "links"를 업데이트 해서 _진짜_ 링크처럼 동작하도록 할 것이다. 브라우저의 "뒤로가기" 버튼이 동작하게 하고 필터를 번경할 때 URL이 변하도록 할 것 이다.

We'll start by adding a parameter to the Route `path` called `filter`. We wrap it in parens to tell React Router that it's optional (we want all Todos shown by default).

Route 엘리먼트의 `path` 에 `filter`라는 파라메터를 추가 하는 것부터 시작한다. 괄호로 감쌈으로서 리액트 라우터에 선택사항 이라는 것을 알린다 (기본적으로 모든 Todo가 보여지기를 원한다).

#### Update `Root.js`
```javascript
const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/(:filter)" component={App} />
    </Router>
  </Provider>
);
.
.
.
```

### `Footer.js` 에서 Link 업데이트 하기
footer의 안쪽에 visibility 필터 링크들을 업데이트 할 필요가 있다.

#### `Footer.js` Before
```javascript
...
const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
);

export default Footer;
```

이전 구현은 `filter` prop 을 사용했지만, 주소창에 "active"와 "completed"가 표시되는 것을 하기 위해 업데이트 한다.

#### `Footer.js` 이후
```javascript
// Rest as above...
<p>
  Show:
  {" "}
  <FilterLink filter="all">
    All
  </FilterLink>
  {", "}
  <FilterLink filter="active">
    Active
  </FilterLink>
  {", "}
  <FilterLink filter="completed">
    Completed
  </FilterLink>
</p>
```

기본 주소를 나타내기 위해 빈 문자열 대신 null 문자열을 사용한다.

### `FilterLink.js` 구현 업데이트
현재 구현에서, `FilterLink` 컴포넌트는 클릭 될 때마다 매번번 액션을 발행하고, 스토어에서 상태를 읽어 스토어에 있는 `setVisibilityFilter`의 `filter` prop과 비교한다.

#### `FilterLink.js` 이전
```javascript
...
const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter,
});

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick() {
    dispatch(setVisibilityFilter(ownProps.filter));
  },
});

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link);

export default FilterLink;
```

하지만, 라우터가 URL에 있는 어떤 상태라도 제어 하기를 원하기 때문에 더이상 이 구현은 필요하지 않다. `react-router`로 부터 `Link`를 import 해서 새로운 구현에 사용할 것이다.

`FilterLink` 는 이제 `filter`를 prop으로 받고, 리액트 라우터의 `Link`를 이용해 랜더링 한다.

`to` prop은 가리키는 지점에 대한 링크의 경로와 일치 하기 때문에, URL의 경로로 만일 `filter`가 `'all'`이면 root 경로를 사용할 것 이고, 그렇지 않으면 `filter` 자체를 사용할 것 이다.


또한, `to` prop이 현재 경로와 일치할 때, 스타일을 다르게 하기 위해 `activeStyle` prop을 사용할 것이다.

`Link` 자신에 `children`을 전달 하고, `children`을 `FilterLink`에 prop으로 추가해서 부모 컴포넌트가 자식을 특정 지을 수 있도록 한다.

#### `FilterLink.js` 이후
```javascript
import React, { PropTypes } from 'react';
import { Link } from 'react-router';

const FilterLink = ({ filter, children }) => (
  <Link
    to={filter === 'all' ? '' : filter}
    activeStyle={{
      textDecoration: 'none',
      color: 'black',
    }}
  >
    {children}
  </Link>
);

FilterLink.propTypes = {
  filter: PropTypes.oneOf(['all', 'completed', 'active']).isRequired,
  children: PropTypes.node.isRequired,
};

export default FilterLink;
```

### 더 할 것들...
더이상 `setVisibilityFilter` 액선 생성자를 사용하지 않기 때문에, `src/action/index.js` 에서 제거할 수 있으니, `addTodo`와 `toggleTodo` 액션 생성자만 남겨둔다.

We can also delete our custom `Link` component from `src/components` since we are now using the one provided by `react-router`.

또한, `react-router`에서 제공하는 것을 사용하기 때문에 `src/components`에서 커스텀 `Link` 컴포넌트를 지울 수 있다.

[Recap at 2:20 in video](https://egghead.io/lessons/javascript-redux-navigating-with-react-router-link?series=building-react-applications-with-idiomatic-redux)
