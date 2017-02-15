# 18. Redux 미들웨어 적용하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)

이전 강좌에서, 미들웨어 함수들을 위해 사용하기를 원하는 원칙을 알아 봤다.

하지만, 미들웨어는 모두가 자신의 것에 `wrapDispatchWithMiddlewares`를 구현해야 한다면 그렇게 활용성이 좋지 않을 것 이다.

Now we'll remove it, and instead import a utility called `applyMiddleware` from Redux.
이제 만든것을 지워 버리고, 대신 `applyMiddleware`라 불리는 Redux에서 제공하는 유틸리티를 import 하자.

#### `configureStore` 이전:
```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];

  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(logger);
  }

  wrapDispatchWithMiddlewares(store, middlewares);

  return store;
};
```

`configureStore` 함수를 보면, 당장 `store`가 필요 없다는 것을 알 수 있다. `store`의 생성부분을 미들웨어가 지정된 곳 아래로 보낼 수 있다.

또한 직접 만든 `wrapDispatchWithMiddlewares` 함수 또한 제거 할수 있고, 대신 바로 미들웨어로 `createStore`를 대신 할수 있다.

저장소를 생성하는 두번째 인자는 장소를 지정하는 인자로서 미들웨어 함수들을 넣어서 `applyMiddleware`를 호출하는 결과가 될 것 이다.

`createStore`에 들어가는 마지막 인자는 개선자로 불리는데, 필수사항은 아니다. 만일 `persistedState`를 지정하기를 원한다면, 개선자 이전에 해야 한다(만일 `persistedState`를 가지고 있지 않다면 역시 건너뛸 수 있다).

#### `configureStore` 이후:
```javascript
// We also deleted `wrapDispatchWithMiddlewares()`

const configureStore = () => {
  const middlewares = [promise];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(createLogger());
  }

  return createStore(
    todoApp,
    applyMiddleware(...middlewares)
  );
};
```

### 직접 만든 미들웨어 교체하기

많은 미들웨어들은 npm 패키지로 이용 가능 하다. 직접 만든 `promise` 와 `logger` 미들웨어 둘 다 예외는 없다.

터미널에서 `npm install --save redux-promise`을 실행 하면 Promise를 지원하도록 구현된 미들웨어가 설치 될 것 이다.

같은 방식으로 `redux-logger`라 불리는 패키지도 설치 할 수 있는데, 직접 작성한 logger 미들웨어와 비슷하지만, 좀 더 다양한 설정을 할 수 있다.

`configureStore.js` 안에, 이제 새로운 미들웨어를 import 해서 사용할 수 있다. 더이상 `store`를 참조할 필요가 없기 때문에, `configureStore`로 부터 직접 반환 하기만 할 수 있다.

#### `configureStore.js` 수정
```javascript
import { createStore, applyMiddleware } from 'redux';
import promise from 'redux-promise';
import createLogger from 'redux-logger';
import todoApp from './reducers';

const configureStore = () => {
  const middlewares = [promise];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(createLogger());
    // Note: you can supply options to `createLogger()`
  }

  return createStore(
    todoApp,
    applyMiddleware(...middlewares)
  );
};

export default configureStore;
```

[Recap at 2:06 in video](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)
