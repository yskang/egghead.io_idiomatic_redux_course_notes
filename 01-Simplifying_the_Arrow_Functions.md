# 01. Arrow 함수를 이용해 단순화 하기
[비디오 링크](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions?course=building-react-applications-with-idiomatic-redux)

액션 생성자들은 정규 자바스크립트 함수이므로, 원하는 대로 정의 할 수 있다.

예를 들어, 만일 화살표를 원치 않으면, 전통적인 함수 선언 문법을 사용해도 된다.

##### Arrow Function Syntax
``` javascript
export const addTodo = (text) => {
  return {
    type: 'ADD_TODO',
    id: (nextTodoId++).toString(),
    text,
  };
};
```
##### Traditional Function Syntax
``` javascript
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    id: (nextTodoId++).toString(),
    text,
  };
}
```

하지만, 만일 arrow 함수를 사용하기를 좋아 한다면, 좀 더 함축적이 될 것이다.

위에 arrow 함수를 사용한 것을 보면, 괄호로 시작되고 끝나고 괄호 안에 `return` 문을 포함하는 함수를 볼 수 있다. return 문은 함수안에 있는 모든 것 이기 때문에 그 자체를 arrow 함수의 몸체로 사용할 수 있다.

object 표현식을 사용해서, block 을 제거 할 수 있다.
```javascript
export const addTodo = (text) => ({
  type:'ADD_TODO',
  id: (nextTodoId++).toString(),
  text,
})
```

*노트:* 괄호로 표현식을 감싸서, 파서가 블럭이 아닌 표현식으로 알게 하는 것이 중요하다.

이런 단계들은 단지 object를 반환하기만 하는 함수들에 대해 반복 가능하다; 단지 `return` 문을 제거하고, 몸체를 표현식으로 바꾸면 된다.

이 단계들은 액션 생성자들에도 역시 적용될 수 있다. 예를 들어, `mapStateToProps` 와 `mapDispatchToProps` 는 단지 object를 반환 하기만 하는 것이 일반적이다.
##### 이전:
```javascript
const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}
```

##### 이후:
```javascript
const mapStateToProps = (state, ownProps) => ({
    active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
})
```

object 안에 함수가 선언되어 있는 경우, arraow 함수를 ES6에서 지원하는 단축 method 선언 방식을 이용해서 `mapDispatchToProps`함수를 더욱 작게 만들 수 있다. 

```javascript
const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick() {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
})
```

#### [Recap at 2:05 in video](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions?course=building-react-applications-with-idiomatic-redux)
