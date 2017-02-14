# 13. 프로젝트에 가짜 백엔드 추가 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-adding-a-fake-backend-to-the-project)

다음 강좌에서, 더이상 지속성에 대해 다루지 않을 것이다. 대신, 앱에 비동기적으로 데이터를 가져 오는 것을 추가 할 것이다.

이제 `localStorage` 지속성에 관련된 코드를 다 없애고, `localStorage.js` 파일 역시 지울 것 인데, 왜냐 하면 가짜 원격 API의 구현하는 새로운 모듈이 추가되어 있기 때문이다.

모든 todos는 메모리에 저장되어 있고, 인공적인 지연이 추가 되어 있다. 또한 진짜 API 구현처럼 Promises를 반환하는 메소드들을 가지고 있다.

#### `src/api/index.js`
```javascript
import { v4 } from 'node-uuid';

// This is a fake in-memory implementation of something
// that would be implemented by calling a REST server.

const fakeDatabase = {
  todos: [{
    id: v4(),
    text: 'hey',
    completed: true,
  }, {
    id: v4(),
    text: 'ho',
    completed: true,
  }, {
    id: v4(),
    text: 'let’s go',
    completed: false,
  }],
};

const delay = (ms) =>
  new Promise(resolve => setTimeout(resolve, ms));

export const fetchTodos = (filter) =>
  delay(500).then(() => {
    .
    .
    .
```

이런 접근은 앱을 위한 진짜 백앤드를 작성하지 않고도 Redux가 비동기적으로 데이터를 가져오는지 알 수 있게 해 준다.

이제 앱의 다른 모듈들에 `fetchTodos`를 imoprt 할 것이다.

`import { fetchTodos } from './api'`

이런 todos를 Redux 저장소에 어떻게 넣는지 나중에 배울 것이지만, 지금은, REST 백앤드가 배열을 반환하는 것 처럼, todos의 배열을 통해 확인되는 Promise를 반환하는 필터를 인자로 `fetchTodos`를 호출 할 수 있다.

#### `index.js`
```javascript
fetchTodos('all').then(todos =>
  console.log(todos)
);
```

가짜 API는 네트워크 연결을 시뮬레이트 하기 위해 500ms동안 기다리고 나서, 마치 원격지 서버로 부터 받는 것 처럼 totos의 배열을 promise를 통해 반환 한다.
