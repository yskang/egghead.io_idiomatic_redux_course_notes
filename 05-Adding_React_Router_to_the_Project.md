# 05. 프로젝트에 리액트 라우터 추가하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux)

이번 강좌에서는 리액트 라우터을 추가할 것 이다.

#### `Root.js` 이전
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
);
.
.
.
```

리액트 라우터를 프로젝트에 추가하기 위해, 아래 명령어를 실행 해라:
`$ npm install --save react-router`

`Root.js` 내부에 `Router`와 `Route` 컴포넌트를 import 한다.

`<App />`을 `<Router />`로 대체 한다. 여전히 `<Provider />`의 안에 여전히 존재해서 라우터에 의해 랜더되는 어떤 컴포넌트들도 여전히 스토어에 접근 가능하다는 것은 중요하다.

Inside of `<Router />` we will put a single `<Route />` element that tells React Router that we want to render our `<App />` component at the root path (`'/'`) in the browser's address bar.

`<Router />`의 안에 `<Route />`를 하나 넣어서 브라우저의 주소창에 있는 루트 주소 (`'/'`)에 `<App />`을 그리고 싶다는 것을 리액트 라우터에 알린다.

#### `Root.js` 이후
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { Router, Route, browserHistory } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/" component={App} />
    </Router>
  </Provider>
);
.
.
.
```

_Note: the video contains a fix for weird address bar symbols stemming from an old release of `react-router`_

_노트: 비디오는 `react-router`의 이전 버전 사용으로 인한 주소창의 이상한 심볼 사용에 대한 수정이 포함되어 있다._

[Recap at 1:09 in video](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux#/tab-transcript)
