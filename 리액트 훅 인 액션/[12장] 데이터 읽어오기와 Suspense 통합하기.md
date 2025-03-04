# [12장] 데이터 읽어오기와 Suspense 통합하기

> ### **이전 장 정리**  
> 지연 적재 컴포넌트(lazy load cmoponent)를 렌더링할 때, 리액트는 먼저 컴포넌트의 상태를 확인한다.  
> 만약 동적으로 임포트한 컴포넌트가 적재됐다면 리액트는 렌더링을 계속하고, 적재 중인 경우에는 동적 임포트에 대한 프로미스를 던진다.  
> 프로미스가 거부되면 그 오류를 잡아내서 적절한 폴백 UI를 보여줄 오류 경계가 필요하다.  

```js
const LazyCalendar = lazy(() => import("./Calendar"));
```

```jsx
<ErrorBoundary fallback="error occur">
  <Suspense fallback="Loading...">
    <lazyCalendar />
  </Suspense>
</ErrorBoundary>
```

위 구조가 로딩과 에러 예외 처리에 대한 가장 일반적인 구조이다.

---

## 데이터 읽어오기와 Suspense

여기서 부터가 이번 장의 시작이고 이번 장의 목적은 **데이터 불러오기** 이다.  

우리는 지연 적재 컴포넌트 말고도 서버에서 데이터를 적재하는 컴포넌트에서도 비슷한 동작이 일어나길 바란다.  

하지만, 지연 컴포넌트에 사용할 lazy 함수는 있지만 데이터를 적재하는 컴포넌트에 사용할 안정적인 리액트 내장 매커니즘은 없다.  
(react-cache 패키지가 있지만 실험적이고 불안정 하다고 함.)

그래서 데이터를 가져오려면 어떤 매커니즘으로 데이터를 적재하는 방식에 맞게 프로미스나 오류를 던지는 방법을 생각해 낼 수 있을 것인지 알아보자.  

```jsx
<ErrorBoundary fallback="Oops!">
  <Suspense fallback="Loading message...">
    <Message>
  </Suspense>
</ErrorBoundary>
```
다음과 같은 코드가 있다고 해보자.

이때 Message의 들어갈 텍스트는 비동기 통신을 통해 받아온다고 생각하자. Message 컴포넌트의 코드는 다음과 같다.
```jsx
function Message() {
  const data = getMessageOrThrow(); // 데이터를 반환하거나 프로미스나 예외를 던지는 함수를 호출함.
  return <p className="message">{data.message}</p>; // UI에 데이터를 포함시킴.
}
```
우리는 getMessageOrThrow 함수가 데이터가 존재할 때 데이터를 반환하기를 원한다. 

하지만 데이터를 위한 프로미스가 있는 경우(예: 브라우저의 fetch API가 반환하는 프로미스), 해당 프로미스의 상태를 확인할 수 있는 방법이 없다.

그래서 우리는 프로미스의 상태(pending, fulfilled, rejected)를 확인 할 수 있도록 상태를 보고하기 위한 코드로 감싸야 한다.
> 이 말이 뭐냐.
> 
> 리액트의 렌더링은 동기적으로 이루어 진다. 즉, 렌더링 함수는 즉시 결과를 반환해야 한다.  
> 
> getMessageOrThrow 함수가 실행되면, Promise 객체가 반환 되긴 하지만, resolve 되지 않아 message 속성이 없다.  
> 
> 이 시점에서 data는 실제 데이터가 아닌 Promise 객체이기 때문이다.
> 
> 즉, "프로미스의 상태를 확인할 수 있는 방법이 없다"는 것은, 동기적인 렌더링 과정에서 비동기 작업의 상태(pending, fulfilled, rejected)를 적절히 처리할 수 없다는 의미이다.

## 프로미스가 상태를 포함하도록 업그레이드 하기

Suspense와 ErrorBoundary를 사용하면, 프로미스의 상태를 사용해 동작을 결정 할 필요가 있다.  

|   프로미스의 상태    |        동작         |
|:-------------:|:-----------------:
| 일시중단(pending) |     프로미스를 던짐      |
| 해소됨(resolved) | 해소된 값, 즉 데이터를 반환함 |
| 거부됨(rejected) |     거부 오류를 던짐     |

프로미스가 자신의 상태를 보고하지 못하는 상황이면, 아래 checkStatus 같이 프로미스의 현재 상태를 반환하면서 프로미스가 해소되거나 거부된 경우 오류 값이나 해소된 값을 반환하는 함수를 원한다.  

```js
const {promise, status, error} = checkStatus();
```

이제 우리는 `if(status === "pending")`과 같은 조건문을 통해 프로미스의 동작을 결정 할 수 있다.

그럼 이러한 지식을 바탕으로 getStatusChecker 함수를 만들어 보자.  
```js
export function getStatusChecker (promiseIn) {
  let status = "pending";
  let result;

  const promise = promiseIn
    .then((response) => {
      status = "success";
      result = response;
    })

    .catch((error) => {
      status = "error";
      result = error;
    });
  
  return () => ({promise, status, result});
}
```
getStatusChecker는 프로미스 객체를 전달받아 `.then`과 `.catch`를 통해 결과와 상태를 저장해 반환한다.  
예를 들어 메세지 데이터를 적재하는 프로미스를 반환하는 fetchMessage 함수가 있다면, 다음과 같이 상태 추적 함수를 얻을 수 있다.  

```js
const checkStatus = getStatusChecker(fetchMessage);
```

## 프로미스 상태를 사용해 Suspense 통합하기
Message 컴포넌트를 다시 한 번 보자.
```js
function Message() {
  const data = getMessageOrThrow();
  return <p className="message">{data.message}</p>
}
```

getMessageOrThrow 함수는 fetchMessage와 같은 함수가 반환하는 프로미스를 프로미스나 오류를 던질 수 있는 데이터 가져오기 함수로 변환하는 makeThrower 함수가 필요할 것이다.   

```js
// 적절히 예외나 프로미스를 던지는 데이터 적재 함수를 반환하는 함수

export function makeThrower (promiseIn) {
  const checkStatus = getStatusChecker(promiseIn);

  return function () {
    const {promise, status, result} = checkStatus();

    if (status === "pending") throw promise;
    if (status === "error") throw result;
    return result;
  }
}
```

```js
const getMessageOrThrow = makeThrower(fetchMessage());
```

> Suspense와 데이터 적를 통합하는 방법을 살펴보면서 두 가지 핵심 함수를 만들었다.
> 1. getStatusChecker: 프로미스의 상태를 살펴볼 수 있는 창을 제공한다.
> 2. makeThrower: 프로미스를 데이터를 반환하거나 오류나 프로미스를 던지는 프로미스로 반환한다.

이제 데이터를 불러오는데 까지는 했고 남은 의문점은 2개다.
- 언제 데이터를 적재를 시작해야 할까?
- 적재를 시작하는 코드를 어디에 위치시켜야 할까?

## 최대한 빨리 읽어오기
컴포넌트에 필요한 데이터 적재를 시작하기 위해 컴포넌트의 렌더링을 기다릴 필요는 없다.  

컴포넌트 외부에서 임포트를 시작하고, 데이터를 읽어오는 프로미스를 사용해 프로미스 상태에 따라 적절히 값을 돌려줄 수 있는 데이터 접근 함수를 구성할 수 있다.

```js
import React, {Suspense} from "react";
import {ErrorBoundary} from "react-error-boundary";
import fetchMessage from ".api";
import {makeThrower} from "./utils";
import "./styles.css";

// 에러 발생시 보여줄 화면
function ErrorFallback ({error}) {
  return <p className="error">{error}</p>
}

// 가능한 빨리 데이터를 읽어오기 시작함.
const getMessageOrThrow = makeThrower(fetchMessage(3000, true));

function Message () {
  const data = getMessageOrThrow(); // 데이터에 접근하거나 프로미스나 오류를 던짐.
  return <p className="message">{data.message}</p>;
}

export default function App () {
  return (
    <div className="App">
      <ErrorBoundary FallbackComponent={ErrorFallback}>
        <Suspense
          fallback={<p className="loading">Loading message...</p>}
        >
          <Message />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}
```

위 코드에서 Message 컴포넌트는 getMessageOrThrow를 호출할 수 있다. 이 둘이 같은 scope에 있기 때문이다.
> 다른 파일에 있었으면 import/export를 통해 명시적으로 연결해야 했을 것이다.

하지만 항상 그런 것은 아니므로 데이터 접근 함수를 프롭으로 Message에 전달하고 싶을 수 있기 때문이다.  
어쩌면 사용자의 액션에 따라 새로운 데이터를 적재하고 싶을 수도 있다.  

프롭과 이벤트를 사용해 데이터 읽어오는 더 유연하게 처리하는 방법을 살펴보자.

## 새 데이터 읽어오기

| ![01.jpeg](assets/12%EC%9E%A5/01.jpeg) |
|:--------------------------------------:|
|            새로운 문제 상황에 대한 설명            |

이번에는 기본 Hello 메세지를 띄워주고 next를 누를 시, Loading이 뜨며 끝난 후 새로운 메세지를 띄우도록 할 것이다.

```js
const getFirstMessage = makeThrower(fetchMessage());

export default function App () {
  const [getMessage, setGetMessage] = useState(() => getFirstMessage);

  function next () {
    const nextPromise = fetchNextMessage();
    const getNextMessage = makeThrower(nextPromise);
    setGetMessage(() => getNextMessage);
  }

  return (
    <div className="App">
      <ErrorBoundary FallbackComponent={ErrorFallback}>
        <Suspense
          fallback={<p className="loading">Loading message...</p>}
        >
          <Message
            getMessage={getMessage}
            next={next}
          />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}
```

useState에게 최초의 메시지를 가져오는 데이터 적재 함수를 반환하는 초기화 함수 getFirstMessage를 전달한다.  

여기서 getFirstMessage를 호출하지 않는 점에 유의하라.  
getFirstMessage를 호출하는 대신 (람다에서) 반환해서 초기 상태로 설정한다.  
**왜냐, getFirstMessage를 호출하고 싶은게 아니라 새로운 상태 값을 설정하고 싶을 뿐이니까.**

> useState에 함수를 직접 전달하지 않고 람다를 사용하는 이유는 성능 때문이다.  
> `() => getFirstMessage`는 getFirstMessage를 바로 실행하지 않고, 나중에 실행할 수 있도록 감싸줍니다.
> 이렇게 하면 컴포넌트가 리렌더링될 때마다 무거운 데이터 가져오기 작업이 실행되는 것을 방지할 수 있습니다.

```js
function Message ({getMessage, next}) {
  const data = getMessage();
  return (
    <>
      <p className="message">{data.message}</p>
      <button onClick={next}>Next</button>
    </>
  );
}
```

next 버튼을 누르면, 다음 메세지를 적재하기 시작하고 즉시 렌더링 된다. 이는 **적재 시 렌더링** 접근법이다.

## 오류 복구하기(retry)
| ![02.jpeg](assets/12%EC%9E%A5/02.jpeg) |
|:--------------------------------------:|
|            오류 복구 상황에 대한 설명             |

우리는 사용자가 오류가 발생하면 Try again 버튼을 클릭해 리셋하고 다시 렌더링 하고 싶다.

```jsx
function ErrorFallback ({error, resetErrorBoundary}) {
  return (
    <>
      <p className="error">{error}</p>
      <button onClick={resetErrorBoundary}>Try Again</button>
    </>
  );
}
```

```js
// 리스트 12.8 ErrorBoundary에 onRest 프롭 추가하기

export default function App () {
  const [getMessage, setGetMessage] = useState(() => getFirstMessage);

  function next () {/* 변경되지 않음 */}

  return (
    <div className="App">
      <ErrorBoundary
        FallbackComponent={ErrorFallback}
        onReset={next}
      >
        <Suspense
          fallback={<p className="loading">Loading message...</p>}
        >
          <Message getMessage={getMessage} next={next} />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}
```

Errorfallback UI에 onReset 이벤트로 이전의 next 함수를 호출할 수 있게 한다.
attr로 들어간 next 함수는 Errorfallback의 resetErrorBoundary 함수가 되어 onClick 이벤트를 통해 실행된다.

## 리액트 쿼리를 사용해 이미지나 데이터를 미리 적재하는법
|            ![03.jpeg](assets/12%EC%9E%A5/03.jpeg)           |           ![04.jpeg](assets/12%EC%9E%A5/04.jpeg)                         |
|:-----------------------------------------------------------:|:----------------------------------:|
|                            as is                            |               to be                | 

현재(as is)는 워터폴 형태로 데이터와 이미지가 적재된다. 하지만 이를 병력적으로(to be) 동시에 적재하고 싶다.  

이를 위해선 리액트 쿼리의 prefetchQuery를 이용하면 제어가 가능하다.

```jsx
// Users 페이지에서 이미지와 데이터 미리 적재하기

// 그 밖의 임포트들
import {useQueryClient} from "react-query";
import getData from "../../utils/api";

export default function UsersPage () {
  const [loggedInUser] = useUser();
  const [selectedUser, setSelectedUser] = useState(null);
  const user = selectedUser || loggedInUser;
  const queryClient = useQueryClient();

  function switchUser (nextUser) {
    setSelectedUser(nextUser);

    queryClient.prefetchQuery(
      ["user", nextUser.id],
      () => getData(`http://localhost:3001/users/${nextUser.id}`)
    );

    queryClient.prefetchQuery(
      `http://localhost:3001/img/${nextUser.img}`,
      () => new Promise((resolve) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.src = `http://localhost:3001/img/${nextUser.img}`;
      })
    );
  }

  return user ? (
    <main className="users-page">
      <UsersList user={user} setUser={switchUser}/>

      <Suspense fallback={<PageSpinner/>}>
        <UserDetails userID={user.id}/>
      </Suspense>
    </main>
  ) : null;
}// 리스트 12.14 Users 페이지에서 이미지와 데이터 미리 적재하기

// 그 밖의 임포트들

import {useQueryClient} from "react-query";
import getData from "../../utils/api";

export default function UsersPage () {
  const [loggedInUser] = useUser();
  const [selectedUser, setSelectedUser] = useState(null);
  const user = selectedUser || loggedInUser;
  const queryClient = useQueryClient();

  function switchUser (nextUser) {
    setSelectedUser(nextUser);

    queryClient.prefetchQuery(
      ["user", nextUser.id],
      () => getData(`http://localhost:3001/users/${nextUser.id}`)
    );

    queryClient.prefetchQuery(
      `http://localhost:3001/img/${nextUser.img}`,
      () => new Promise((resolve) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.src = `http://localhost:3001/img/${nextUser.img}`;
      })
    );
  }

  return user ? (
    <main className="users-page">
      <UsersList user={user} setUser={switchUser}/>

      <Suspense fallback={<PageSpinner/>}>
        <UserDetails userID={user.id}/>
      </Suspense>
    </main>
  ) : null;
}
```

데이터와 이미지를 빨리 적재하면 사용자를 계속 기다리게 하지 않아도 되고 폴백 이미지를 표시할 필요도 줄일 수 있다. 
워터폴 형태의 적재 방식을 최대한 피할수 있도록 하자.  