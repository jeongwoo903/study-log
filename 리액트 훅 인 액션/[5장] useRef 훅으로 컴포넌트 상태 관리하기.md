# [5장] useRef 훅으로 컴포넌트 상태 관리하기

## UI를 갱신하지 않고 상태 변경하기
컴포넌트에 저장되는 변수의 대부분은 UI에 직접 표시된다.  
그러나 때로는 앱 매커니즘의 일부분으로만 변수를 사용할 때가 있다.

후자의 경우 유저에게 **값을 나타낼 필요가 없으므로 값이 달라져도 자동으로 재렌더링이 일어나지 말아야 한다.**

## useState와 useRef
### useState
useState에 대해서 잠깐 복습을 해보자.  

useState는 아래와 같은 형태를 띈다.
```jsx
const [state, setState] = useState(initValue);

/**
 * state: 값을 담을 변수
 * setState: 값을 변경하는 setter 함수
 * */
```
state는 처음 렌더링 될 때 초기값이 담기게 된다. 그리고 setter 함수를 통해 값이 바뀌게 되면 리렌더링이 일어난다.

### useRef
이런 정보를 가지고 useRef를 바라보자.  

useRef의 경우 다음과 같은 형태를 띈다.  
```jsx
const ref = useRef(initValue);

/**
 * ref 변수에는 current라는 프로퍼티가 있는 객체가 담기게 된다.
 * */
```

리액트가 처음 컴포넌트를 호출하게 되면, useRef 함수를 통해 전달받은 초기값을 참조객체의 current 프로퍼티에 대입한다.  

두 번째 렌더링 부터 리액트는 useRef 호출의 순서에 따라 동일한 참조객체를 변수에 대입한다.  
참조객체의 current 프로퍼티에 값을 대입하게 되면 상태 값을 영속화 할 수 있다.  

```jsx
const ref1 = useRef("apple");
console.log(ref1.current) // apple

ref1.current = "banana";
console.log(ref1.current) // banana
```
참조객체의 current 프로퍼티에 새 값을 대입해도 재렌더링이 발생하지 않는다.

## 엘리먼트 참조
리액트는 DOM 엘리먼트 참조를 자동으로 참조객체의 current 프로퍼티에 대입할 수 있다.  
JSX에서 ref 애트리뷰트에 참조객체 변수를 대입하면 된다.

```jsx
const myRef = useRef();
...
return (
  <button ref={myRef}>Click me!</button> // JSX ref 애트리뷰트의 값으로 참조객체를 지정
);
...
myRef.current;
```



