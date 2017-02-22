# Building React Applications with Idiomatic Redux (Egghead.io) [한국어판]

![](https://s3.amazonaws.com/f.cl.ly/items/212E0u153X2A18131808/Image%202016-07-10%20at%2012.00.28%20PM.png?v=feaddbc8)

이 저장소는 Egghead.io에 있는 [Dan Abramov의](https://github.com/gaearon) 두번째 [Redux 강좌](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)에 있는 노트를 모아 놓은 곳 이다.

## 01\. Arrow 함수를 이용해 단순화 하기
ES6 에서 제공하는 특성을 이용해 arrow 함수가 어떻게 더 깔끔해 지는지 볼 수 있다. [Video](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions)


## 02. 초기 상태 제공하기
이전에 저장된 상태로 Redux앱을 시작하는 방법을 배울 것 이다. [Video](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)


## 03. 내부 저장소에 상태 저장 해 놓기
store.subscribe()를 이용해 앱의 몇가지 상태를 효율적으로 내부 저장소에 저장하고 리프레시 후에 복구 하는 방법을 배울 것 이다. [Video](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage#/tab-transcript)


## 04. 진입점 리펙토링
라우터를 추가 하기 위해 진입점에 있는 코드를 어떻게 더 잘 분리 하는지 배울 것 이다. [Video](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux#/tab-transcript)


## 05. React Router를 프로젝트에 추가 하기
Redux 프로젝트에 React Router를 추가 하고 root 컴포넌트를 랜더 하는 방법을 배울 것 이다. [Video](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux#/tab-transcript)


## 06. React Router `<Link>`를 사용해 이동하기
React Router에서 제공하는 컴포넌트를 이용해 주소창을 변경하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-navigating-with-react-router-link?series=building-react-applications-with-idiomatic-redux)


## 07. React Router Params로 Redux 상태 필터링 하기
React Router를 추가 해서 책임의 균형을 이동 시키고, 컴포넌트들이 어떻게 동시에 둘을 사용하는지 배우게 될 것이다.
[Video](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)


## 08. `withRouter()`를 사용해서 연결된 컴포넌트들에 Params를 주입하기
React Router에서 제공하는 `withRouter()`를 사용해서 prop으로 모든 경로에 있는 컴포넌트들에 전달하지 않고, 트리 깊이 있는 연결된 컴포넌트들에 params를 주입하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components)


## 09. `mapDispatchToProps()` 간결한 형식 사용하기
액션 생성자가 인자가 컬백 prop 인자와 일치 하는 일반적인 경우 `mapDispatchToProps()`에 있는 보일러플레이트 코드를 피하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)


## 10. Selector와 리듀서 같은 곳에 놓기
리듀서 파일에 있는 상태 모습에 관한 내용을 숨겨서, 컴포넌트들이 의존할 필요 없게 하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux#/tab-transcript)


## 11. 상태 모습 정규화
실제 어플리케이션에서 중요한 데이터 일관성을 보장하기 위해 상태 모습을 정규화하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)


## 12. 액션들을 로그로 기록하기 위해 `dispatch()`를 감싸기
액션에 따른 변경 사항을 콘솔에 로그로 기록할 수 있게 해주는 Redux에서 제공하는 중앙 갱신 방법을 배울 것 이다. 
[Video](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)


## 13. 프로젝트에 가짜 백앤드 추가 하기
데이터 가져오기를 시뮬레이트 할 수 있는 다음 강좌에서 사용되는 백앤드 모듈에 관해 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-adding-a-fake-backend-to-the-project)


## 14. 경로 변경에 따라서 데이터 가져오기 
경로 변경시 비동기 요청을 하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)


## 15. 데이터를 가져오는 액션 발행하기
데이터를 가져온 후 Redux 액션을 발행하는 방법과 경로가 변경 됐을 때 동작하는 방법을 되풀이하는 것을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)


## 16. Promise들을 인식 하게 만들기 위해 `dispath()` 감싸기
`dispatch()`가 Promise를 인식하게 하게 해서 액션 생성자에 비동기적으로 컴포넌트들의 비동기 로직을 옮길 수 있게 하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)


## 17. 미들웨어 체인
서로 다른 목적을 위해 `dispatch()`를 일반적으로 감싸서 Redux 에코시스템에 널리 사용가능한 "미들웨어"라고 불리는 컨벱으로 만드는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-the-middleware-chain)


## 18. Redux 미들웨어 적용하기
작성한 미들웨어를 대체 해서 기존 코어와 서드파티 유틸리티를 함께 붙여서 사용할수 있는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)


## 19. 가져온 데이터로 상태 갱신 하기
데이터의 저장소를 서버로 이동 하는 것이 앱의 상태 모양과 리듀서를 어떻게 변경 하는 지 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data)

## 20. 리듀서들 리팩토링 하기
어떻게 리듀서 파일들에 있는 중복을 제거 하고, 새롭게 추출된 리듀서와 같이 있는 상태 모습에 관한 내용을 유지 하는지 배울 것 이다. 
[Video](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)


## 21. 로딩 인디케이터 표시 하기
데이터를 가져오는 동안 로딩 인디테이터를 표시하는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

## 22. Thunk를 이용해 액션을 비동기적으로 발행하기
Redux에서 가장 일반적으로 비동기 액션을 생성하는 "thunk"에 대해 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)

## 23. Thunk 사용중 경합 상황 피하기
어떻게 Redux 미들웨어가 불필요한 네트웍 요청을 피하고 경합 상황이 생길 가능성을 배제 하도록 액션을 조건적으로 발행 할 수 있게 하는지 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)

## 24. 에러 메세지 표시하기
비동기 액션의 에러를 다루고, UI에 표시하고, 사용자에게 다시 시도할 기회를 주는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-displaying-error-messages)

## 25. 서버에 데이터 생성하기
아이템이 서버에 생성되서, 지역 상태에 맞게 갱신되기 까지 기다리는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)

## 26. `normalizr`로 API 응답 정규화 하기
모든 API 응답을 정규화된 형식으로 변경해서 리듀서를 단순하게 만드는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)

## 27. 서버에서 데이터 갱신하기
서버에 있는 아이템이 갱신되고나서, 지역 상태에 맞게 갱신되는 것을 기다리는 방법을 배울 것 이다.
[Video](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)
