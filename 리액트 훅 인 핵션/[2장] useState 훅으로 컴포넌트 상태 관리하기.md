# [2장] useState 훅으로 컴포넌트 상태 관리하기

## 리액트의 상태와 렌더링

리액트는 **현재 상태**를 활용해 **UI를 렌더링** 해야한다.  
리액트는 **상태**와 **UI**가 동기화 되도록 보장한다.

상태는 **전역, 컴포넌트 간, 컴포넌트 내부**에서 관리 될 수 있다.

함수 컴포넌트 내에서 내부 변수를 그냥 변경하는 것 만으로는 리액트는 그 사실을 알 수 없다.

컴포넌트 호출과 호출 사이에 상태를 유지하고 컴포넌트의 상태 변경을 리액트에 알리는 가장 간단한 방법은 **useState** 훅이다.

## useState

```jsx
const [value, setValue] = useState();
```

위와 같은 형태로 알려져있다. 두번째 인자인 갱신 함수(updater 함수)는 set으로 시작하는 것이 일반적이다.  
아래 같은 형태도 가능은 하다.

```jsx
const valueState = useState();
const value = valueState[0];
const setValue = valueState[1];
```

초기 값으로 함수를 받을 수도 있는데, 이 경우 컴포넌트 렌더링 시마다 함수가 실행된다.
하지만 [Lazy Initialization(게으른 초기화)](https://github.com/jeongwoo903/study-log/blob/main/2024-12/%5BReact%5D%20Lazy%20Initial%20State%20(%EC%A7%80%EC%97%B0%20%EA%B3%84%EC%82%B0%20%EC%B4%88%EA%B8%B0%20%EC%83%81%ED%83%9C).md#react-lazy-initialization-%EA%B2%8C%EC%9C%BC%EB%A5%B8-%EC%B4%88%EA%B8%B0%ED%99%94)
를 이용하면 초기 상태만 함수를 이용하는 방법도 가능하다.

갱신 함수는 현재 값의 상태를 반환하게 할 수도 있는데 이를 통해서 누산기를 만들수도 있다.

```jsx
const [count, setCount] = useState(0);

function asd () {
  // 1. setCount(count => {return count + 1});
  // 2. setCount(count + 1);
}
```

두 방식에 대해서 클로드 한테 물어보니 다음과 같은 답을 받았다.

| ![01.png](assets/2%EC%9E%A5/01.png) |
|:-----------------------------------:|
|          갱신 함수의 사용 방식의 차이           |

## 갱신 함수를 호출하면 이전 값이 치환된다

**클래스 컴포넌트**의 경우에는 this.setState를 통해 값이 변경 되었는데 이때는 값이 병합되는 방식이였다.  
하지만 **함수 컴포넌트**의 경우에는 갱신함수를 사용할 경우 값이 치환되기 때문에, `setState({...state, value: value})` 이런식의 spread
연산자를 통해 새 상태로 값을 복사해야 한다.

| ![02.jpeg](assets/2%EC%9E%A5/02.jpeg) | ![03.jpeg](assets/2%EC%9E%A5/03.jpeg) |
|:-------------------------------------:|:-------------------------------------:|
|             클래스 컴포넌트의 경우              |              함수 컴포넌트의 경우              |

## 여러번 호출 가능하다
상태 값이 여럿인 경우, useState를 여러 번 호출할 수 있다.  
리액트는 호출 순서를 사용해 값을 일관성 있게 대입하고 갱신 함수를 올바른 변수에 대입해 준다.