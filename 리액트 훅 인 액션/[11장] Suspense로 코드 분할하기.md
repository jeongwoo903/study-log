# [11장] Suspense로 코드 분할하기

## 코드 분할이 필요한 이유
앱의 사용자는 다른 컴포넌트에 비해 특정 컴퐆넌트와 더 자주 상호작용한다.  
예를 들어, 페이지에 접속하자마자 보이는 컴포넌트가 있는 반면, 이벤트를 통해서 드러나게 되는 컴포넌트도 있을 것이다.  

브라우저에서 앱의 모든 코드를 한꺼번에 적재하지 않고 필요할 때 덩어리(chunk)로 코드를 적재하는 코드 분할(code splitting) 기술을 사용하면 적재할 코드의 양을 관리할 수 있다.

지금까지 책에서는 예제에서 **정적 import**를 써왔다.  
> ### 정적 import
> 자바스크립트 파일의 맨 위에는 현재 파일에서 사용하는 외부 파일 의존 관계를 지정하는 import 문이 있다.  
> 빌드시 웹팩과 같은 번들러는 코드를 검사하고 가져올 파일의 경로를 따라가서, 앱이 실제로 사용하는 모든 코드를 포함하는 파일인 번들을 생성한다.

이런 트리 셰이킹 과정은 중복 코드를 피하고 사용하지 않는 코드를 제거하기 위한인데, 번들을 잘 조직화 하고 가능한 작게 유지하는 데 도움이 될 수 있다.
> ### 트리 셰이킹
> 트리 셰이킹은 JavaScript 맥락에서 사용하지 않는 코드를 제거하는 방식이다.
> import 및 export 문을 사용하여, JavaScript 파일에서 사용하기 위한 코드 모듈을 내보내고 가져오는 것을 감지한다.

하지만 앱의 일부가 사용되지 않을 가능성이 높거나 아주 무거운 컴포넌트를 포함하는 경우, 초기 번들의 크기를 줄이고 사용자가 특정 경로를 방문하거나 특정 상호작용을 시작할때만 추가 번들을 적재하는 쪽이 더 유용할 수 있다.

## import 함수를 통해 동적으로 코드 임포트 하기
```js
// helloModule.js

export default function sayMessage(id, msg) {
  document.getElementById(id).innerHTML = msg;
}

export function sayHi(id) {
  sayMessage(id, "Hi");
}
```

#### 정적 import 방식
```js
import showMessage, { sayHi } from "./helloModule";

function handleClick() {
  showMessage("messagePara", "Hello World!");
  sayHi("hiPara");
}

document.getElementById("btnMessages").addEventListener("click", handleClick);

```

#### 동적 import 방식
```js
function handleClick() {
  import("./helloModule").then((module) => {
    module.default("messagePara", "Hello World!");
    module.sayHi("hiPara");
  });
}

document.getElementById("btnMessages").addEventListener("click", handleClick);
```

굳이 사용하지 않은 큰 파일을 적재할 필요가 없으므로, 버튼이 클릭된 경우에만 모듈을 적재한다. handleClick 함수는 import 함수를 사용해 모듈을 적재한다.

### 동적 import 분석하기
```js
import("./helloModule")
```
import 함수는 익스포트한 모듈로 해소되는 프로미스를 반환한다.
모듈이 적재된 다음에 작업을 수행하기 위해 프로미스의 then 메서드를 호출한다.
```js
import("./helloModule").then((module) => { /* module을 사용함. */})
```
then 대신 async/await 구문을 사용할 수 있다.

```js
async function handleClick() {
  const module = await import("./helloModule");
  
  module.default("messagePara", "Hello World!");
  module.sayHi("hiPara");
}

// 혹은 구조분해 할당을 통해
async function handleClick() {
  const {default: showMessage, sayHi} = await import("./helloModule");

  showMessage("messagePara", "Hello World!");
  sayHi("hiPara");
}
```

여기까지는 동적 import에 대한 소개다.
다음 부터는 아래에 대해 알아볼 것이다.
1. 컴포넌트를 동적으로 import 하는법
2. 컴포넌트 import를 지연시키는법
3. 리액트의 렌더링 과정을 망치지 않고 컴포넌트를 동적으로 임포트 하는 방법

## lazy와 Suspense를 사용해 컴포넌트를 동적으로 임포트 하기
리액트를 사용할 때 **우리는 상태 갱신에만 집중**하고 **리액트가 DOM을 관리**하게 해야 한다.

리액트를 사용할 때 어떻게 컴포넌트의 지연 적재를 리액트의 렌더링 과정과 결합할 수 있을까?  
이를 위해, 렌더링할 컴포넌트가 아직 준비되지 않았을 때 리액트가 무엇을 해야 할지를 선언적으로 알려줄 방법이 필요하다.  

### Lazy 함수를 사용해 컴포넌트를 지연 컴포넌트로 변환하기
만약 자주 사용하지 않는 "달력 보기" 버튼을 클릭한 경우에만 달력 코드를 적재하고 싶다고 해보자.

우선 아래 그림에서는 달력을 열수 있는 두 가지 방법을 대략적으로 보여준다.
1. 한 버튼은 달력을 해당 영역 내에서 열림.
2. 한 버튼은 현재의 뷰(전체 페이지)를 대체함.

| ![01.jpeg](assets/11%EC%9E%A5/01.jpeg) |
|:--------------------------------------:|
|            위 상황에 대한 대략적인 그림            |

```js
// 페이지 전반적인 코드
<div className="App">
  <main>Main App</main>
  <aside>
    <CalendarWrapper />
    <CalendarWrapper />
  </aside>
</div>
```

```js
// 달력을 표시하는 버튼이 있는 컴포넌트
function CalendarWrapper() {
  const [isOn, setIsOn] = useState(false);
  return isOn ? (
    <LazyCalendar /> // 지연 적재 컴포넌트를 포함함.
  ) : (
    <div>
      <button onClick={() => setIsOn(true)}>Show Calendar</button>
    </div>
  );
}
```
LazyCalendar 컴포넌트는 처음 렌더링이 이뤄지기 전까지 임포트가 이뤄지지 않는 특별한 컴포넌트이다.  
> 왜 바로 임포트가 이뤄지지 않을까.  
> 
> 그 이유는 동적 임포트와 리액트 lazy 함수를 조합해서 Calendar를 LazyCalendar로 변환했기 때문이다. 
> 
> 조건부 렌더링 만으로는 렌더링이 안되게 할 순 있으나 초기에 페이지에 필요한 컴포넌트들을 import 할때는 반영된다.
> ```js
> const LazyCalendar = lazy(() => import("./Calendar.js"));
> ```

LazyCalendar를 위해서는 Calendar 컴포넌트가 기본적으로 있어야 하니 Calendar 모듈을 만들어 보자.
```js
// 가짜 모듈을 만들고 컴포넌트를 지연 컴포넌트로 만들기
const moudle = {
  default: () => <div>Big Calendar</div> // default 프로퍼티에 함수 컴포넌트를 전달
};

function getPromise() {
  return new Promise(
    (resolve) => setTimeout(() => resolve(module), 3000)
  );
}

const LazyCalendar = lazy(getPromise);
```
이제 최초의 지연 컴포넌트를 시도할 모든 요소가 갖춰졌다.
> - 거대한 달력 컴포넌트(() => <div>Big Calendar</div>)
> - 달력 컴포넌트를 default 프로퍼티로 할당한 모듈
> - 모듈로 해소되는 프로미스(3초 후 해소됨)
> - 프로미스를 생성하고 반환하는 함수 getPromise
> - getPromise를 lazy에 전달해 생성된 지연 컴포넌트 LazyCalendar
> - 사용자가 버튼을 클릭한 경우에만 LazyCalendar를 표시하는 래퍼 컴포넌트 CalendarWrapper

```js
// 코드 전문
import React, { lazy, useState } from "react";
import "./styles.css";

const module = {
  default: () => <div>Big Calendar</div>
};

function getPromise() {
  return new Promise((resolve) => setTimeout(() => resolve(module), 3000));
}

const LazyCalendar = lazy(getPromise);

function CalendarWrapper() {
  const [isOn, setIsOn] = useState(false);
  return isOn ? (
    <LazyCalendar />
  ) : (
    <div>
      <button onClick={() => setIsOn(true)}>Show Calendar</button>
    </div>
  );
}

export default function App() {
  return (
    <div className="App">
      <main>Main App</main>
      <aside>
        <CalendarWrapper />
        <CalendarWrapper />
      </aside>
    </div>
  );
}
```

해당 코드를 실행하면 다음과 같은 에러가 발생한다.
> A React component suspended while rendering, but no fallback UI was specified.
>
> Add a <Suspense fallback=...> component higher in the tree to provide a loading indicator or placeholder to display.

### Suspense 컴포넌트를 사용해 폴백 콘텐츠 지정하기

위의 에러는 Suspense 라는 컴포넌트로 지연 컴포넌트를 감싸주지 않았기 때문에 발생하는 에러이다.  

`Suspense` 컴포넌트는 자신의 모든 자손 컴포넌트가 제대로 UI를 반환할 때까지 fallback UI를 보여준다.
```js
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```
이 Suspense 컴포넌트는 자식 컴포넌트 대신 폴백 UI를 렌더링한다.  
만약 Suspense 컴포넌트가 없으면 아까전의 에러를 반환한다.

### 지연 적재와 Suspense가 어떻게 함께 동작하는지 이해하기
지연 컴포넌트는 4가지의 내부 상태를 갖는 것으로 생각할 수 있다.
> 1. 초기화되지 않음(uninitialized)
> 2. 대기 중(pending)
> 3. 해소됨(resolved)
> 4. 거부됨(rejected)

아래 예시를 통해서 알아보자.

```js
const getPromise = () => import("./Calendar");
const LazyCalendar = lazy(getPromise);
```
프로미스는 컴포넌트를 default 프로퍼티로 제공하는 모듈로 resolve 되어야만 한다.  
resolve 되고 나면 리액트는 지연 컴포넌트의 상태를 resolved로 설정하고 렌더링할 준비가 된 컴포넌트를 반환한다.  

이 단계는 다음과 비슷하게 이뤄진다.
```js
if(status === "resolved") {
  return component;
} else {
  throw promise;
}
```
else 절에는 상위 컴포넌트의 Suspense와 통신하는 핵심 코드가 들어 있다.  
즉 프로미스가 해소되지 않으면 리액트가 프로미스를 마치 예외처럼 throw 한다.  
Suspense 컴포넌트는 던져진 프로미스를 catch 하면서 폴백 UI를 렌더링 하게 되어있다.

| ![02.jpeg](assets/11%EC%9E%A5/02.jpeg) |
|:--------------------------------------:|
|                 동작 과정                  |

Suspense는 오류에 대한 UI는 처리하지 않는다.
오류 처리는 ErrorBoundary의 역할이다.

## 오류 경계를 사용해 오류 잡아내기

리액트는 자식 컴포넌트에서 발생하는 오류를 잡아내는 컴포넌트를 제공하지 않는다.  

하지만 클래스 컴포넌트가 오류를 잡아서 보고하고 싶을때 구현할 수 있는 몇가지 생명 주기 메서드를 제공한다.

> Error Bundary (에러 경계)
> 하위 컴포넌트 트리의 어디에서든 자바스크립트 에러를 기록하며 깨진 컴포넌트 트리 대신 폴백 UI를 보여주는 컴포넌트 이다.

```js
// reactjs.org의 ErrorBoundary 컴포넌트

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 다음 랜더링 시 폴백 UI를 보여주도록 상태를 갱신함
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 오류 로그를 오류 보고 서비스에 전달할 수 있음
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 원하는 커스텀 폴백 UI를 랜더링할 수 있음
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

자식 컴포넌트를 렌더링 하는 동안 오류가 발생할 때 리액가 렌더링할 폴백UI를 지정하기 위해 오류 경계 컴포넌트를 사용함. 
```js
<ErrorBoundary>
  {/*앱이나 하위 트리*/}
</ErrorBoundary>
```

커스텀화 한 오류 경계를 만들어 폴백 UI와 오류 복구 전략을 맞춤형으로 제공한다.
