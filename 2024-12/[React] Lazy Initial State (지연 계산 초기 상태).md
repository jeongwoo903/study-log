# [React] Lazy initialization (게으른 초기화)

# 서론
가끔 개발을 하다보면 어떤 복잡한 string에서 특정 string을 추출한 값을 이용해야 하는 경우가 있을 수 있다.
```jsx
function stringParser(complexStr) {
  // str 가공 로직
  return str
}

export default function TextComponent() {
  const [str, setStr] = useState(stringParser(complexStr));
  
  
}
```

이 경우 TextComponent를 실행할 때 마다 값비싼 stringParser를 수행해야한다.

React의 useState는 `useState(여기)`에 `useState(함수())`이런식으로 호출하는 방식으로 함수 return의 값으로 초기값을 받아온다면, 이 함수는 컴포넌트가 재렌더링 될 때 마다 실행이 된다.  

## 지연 계산 초기 상태(lazy initial state)
useState 훅은 함수를 인자로 받을 수 있다. 이런 인자를 **지연 계산 초기 상태(lazy initial state)** 라고 한다.
```jsx
function stringParser(complexStr) {
  // str 가공 로직
  return str
}

export default function TextComponent() {
  const [str, setStr] = useState(() => stringParser(complexStr));
  
  
}
```

다음과 같이 함수를 인자로 넘겨줄 경우, 리액트는 컴포넌트가 맨 처음 렌더링될 때 함수를 한 번만 실행한다.  
리액트는 함수가 반환한 값을 초기 상태로 사용하고 다시 렌더링 되었을 때는 더이상 함수를 실행하지 않는다.

정리하면 아래와 같다.  
> Lazy initailize는 useState를 사용하여 state를 초기화하는 과정을 lazy(게으르게)하게 실행하는 것이다.
> 1. setState()에 인수로 함수를 값으로 넘겨준다.
> 2. setState(함수)이 진행되면 최초 초기화 진행 과정에서 함수을 실행하게 된다.
> 3. 이후 업데이트(리렌더링 과정)에서 초기화가 진행되지 않기 때문에 함수을 실행하는 부분이 생략된다.
   이와 같이 lazy initialize는 초기값 계산에 많은 비용(연산 시간, 메모리 등)이 소요될 때 비효율적인 부분을 최적화하는데 사용할 수 있다.

