# [13장] useDeferredValue, SuspenseList 연습과 실험

**동시성 렌더링**은 리액트가 한 번에 여러 버전의 UI에서 작동하게 하는데, 새 버전이 준비될때 까지 완전히 상호작용할 수 있는 이전 버전을 표시한다.
> ### 동시성 렌더링의 사례
> - **기존 UI 유지**: 새로운 데이터나 업데이트가 준비되는 동안 현재 화면의 UI는 계속 반응하고 작동한다.
> - **백그라운드 준비**: 동시에 React는 메모리상에서 새로운 버전의 UI 트리를 준비한다.
> - **전환**: 새 버전이 완전히 준비되면 그때 한번에 전환된다.

이 말은 단기적으로 쵯신 상태가 브라으저의 현재 UI와 일치하지 않을 수도 있다는 뜻이다.  
이를 위해 리액트는 여러 훅과 기능을 지원한다.  

## 상태 전환을 더 부드럽게 하기
데이터 적재를 기다리는 일은 피할 수 없을 수도 있지만, 스피너를 표시하지 않고 새 데이터를 적재하는 동안 이전 데이터를 계속 표시하면 사용자가 인지하는 페이지 응답 속도를 개선할 수 있다.  

리액트 18부터는 동시성 모드와 일반 모드를 구분하지 않고 모든 리액트 앱이 createRoot를 통해 만들어진 루드의 render() 함수를 통해 렌더링되어야 한다.  
따라서 전통적인 ReactDOM.render()는 더 이상 사용할 수 없다.

```js
import ReactDOM from 'react-dom/client'; // 원서에서는 import ReactDOM from 'react-dom';
import App from './components/App';

const root = document.getElementById("root");
ReactDOM
  .createRoot(root)  // 원서에서는 unstable_createRoot(root)
  .render(<App />);
```

### useTransition을 사용해 후퇴 상태 피하기
현재 데이터에서 다음 데이터를 클릭했을때 화면에 스피너를 보여주는 것 보다 이전 데이터를 보여주는 것이 페이지의 응답 속도를 개선하는데 도움이 된다.
이를 위해서는 상태 변경으로 인해 하위 컴폰너트가 일시중단될 때 리액트가 이전 UI를 표시할 수 있도록 useTransition 훅을 사용해 권한을 부여하는 방법을 사용할 수 있다.
```jsx
// 리스트 13.2 UsersPage에서 UX를 개선하기 위해 전환 사용하기

import {
  useState,
  useTransition,  // 원서에서는 unstable_useTransition as useTransition,
  Suspense
} from "react";

// 변경되지 않은 임포트

export default function UsersPage () {
  const [loggedInUser] = useUser();
  const [selectedUser, setSelectedUser] = useState(null);
  const user = selectedUser || loggedInUser;
  const queryClient = useQueryClient();

  // 원서에서는 const [startTransition] = useTransition();
  const [isPending, startTransition] = useTransition();

  function switchUser (nextUser) {
    // 사용자 상태 변경을 전환으로 감쌈
    startTransition(() => setSelectedUser(nextUser));

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

      <Suspense fallback={<PageSpinner/>}> // 첫 번째로 적재될 때를 위한 스피너
        <UserDetails userID={user.id}/>
      </Suspense>
    </main>
  ) : <PageSpinner/>;
}
```
useTransition 훅은 하위 컴포넌트가 일시중단 될 수 있는 상태 변경을 감쌀 때 사용하는 함수가 포함된 배열을 반환한다. 이 함수를 startTransition 변수에 할당한다.  

우리가 처리할 상태 변경은 switchUser 함수 안에 있다. 새 새용자로 전환할 때 리액트 쿼리가 아직 해당 사용자의 데이터를 적재하지 않았다면 UserDetais 컴포넌트가 일시중단 될 수 있다.  

startTransition으로 이 상태 변경을 감싸면 데이터가 적재될 때까지 Suspense의 폴백 대신에 이전 UI를 계속 표시하도록 리액트에 지시한다.  

이전 UI가 없는 경우 리액트는 데이터를 기다리는 동안 Suspense의 폴백을 표시한다.

```jsx
startTransition(() => setSelectedUser(nextUser));
```

상태 변경 시 스피너로 다시 돌아가지 않는 것은 **새로운 상태 데이터 적재에 시간이 오래 걸리지 않는 한** 개선이라 할 수 있다. 

하지만 새 데이터 적재에 오랜 시간이 걸린다면, 사용자가 이전 UI를 보면서 혼란에 빠질 수 있기 때문에 적절한 피드백을 제공해야 한다.

### isPending을 사용해 사용자에게 피드백 제공하기

useTransition 훅은 상태가 변경되는 동안 리액트가 이전 UI를 표시할 수 있게 해준다.
우리는 더 나은 편의를 위해 데이터가 적재될 동안 화면에 이전 데이터의 불투명도를 줄여 상태 정보가 오래됐음을 보여준다.

useTransition이 배열에서 isPending 이라는 지역변수는 전환이 진행 중임을 알려주는 boolean 값이 담긴다.

```jsx
const [isPending, startTransition]= useTransition();
```

```jsx
// 전환 중 프로퍼티를 설정하기 위해 isPending 값 구조 분해하기

export default function UsersPage () {
  // 상태를 설정함
  
  // 원서에서는 const [startTransition, isPending] = useTransition();
  const [isPending, startTransition] = useTransition();

  function switchUser (nextUser) {
    startTransition(() => setSelectedUser(nextUser));

    // 사용자 상세 정보와 이미지를 미리 적재함
  }

  return user ? (
    <main className="users-page">
      <UsersList user={user} setUser={switchUser}/>

      <Suspense fallback={<PageSpinner/>}>
        <UserDetails userID={user.id} isPending={isPending}/>
      </Suspense>
    </main>
  ) : <PageSpinner/>;
}
```

```jsx
// UserDetails에서 isPending을 써서 클래스 이름 설정하기

export default function UserDetails ({userID, isPending}) {
  const {data: user} = useQuery(/* 사용자 상세 정보 읽어오기 */);

  return (
    <div
      className={isPending ? "item user user-pending" : "item user"}
    >
      {/* 변경되지 않은 UI */}
    </div>
  );
}
```

이렇게 되면 리액트는 같은 컴포넌트의 두 가지 버전을 동시에 관리하게 된다.  

새 userID 값에 대한 UserDetails 컴포넌트는 사용자 데이터를 적재하는 동안 일시중단 된다.  
그러나 전환이 이뤄지는 동안 리액트는 이전 사용자의 UI를 계속 사용하되, isPendnig을 true로 설정해 다시 렌더링한다.

## 공통 컴포넌트와 전환 통합하기
리액트 문서에서는 코드 기반 전체에 useTransitoin 호출을 삽입하는 대신 useTransition 호출을 디자인 시스템에 통합하는 편이 더 낫다고 제안한다.  

```jsx
// 전환을 사용하는 ButtonPending 컴포넌트

import {useTransition} from "react";

import Spinner from "./Spinner";

export default function ButtonPending ({children, onClick, ...props}) {
  // 원서에서는 const [startTransition, isPending] = useTransition();
  const [isPending, startTransition] = useTransition();

  function handleClick () {
    startTransition(onClick);
  }

  return (
    <button onClick={handleClick} {...props}>
      {isPending && <Spinner/>}
      {children}
      {isPending && <Spinner/>}
    </button>
  );
}
```

## useDeferredValue로 이전 값 유지하기
동시성 사용자 인터페이스 소개를 마무리하면서 마지막으로 useDeferredValue 훅을 소개하려한다.  
이전 버전과 새 버전의 값을 모두 유지하고 UI에서 두 버전을 모두 사용한다.

이전 정보에서 새 정보로 전환하면 렌더링하면서 지연이 발생하기 때문에, 새 값을 UI로 렌더링 할 수 있을때까지 지연이 발생하기 때문에, 새 값을 UI로 렌더링할 수 있을 때까지 이전 값을 계속 사용한다.  
즉, 새 값이 지연된다.  

```jsx
// UserDetails에게 지연된 값 전달하기

import {
  useState,
  useDeferredValue, // 원서에서는 unstable_useDeferredValue as useDeferredValue,
  Suspense
} from "react";

// 그 밖의 임포트들

export default function UsersPage () {
  const [loggedInUser] = useUser();
  const [selectedUser, setSelectedUser] = useState(null);
  const user = selectedUser || loggedInUser;
  const queryClient = useQueryClient();
  
  // 사용자 값 추적: 새 값이 렌더링을 지연시키면 이전 값을 반환
  const deferredUser = useDeferredValue(user) || user;
  // 지연된 값이 유효하지 않은 값인 경우 true가 되는 pendin 플래그를 만듬
  const isPending = deferredUser !== user;

  function switchUser (nextUser) {
    // 사용자 값 변경
    setSelectedUser(nextUser)

    queryClient.prefetchQuery(/* 사용자 상세 정보 미리 읽어오기 */);
    queryClient.prefetchQuery(/* 사용자 이미지 미리 읽어오기 */);
  }

  return user ? (
    <main className="users-page">
      <UsersList
        user={user} // 새 사용자를 목록에 반영
        setUser={switchUser}
        isPending={isPending} // 목록의 사용자가 useDetails의 사용자와 일치하지 않는다는 사실을 목록에게 알림.
      />

      <Suspense fallback={<PageSpinner/>}>
        <UserDetails
          userID={deferredUser.id} // 새 사용자의 상세 정보를 기다리는 동안 이전 사용자의 상세 정보를 표시
          isPending={isPending} // 사용자 정보가 오래된 정보라는 사실을 UserDetails에게 알림
        />
      </Suspense>
    </main>
  ) : <PageSpinner/>;
}
```
UsersPage는 이전 사용자와 새로운 사용자 값을 관리하기 위해 useDeferredValue 훅을 사용한다.  
다음과 같이 추적할 값을 인자로 useDeferredValue를 호출한다.  
```jsx
const deferredValue = useDeferredValue(value);
```
이 훅은 value를 추적한다.  
value 가 이전 값에서 새 값으로 변경되면 훅은 두 값 중 하나를 반환할 수 있다.  
리액트가 새 값을 사용해 새 UI를 성공적으로 렌더링하고 자손 컴포넌트 중에 렌더링이 일시중단 되거나 지연된 자손이 없다면 훅이 새 값을 반환하고, 리액트는 UI를 갱신한다.  
리액트가 새 값으로 렌더링을 마치기 위해 어떤 프로세스의 완료를 기ㅣ다려야 하는 경우, 혹은 이전 값을 반환하고 리액트는 기존 값을 사용해 UI를 표시한다.  

## SuspenseList 사용하기
...생략

## 리액트 18과 동시성 모드  
동시성 모드를 사용하면 리액트가 동시에 여러 버전의 UI를 메모리에서 렌더링하고 현재 상태에 가장 적합한 버전만 DOM에 갱신한다.  

이를 통해 현재 상태의 갱신이 완료될 때까지 시간이 걸려도 이전 버전의 UI를 DOM에 표시할 수 있고, 우선순위가 더 높은 갱신이 이뤄지는 동안 리액트가 렌더링을 일시중단할 수 있다.  

이런 유연성은 앱의 반응성을 유지하고 사용자가 이니지하는 앱의 성능을 개선하는 데 도움이 된다.  

메모리에서 갱신을 준비할 수 있는 기능으로 인해 리액트는 충분한 준비가 이뤄졌을 때 새 페이지나 필터링된 목록, 사용자 상세 정보 등의 UI로 전환할 수 있다.

동시성 모드는 코드, 데이터, 자원을 더 의도적으로 명확히 목표를 정해 적재할 수 있게 해주며, 서버 측 렌더링과 클라이언트 측 컴포넌트의 수화(hydration)를 더욱 원활하게 통합해서 컴포넌트가 사용자와 적시에 상호작용할 수 있도록 자원을 주입할 수 있게 해준다.  

