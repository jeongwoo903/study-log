# [JS] 몰랐던 문법들 정리
## 구조 분해 할당(destructuring assignment)
이는 객체의 값을 편하게 꺼내쓰기 위한 방법이다.

하나의 사례로, react에서 props를 그냥 전달 받으면
```typescript jsx
function Component(props) {
    return (
        <div>
            <p>Name: {props.name}</p>
            <p>Age: {props.age}</p>
        </div>
    );
}
```
위의 코드처럼 `props.name`과 같이 명시적으로 접근해야 한다.
하지만, 아래 코드처럼 중괄호를 통해 분해해서 받으면 각각의 변수로 바로 사용할 수 있어서 편하다.
```typescript jsx
function Component({ name, age }) {
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
    </div>
  );
}
```
## nullish 병합 연산자 `??`

nullish 병합 연산자(nullish coalescing operator) `??`를 사용하면, 여러 피연산자 중 그 값이 ‘확정되어있는’ 변수를 찾을 수 있다.

아래 예시를 보면 `firstName`과 `lastName`의 값이 null이라서 생략되고 "바이올렛" 값이 출력되는 것을 볼 수 있다.
이처럼 짧은 문법으로 값이 있고 없음을 쉽게 판단해 값을 결정할 수 있다.
```js
let firstName = null;
let lastName = null;
let nickName = "바이올렛";

alert(firstName ?? lastName ?? nickName ?? "익명의 사용자"); // 바이올렛
```
아래 코드와 같이 메서드 유무에 따라 Layout의 적용을 결정 할 수도 있다.
```js
const getLayout = Component.getLayout ?? ((page: ReactNode) => page)
```