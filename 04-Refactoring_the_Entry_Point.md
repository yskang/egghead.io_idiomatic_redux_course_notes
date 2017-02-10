# 04. 진입점 리팩토링
[비디오 링크](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux)

이번 강좌에서는 스토어의 생성 및 구독을 위한 로직을 추출해서 새로운 파일로 분리 할 것이다.

#### `index.js` 이전
```javascript
import 'babel-polyfill'
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import throttle from 'lodash/throttle'
import todoApp from './reducers'
import App from './components/App'
import { loadState, saveState } from './localStorage'

const persistedState = loadState()
const store = createStore(
  todoApp,
  persistedState
);

store.subscribe(throttle(() => {
  saveState({
    todos: store.getState().todos,
  })
}, 1000))

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

파일을 새로 만들어 이름을 `configureStore.js`로 한 후, 스토어를 생성 및 저장 하는 로직을 담고 있는 함수 `configureStore` 함수를 생성 하는 것 부터 시작한다.

이렇게 하는 이유는 앱이 어떻게 스토어가 생성되고, 핸들러를 구독하는지 여부를 정확하게 알수 없도록 하기 위해서 이다. 앱은 단지 `index.js`로 부터 반환받은 스토어를 사용할 수 있다.

####`configureStore.js`
```javascript
import { createStore } from 'redux'
import throttle from 'lodash/throttle'
import todoApp from './reducers'
import { loadState, saveState } from './localStorage'

const configureStore = () => {
  const persistedState = loadState()
  const store = createStore(todoApp, persistedState)

  store.subscribe(throttle(() => {
    saveState({
      todos: store.getState().todos
    })
  }, 1000))

  return store
}

export default configureStore
```

`store`만이 아닌 `configureStore`를 export 함 으로서, 테스트를 위해 필요한만큼 스토어 인스턴스를 만들 수 있다.

#### `index.js` After
```javascript
import 'babel-polyfill'
import React from 'react'
import { render } from 'react-dom'
import App from './components/App'
import configureStore from './configureStore'

const store = configureStore()
render(
  <Root store={store} />,
  document.getElementById('root')
);
```

랜더된 root 엘리먼트를 `Root`라고 불리는 분리된 컴포넌트로 추출한 것을 주목해라. 이 컴포넌트는 prop으로 `store`를 받고, `src/components` 폴더에 분리된 파일로 정의 될 것이다.

---

#### `Root` Component
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
);

Root.propTypes = {
  store: PropTypes.object.isRequired,
};

export default Root;
```

단지 `store`를 prop으로 받고, `react-redux`의 `Provider` 대신 `<App />`을 반환하는 상태없는 함수형 컴포넌트를 정의 했다.

이제 `index.js` 안에서, `Provider`를 import한 것을 지울 수 있고, `App` 컴포넌트를 대신해 `Root` 컴포넌드를 import 할 수 있다.

[Recap at 1:45 in video](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux#/tab-transcript)
