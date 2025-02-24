# [JS] 다시 한 번 JS: this

this 바인딩은 실행 컨텍스트가 실행될 때 한다.  
this가 함수가 호출 될 때 비로소 결정된다.  

this는 바로 예측 가능한게 아니라 어떤 식으로 호출했는 가에 따라서 달라질 수 있다.  
> [[매일메일] 상황에 따라 this 바인딩이 어떻게 이뤄지는지 설명해주세요.](https://www.maeil-mail.kr/question/160)

this가 상황에 달라지는 경우는 5가지가 있다.

1. 전역 함수에서 호출 되었을때
2. 함수에서 호출 되었을때
3. 메서드에서 호출 되었을때 
4. callback에서 호출 되었을때
5. 생성자함수에서 호출 되었을때

## 전역 함수에서 호출 되었을때
전역 공간에서의 this는 전역 객체를 가리킨다.  
브라우저 환경에서는 window를, node.js에서는 global을 가리킨다.  

## 함수에서 호출 되었을때
일반 함수로써 호출이 이루어 진다면 이는 항상 전역 객체(window/global)을 가리킨다.  
즉 a() 와 같은 형태로 호출 되었을 때를 말한다.   

아래의 예시들은 모두 window를 가리킨다.

```js
// case 1.
function a() {
  console.log(this);
}

a();

// case 2.
function a() {
  function b() {
    console.log(this);
  }
  b();
}

a();

// case 3.
let a = {
  b: function() {
    function c() {
      console.log(this);
    }
    c();
  }
}

a.b();
```

## 메서드에서 호출 되었을때 
객체에 매서드를 호출한다면 그 객체가 바로 this가 가리키는 대상이 된다.  
즉 `a.b()`라면, a가 대상이 된다. 왜냐면 b함수를 a함수의 매서드 라고 생각하기 때문이다.

```js
let a = {
  b: function() {
    console.log(this);
  }
}

a.b();
```
그럼 다음의 경우는 어떨까?

```js
let a = {
  b: {
    c: function() {
      console.log(this);
    }
  }
}

a.b.c();
```
이 경우에는 `a.b`가 this에 해당한다.

## callback에서 호출 되었을때
callback에서의 this는 지정하는 바에 따라서 달라져서 어렵다.
하지만 기본적으로는 함수에서 호출했을때와 동작방식이 같다.

> ### 명시적 바인딩
> 
> call(), apply(), bind() 메서드를 사용하면 this를 명시적으로 설정할 수 있다.
> 
> ```js
> function greet() {
>   console.log(this.name);
> }
> const user = { name: "Alice" };
> 
> greet.call(user); // "Alice"
> ```

다시 본문으로 돌아와서 아래 코드를 보자.

```js
let callback = function() {
  console.log(this);
}

let obj = {
  a: 1,
  b: function(cb) {
    cb();
  }
}

obj.b(callback);
```
이 경우엔 this는 전역을 가리킨다.  
다만, 아래 코드와 같이 인자를 받아 명시적으로 this를 바인딩 하면 해당 객체를 가리킨다.

```js
let callback = function() {
  console.log(this);
}

let obj = {
  a: 1,
  b: function(cb) {
    cb.call(this);
  }
}

obj.b(callback);
```

정리하자면 다음과 같다.
- 기본적으로 함수의 this와 같다.
- 제어권을 가진 함수가 콜백의 this를 지정해둔 경우도 있다.
- 이 경우에도 개발자가 this를 바인딩해서 콜백을 넘기면 그에 따른다.

## 생성자함수 호출시
new 키워드를 사용해 생성자함수로써 호출을 하면, this는 새로 만들 인스턴스 객체를 가리킨다.

```js
function Person(n,a) {
  this.name = n;
  this.age = a;
}

let person = new Person("장정우", 25);
console.log(person.name) // 장정우

```