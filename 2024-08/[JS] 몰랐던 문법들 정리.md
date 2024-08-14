# [JS] 몰랐던 문법들 정리
## 구조 분해 할당(destructuring assignment)
이는 객체의 값을 편하게 꺼내쓰기 위한 방법이다.

하나의 사례로, react에서 props를 그냥 전달 받으면
```typescript typescript jsx
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

