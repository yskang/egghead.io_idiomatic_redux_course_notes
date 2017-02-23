# 27. 서버에 있는 데이터 갱신 하기

[비디오 링크](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)

`toggleTodo` 액션 생성자를 thunk 액션 생성자가 되도록 변경해서, 커링 인자로 `dispatch`를 추가 하는 것 부터 시작 할 것 이다. 다음으로, `toggleTodo` API endpoint를 호출 하고, 응답이 오기를 기다릴 것 이ㅏㄷ.

응답이 가능하면, `'TOGGLE_TODO_SUCCESS'` 형 액션을 발행 하고, 응답 할 것 이다. 첫번째 인자로 원래 `response`를 전달하고, 두번째 인자로 todo schema를 전달해서 `normalize`를 다시 사용할 것 이다.

##### 갱신된 `toggleTodo` 액션 생성자
```javascript
export const toggleTodo = (id) => (dispatch) =>
  api.toggleTodo(id).then(response => {
    dispatch({
      type: 'TOGGLE_TODO_SUCCESS',
      response: normalize(response, schema.todo),
    });
  });
```

### `ids` 리듀서 갱신하기

`createList.js` 에 `'TOGGLE_TODO_SUCCESS'` 액션을 위한 새로운 case를 추가 할 것 이다.

이 case를 위해 코드를 함수로 추출해서 `handleToggle`로 하고, `state`와 `action`을 넘길 것 이다.

```javascript
// Inside the `ids` reducer
  case 'TOGGLE_TODO_SUCCESS':
    return handleToggle(state, action);
```

`ids` 리듀서 위에 `handleToggle` 함수를 놓을 것 이다. `state`(id들의 배열)와 `'TOGGLE_TODO_SUCCESS'`액션을 받는다.

`toggledId`로 값을 설정한 `result`와 정규화된 응답으로 부터 가져온 `entities`로 응답을 구조화 한다. 다음으로, `toggleId`에 의해 참조된 `entities.todos`에서 구한 `todo`로 부터 `completed` 값을 읽을 것 이다.

리스트에서 지우고 싶은 todo에는 두가지가 있다:
 * `completed` 필드가 `true` 지만 `filter`는 `active`
 * `completed` 는 `false` 지만 `filter`는 `completed` 

`shouldRemove`가 `true`일 때, 방금 토글된 todo의 id를 포함하지 않은 리스트의 복사본을 반환 하고 싶다. 리스트를 id로 필터링을 해서 다른 id를 가진 것은 남겨 둠으로서 완료 한다. 그렇지 않으면 원래 배열을 반환 한다.

##### Completed `handleToggle` Function
```javascript
const handleToggle = (state, action) => {
  const { result: toggledId, entities } = action.response;
  const { completed } = entities.todos[toggledId];
  const shouldRemove = (
    (completed && filter === 'active') ||
    (!completed && filter === 'completed')
  );
  return shouldRemove ?
    state.filter(id => id !== toggledId) :
    state;
};
```

[Demonstration and recap at 3:20 in video](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)
