# [8장] 컨텍스트 API로 상태 관리하기

컴포넌트에게 상태가 필요하다면 가급적 해당 컴포넌트 안에서 해결하는 것이 좋다.

하지만 상태를 전달해야하는 경우 상태에 관심이 없는데도 받아야 하는 경우가 생길수도 있다.  
컨텍스트 API는 컴포넌트 트리에서 이런 정보에 관심이 없는 중간 단계를 굳이 거치지 않고 직접 필요한 컴포넌트에게 상태를 전달하는 방법을 제공한다.

## 컨텍스트 API
컨텍스트 API는 `createContext()`를 통해 객체를 생성한다.
```jsx
const UserContext = createContext();
```
여기서 UserContext라는 객체는 앱 전체에서 현재 사용자 값을 공유하는 열쇠다.

이러한 컨텍스트 객체를 사용하려면 **컨텍스트의 Provider 컴포넌트를 사용해 상태를 필요로하는 컴포넌트를 감싸주면 된다.**

## 컨텍스트와 렌더링
```jsx
export default function App () {
  const [user, setUser] = useState();

  return (
    <UserContext.Provider value={user}>
      <Router>
        <div className="App">
          <header>
            <nav>
              // ...
            </nav>

            <UserPicker user={user} setUser={setUser}/>
          </header>

          <Routes>
            // ...
          </Routes>
        </div>
      </Router>
    </UserContext.Provider>
  );
}
```
위 컴포넌트에서 만약 UserPicker 내에서 setUser이 동작해 유저를 결정하게 되면, App 컴포넌트가 재렌더링되며 갱신된 유저를 UserContext.Provider의 값으로 설정한다.
> Context 값이 바뀌면, 그 Context Provider 하위에 있는(즉, 트리 구조상 해당 Provider를 감싸고 있는) 모든 컴포넌트들이 새로 렌더 과정을 거치게 된다.

이때 문제가 발생한다.  
재렌더링의 범위는 Context Provider 하위에 있는 모든 컴포넌트이기 때문에 컨텍스트의 상태 값을 사용하지 않는 컴포넌트도 렌더링이 발생하게 된다.

이 문제를 어떻게 해결하면 좋을까?

## 커스텀 프로바이더와 여러 컨텍스트 사용하기
우리는 컨텍스트를 통해 상태 값을 소비하는 컴포넌트들만 재렌더링 되길 원한다.

### 객체를 컨텍스트 프로바이더의 값으로 설정하기
다시 한 번 아까 전의 예시코드를 보자.
```jsx
export default function App () {
  const [user, setUser] = useState();

  return (
    <UserContext.Provider value={user}>
      <Router>
        <div className="App">
            // ...
            <UserPicker user={user} setUser={setUser}/>
```
이 경우 UserPicker는 App 컴포넌트의 user와 setUser를 필요로 한다.  
하지만 이 경우 UserPicker에서 갱신함수가 발생하면, 아까 말했듯이 불필요한 렌더링이 발생하게 된다.

우선 이 문제를 해결하기 위해서 UserPicker가 컨텍스트로 부터 user와 setUser값을 받아서 쓰게 하자.
```jsx
    <UserContext.Provider value={{ user, setUser }}>
    // jsx
    <UserContext.Provider/>
```
그 후 만약 컨텍스트의 값이 필요하다면 다음과 같이 구조분해를 통해 사용하면 된다.
```jsx
    // case 1
    const {user} = useContext(UserContext);
    // case 2
    const {user, setUser} = useContext(UserContext);
    // case 3
    const {user: loggedInUser} = useContext(UserContext);
```

### 커스텀 프로바이더로 상태 옮기기
위 예시코드를 보면 App 컴포넌트 내부에서 user에 대한 상태가 관리되고 있다.

|   ![01.jpeg](assets/8%EC%9E%A5/01.jpeg)   |
|:-----------------------------------------:|
| user 값이 바뀜에 따라 App 하위 모든 컴포넌트가 재렌더링 되는 경우 |

컨텍스트의 프로바이더의 값을 갱신하면 컴포넌트 트리 전체가 재렌더링되는 것이 아니라 프로바이더의 값이 변할 때 컨텍스트 소비자들만 재렌더링 되도록 하고 싶다면 아래 개념들을 이해해야 한다.
1. 커스텀 프로바이더 생성하기
2. children 프롭을 사용해 감싸인 컴포넌트들만 렌더링하기
3. 불필요한 재렌더링 피하기
4. 커스텀 프로바이더 사용하기

#### children 프롭을 사용해 감싸인 컴포넌트들만 렌더링하기
```jsx
import {createContext, useState} from "react";

const UserContext = createContext();
export default UserContext;

export function UserProvider ({children}) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
        {children}
    </UserContext.Provider>
  );
}
```
다음과 같이 커스텀 프로바이더를 생성해 children에 할당된 컴포넌트들에게만 값을 전달하게 할 수 있다.  
위와 같은 코드는 조직화, 코드에 대한 이해, 코드 유지보수 측면에서 유용하다.

#### 불필요한 재렌더링 피하기
```jsx
export default function App () {
  return (
    <UserProvider>
      <Router>
        <div className="App">
            // ...
            <UserPicker />
```
다음과 같은 구조에서 생각을 해보자.

UserPicker가 UserProvider 컴포넌트 내에서 setUser을 호출해 user 값을 변경하면, 리액트는 상태가 바뀌었음을 감지하고 해당상태를 관리하는 UserProvider 컴포넌트를 재렌더링 한다.  
하지만 UserProvider를 제외한 모든 자식 컴포넌트는 재렌더링 되지 않는다. 이를 아래 그림이 보여준다.

| ![02.jpeg](assets/8%EC%9E%A5/02.jpeg) |
|:-------------------------------------:|
|         컨텍스트 소비자들만 재렌더링 되는 경우         |

왜 리액트는 프로바이더의 자식들을 렌더링하지 않을까.

그 이유는 UserProvider가 자식들을 프롭을 통해 접근하고 컴포넌트 안에서 상태를 갱신해도 프롭은 변경되지 않기 때문이다.  
UserPicker 같은 후손이 setUser을 호출해도 children의 정체성은 변하지 않는다.  

하지만 Context 소비자는 예외다!  
컨텍스트 소비자는 자신의 컨텍스트에 속하는 가장 가까운 프로바이더의 값이 변경될 때마다 항상 재렌더링된다.

#### 상태 값과 그에 해당하는 갱신 함수에 대해 별도의 컨텍스트 사용하기

프로바이더의 값이 객체이고 코드가 프로바이더를 렌더링할 때마다 이 값 객체를 새로 생성하는 경우, 객체에 할당한 프로퍼티 값이 같아도 재렌더링이 일어날 때마다 값이 변경된다.  
```jsx
import {createContext, useState} from "react";

const UserContext = createContext();
export default UserContext;

export function UserProvider ({children}) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}> // 렌더링이 일어날 때마다 새 객체가 value props에 대입됨.
        {children}
    </UserContext.Provider>
  );
}
```

UserProvider가 렌더링 될때마다 user, setUser의 값이 이전과 같더라도 매번 새로운 객체가 value에 할당된다.  
이 컨텍스트의 소비자들은 UserProvider가 재렌더링 될 때마다 재렌더링 된다.

즉, 두가지 문제가 있다.
- 렌더링이 일어날 때마다 프로바이더의 value에 새로운 객체가 할당된다.
- value에 할당된 값이 어느 한 프로퍼티를 변경하면 그 프로퍼티를 사용하지 않는 소비자들도 재렌더링 된다.

이 문제는 2개의 컨텍스트를 쓰면 문제를 해결할 수 있다.

```jsx
import {createContext, useState} from "react";

const UserContext = createContext();
export default UserContext;

export const UserSetContext = createContext();

export function UserProvider ({children}) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={user}>
      <UserSetContext.Provider value={setUser}>
        {children}
      </UserSetContext.Provider>
    </UserContext.Provider>
  );
}
```

이렇게 하면 렌더링이 일어나도 user와 setUser가 매번 재생성되지 않으며, 각각 별도의 프로바이더를 사용하기 때문에 한 가지 값을 사용하는 소비자가 다른 값에 변화에 영향을 받지 않는다.

## 요약

| ![03.jpeg](assets/8%EC%9E%A5/03.jpeg) |
|:-------------------------------------:|
|                  요약                   |
