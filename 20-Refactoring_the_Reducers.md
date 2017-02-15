# 20. 리듀서 리팩토링
[비디오 링크](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)

앞에서, `visibilityFilter` 리듀서를 제거 해서, 앱에 있는 루트 리듀서는 오직 한개로 합쳐진 `todos` 리듀서만 있게 되었다. `index.js`는 `todos` 리듀서의 프록시로서 효율적으로 동작 하기 때문에, `index.js`은 완전히 없앨 것 이다. 그리고 나서 `todos.js`의 이름을 `index.js`로 변경해서, 새로운 루트 리듀서로 `todos`를 만들 것 이다.

루트 리듀서 파일은 이제 `byId`, `allIds`, `activeIds`, `completedIDs`를 포함 한다. 이들 중 일부를 추출해서 별도의 파일로 만들 것 이다.

`byid.js`라는 파일을 생성해서, `byId` 리듀서를 위한 코드를 붙여 넣는다.

상태가 `byId`리듀서의 상태와 일치 할 때, `state`와 `id`를 받는 `getTodo`라고 불리는 선택자를 위해 export 하는 것을 추가 할 것 이다. 이제 `index.js`로 돌아가서, 기본 import로 리듀서를 import할 수 있다.

네임스페이스 import를 사용해 단일 객체에서 관련된 선택자들중 어떤 것 이라도 import 할 수 있다:

```javascript
import byId, * as fromById from './byid'
```

이제 ID들을 관리하는 리듀서들을 본면, 코드는 `action.filter`와 필터 값을 피교하는 것만 빼면 거의 정확히 일치 하는 것을 볼 수 있다.

### `createList` 생성하기

`filter`를 인자로 하는 `createList`라 불리는 새로운 함수를 만들자.

`createList`는 다른 함수 - 특정한 필터를 위한 `id`들을 다루는 리듀서 -를 반환할 것 이기 때문에, 상태의 모습은 배열이다.  시간을 절약하기 위해, `allIds`로 부터 구현을 복사해 와서 붙여 넣을 수 있고, 단지 `'all'` 을 `createList`의 `filter` 인자로 바꾸기만 하면되므로, 어떤 필터에 대해서도 만들 수 있다.

```javascript
const createList = (filter) => {
  return (state = [], action) => {
    if (action.filter !== filter) {
      return state;
    }
    switch (action.type) {
      case 'RECEIVE_TODOS':
        return action.response.map(todo => todo.id);
      default:
        return state;
    }
  };
};
```

이제 `allIds`, `activeIds`, `completedIds`리듀서 코드를 완전히 제거 할 수 있다. 대신, 새로운 `createList` 함수를 이용해 리듀서들을 생성 할 것 이며, 인자로 필터를 넘길 것 이다.

다음으로, `createList` 함수를 추출해서 `createList.js` 라는 파일을 새로 만들어서 집어 넣는다.

이제 별도의 파일안에 존재 하게 되서, 선택자의 형태에 구성에 있는 상태에 접근하는 공개 API를 추가 할 것 이다. 이제 부터, 이것을 `getIds`라고 부를 것 이고, 단지 리스트의 상태를 반환 할 것 이다(나중에 이것을 바꿀 수 있다).

#### 마무리 하기

`index.js`로 돌아와서, 이 파일로 부터 `createList`와 어떤 이름을 가진 선택자도 import 할 것 이다.

```javascript
import createList, * as fromList from './createList';
```

또한, `idsByFilter`를 `listByFilter`로 이름을 바꿀 것인데 이유는 이제 리스트 구현은 별도의 파일에 있기 때문이며, `listByFilter`가 export 하는 `getIds` 선택자를 사용할 것이다.

```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = fromList.getIds(state.listByFilter[filter]);
  return ids.map(id => fromById.getTodo(state.byId, id));
};
```

`byId` 리듀서를 별도의 파일로 옮겼기 때문에, 그것이 단지 lookup 테이블이라는 가정을 하지 않는 것을 확신하고 싶다. 지점을 마음에 두고, 상태를 export하고 해당하는 ID를 전달하는 `fromById.getTodo` 선택자를 사용할 것 이다.

이 리팩토링으로, 코드기반 전반적으로 추가적인 변화 없이 앞으로 어던 리듀서의 상태 모양도 변결 할 수 있다.

[Recap at 3:41 in video](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)
