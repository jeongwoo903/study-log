# [3장] useReducer 훅을 이용해 컴포넌트 상태 관리하기

# Reducer
리듀서의 이름만 봐서는 그 의미를 알기 어렵다.  
> Re(act state Pro)ducer = 리액트 상태 제공자

리듀서는 위 숨겨진 뜻에서 알 수 있듯이 상태를 관리 하기 위한 함수다.  

리듀서는 **상태를 관리하는 모든 방법을 한 장소에 모아서** 주로 switch-case를 통해 type에 따라 관리한다.  
그래서 **한 액션이 여서 상태에 영향을 미쳐야 하는 경우** 상태 관리를 쉽게 할 수 있다.

## useReducer
useReducer는 리액트에 내장된 reducer 함수를 이용하기 위한 훅이다.

### 구조
```jsx
// 주로 switch-case를 통해 type에 따른 이벤트를 발생시킨다.
const reducer = (state, action) => {
  switch (action.type) {
    case 'ACTION_1':
    //...
    case 'ACTION_2':
    //...
    default:
    //...
  }
}

/**
 * 3번째 인자는 필수는 아니다.
 * 3번째 인자는 초기 상태를 lazy하게 생성할 때 사용된다.
 * - 초기 상태 계산을 지연시킬 수 있다. (lazy initialization)
 * - 복잡한 초기 상태 로직을 리듀서 밖으로 분리할 수 있다.
 * - 컴포넌트가 리렌더링되어도 초기화 함수는 한 번만 실행된다.
*/
const [state, dispatch] = useReducer(reducer, initArg, initFn);

// dispatch는 "보내다" 라는 뜻.
// dispatch 함수를 이용해 reducer에게 action을 dispatch 한다.
dispatch({ type: 'ACTION_1', payload: { ... } })
```

| ![01.jpeg](assets/3%EC%9E%A5/01.jpeg) |
|:-------------------------------------:|
|      useReducer의 이해를 위해 필요한 용어들       |

### 특징
- 순수 함수(동일한 입력 -> 동일한 출력)
  - 이 때문에 async 함수가 될 수 없다.
  - 즉, Promise 타입의 action도 받지 못한다.
- 비동기 로직을 포함하지 않는다.
- 상태를 직접 수정하지 않는다.
- Side Effect를 포함하지 않는다.


