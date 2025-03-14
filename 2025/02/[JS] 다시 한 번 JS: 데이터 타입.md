# [JS] 다시 한 번 JS: 데이터 타입

## 데이터 타입

js는 Primitive Type(원시 타입)과 Reference Type(참조 타입) 두 가지의 타입으로 나뉜다.  

원시 타입
number, string, boolean, undefined, null, symbol

참조 타입
Array, Function, RegExp, Set, Map 등등..

### JS 메모리 구조

자바스크립트 메모리 구조는 Stack Memory와 Heap Memory로 나뉘어져 있다.

- Stack Memory: 원시 타입 값과 실행 컨텍스트의 선언 순서를 저장한다.
- Heap Memory: 동적인 크기의 참조 타입 값들을 저장한다.

### 원시 타입의 데이터 저장 방법

우리가 변수를 아래와 같이 선언한다면 컴퓨터는 선언과 할당 두 부분으로 나뉘어 생각한다.  
(이는 차후 호이스팅과 연관이 있다.)
```js
// 우리가 작성한 방식
let a = 1;

// 컴퓨터가 이해하는 방식
let a;
a = 1;
```

그렇기 때문에 `let a = 1`을 작성하면, 
1. 우선 스택에 a에 대한 공간을 마련한다. 그리고 공간의 이름을 a로 지정한다.
2. 그리고 1이라는 값을 새로운 공간에 저장해두고 1의 주소를 a 공간에 값으로 지정해준다.

이후 만약 `a = 2`라고 할당하게 된다면,
1. a라는 공간을 찾는다.
2. 2라는 값이 선언되었는지 알아보고 없다면 2를 새로운 공간에 저장한다.
3. a공간의 값을 2의 주소값으로 지정해준다.

이런 프로세스가 진행된다.  

| ![01.png](assets/%5BJS%5D%20%EB%8B%A4%EC%8B%9C%20%ED%95%9C%20%EB%B2%88%20JS%3A%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%ED%83%80%EC%9E%85/01.png) |
|:-----------------------------------------------------------------------------------------------------------------------------------------:|
|                                                                   강의 사례                                                                   |

### 참조 타입의 데이터 저장 방식

참조 타입은 데이터가 가변적이다.
이 때문에 const로 정의를 하더라도 내부의 값들은 const의 영향을 받지 못한다.

| ![03.png](assets/%5BJS%5D%20%EB%8B%A4%EC%8B%9C%20%ED%95%9C%20%EB%B2%88%20JS%3A%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%ED%83%80%EC%9E%85/03.png) |
|:-----------------------------------------------------------------------------------------------------------------------------------------:|
|                                                     const로 정의 했음에도 내부 데이터는 보호 받지 못함.                                                      |

이를 해결하기 위해선, Object.freeze와 같은 매서드를 사용해야 하는데, 이런 참조 타입의 특성에 대해 알아보자.

다음과 같이 선언과 할당을 했다고 해보자.
```js
let obj = {
  a: 1,
  b: 'bbb',
}
```

그럼 다음과 같은 일들이 일어난다.
1. 데이터 공간을 마련한 다음 이름을 obj로 지정한다.
2. 데이터 공간에는 하나의 값만 넣을 수 있기 때문에 객체를 통째로 넣지 못한다.
3. 데이터 공간은 힙 메모리에 공간을 많이 마련해두고 각 key에 해당하는 공간을 마련해 이름을 붙여준다.
4. 각각의 값들은 콜 스택 공간에 따로 저장해 key의 이름이 저장된 곳의 값으로 주소가 등록된다.

텍스트로 설명하니 복잡한데 그림으로 나타내면 다음과 같다.

| ![02.png](assets/%5BJS%5D%20%EB%8B%A4%EC%8B%9C%20%ED%95%9C%20%EB%B2%88%20JS%3A%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%ED%83%80%EC%9E%85/02.png) |
|:-----------------------------------------------------------------------------------------------------------------------------------------:|
|                                                     const로 정의 했음에도 내부 데이터는 보호 받지 못함.                                                      |

위 그림에서 obj.a의 값을 2라고 할당을 해도 obj의 참조 값은 바뀌지 않는 모습을 볼 수 있다.  
그러면 아래와 같은 상황이 일어나면 어떻게 될지 예상해보자.  
```js
let obj = {
  a: 1,
  b: 'bbb',
}

let obj2 = obj;

obj2.a = 2;

console.log(obj);
console.log(obj2);
```
결과는 아래 이미지와 같다.

| ![04.png](assets/%5BJS%5D%20%EB%8B%A4%EC%8B%9C%20%ED%95%9C%20%EB%B2%88%20JS%3A%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%ED%83%80%EC%9E%85/04.png) |
|:-----------------------------------------------------------------------------------------------------------------------------------------:|
|                                                                    결과                                                                     |

왜 obj2의 값을 바뀌었는데 obj의 값도 바뀌었나면, 두 객체가 저장된 데이터 공간의 참조값은 변하지 않았는데 내부의 값은 바뀌었기 때문이다.

| ![05.png](assets/%5BJS%5D%20%EB%8B%A4%EC%8B%9C%20%ED%95%9C%20%EB%B2%88%20JS%3A%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%ED%83%80%EC%9E%85/05.png) |
|:-----------------------------------------------------------------------------------------------------------------------------------------:|
|                                                                   참고 사진                                                                   |

위 그림에서 처험 obj2.c의 값이 바뀐다면 7103의 값의 주소는 바뀌지만, obj1과 obj2는 여전히 5003을 가리키고 있기 때문에 두 변수가 값이 동시에 바뀐다.

### 왜 원시 값을 그대로 넣지 않을까?

왜 원시 값을 데이터의 값으로 바로 넣지 않고 참조 값을 부여할까?  
이 질문의 답은 다음과 같다.  

1. 간단한 데이터의 경우 문제가 되지 않지만, 큰 용량을 차지하는 데이터의 경우 용량이 낭비된다.
2. 차후 값에 대한 비교 연산을 할 경우 보다 간단해진다.  