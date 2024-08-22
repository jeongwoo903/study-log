# [CS] 컴포넌트 분해와 추상화

SOLID 원칙의 SRP(단일 책임 원칙)에 따라, 프론트엔드의 컴포넌트도 **"하나의 컴포넌가 하나의 책임만을 담당하게 하는게 좋다"**고 한다.  
여기서 **책임**이란 뭘까? 책임이란 어플리케이션에서 컴포넌트가 수행하는 행동을 의미한다.

## 컴포넌트 분해 (Decomposition)
컴포넌트 분해는 복잡한 시스템이나 어플리케이션을 작은 부분으로 나누는 과정이다.  
복잡한 문제를 여러 개의 독립적인 모듈이나 컴포넌트로 분리함으로써 더 쉽게 관리하고 유지보수할 수 있도록 한다.  
각 컴포넌트는 특정한 기능을 담당하며, 전체 시스템에서 중요한 역할을 한다.  

컴포넌트 분해는 왜 하는 걸까? **복잡성을 줄이기 위해서이다.**  
예를 들어, 대규모 웹 애플리케이션에서 다양한 기능을 담당하는 여러 컴포넌트를 설계할 수 있다. 각 컴포넌트는 독립적으로 개발, 테스트, 유지보수될 수 있으며, 다른 컴포넌트와 상호작용할 때 명확하게 정의된 인터페이스를 통해 통신한다. 이를 통해 전체 시스템의 안정성이 향상되고, 코드의 재사용성이 증가한다.

컴포넌트 분해는 개발자 간의 협업을 촉진하고, 새로운 기능을 추가하거나 기존 기능을 수정할 때 영향을 최소화할 수 있다. 또한, 시스템이 점점 더 커지고 복잡해짐에 따라 컴포넌트 단위로 변경 사항을 관리할 수 있어 유지보수 비용을 절감할 수 있다.  

### 컴포넌트 분해의 장점
1. 테스트 용이성의 증가
   - 작은 컴포넌트는 테스트를 해보기 편하다.
2. 재사용성의 증가
   - 잘 설계된 컴포넌트는 필요한 곳에 따라 재사용할 수 있다.
3. 유지보수성 증가
   - 각 컴포넌트의 기능이 독립적이라 의존성이 적다. 즉, 다른 부분에 미치는 영향을 최소화 할 수 있다.

## 컴포넌트 추상화 (Abstraction)
컴포넌트 추상화는 시스템의 복잡한 내부 구조를 감추고, 외부에 노출되는 인터페이스만을 통해 컴포넌트를 다룰 수 있도록 하는 과정이다.  
추상화를 통해 사용자는 컴포넌트가 어떻게 구현되었는지 알 필요 없이, 제공되는 기능만을 이용할 수 있다.

추상화의 핵심은 컴포넌트의 내부적으로 구현된 것들을 감추고 외부적으로 필요한 정보만 노출하는 것이다.  
이는 시스템의 복잡성을 줄이는 동시에, 컴포넌트의 변경에 대한 영향을 최소화(유지보수성 증가)하는 데 도움이 된다. 예를 들어, 데이터베이스 연결을 처리하는 컴포넌트가 있다고 가정하자. 이 컴포넌트가 내부적으로 어떤 방식으로 데이터를 처리하는지는 중요하지 않으며, 외부에서는 데이터베이스 연결 및 쿼리 실행에 필요한 인터페이스만 알면 된다.

결론적으로, 컴포넌트 분해와 추상화는 복잡한 시스템을 보다 효율적으로 설계하고 관리하는 데 중요한 기법이다. 분해를 통해 시스템을 작은 단위로 나누고, 추상화를 통해 복잡성을 감추어 시스템의 유연성과 재사용성을 높일 수 있다.

### 컴포넌트 추상화의 장점
1. 복잡성 관리
   - 복잡한 시스템을 더 쉽게 이해하고 사용할 수 있다.
2. 유연성
   - 추상화된 인터페이스를 사용하면, 내부 구현을 변경하더라도 외부에 미치는 영향을 줄일 수 있다.
3. 모듈화
   - 추상화를 통해 모듈 간의 의존성을 줄이고, 서로 독립적으로 동작할 수 있도록 만든다.

- ex) API 설계
```typescript jsx
// StorageInterface.js
class StorageInterface {
    save(key, value) {
        throw new Error("save() must be implemented");
    }

    load(key) {
        throw new Error("load() must be implemented");
    }
}

// LocalStorage.js
class LocalStorage extends StorageInterface {
    save(key, value) {
        localStorage.setItem(key, value);
    }

    load(key) {
        return localStorage.getItem(key);
    }
}

// SessionStorage.js
class SessionStorage extends StorageInterface {
    save(key, value) {
        sessionStorage.setItem(key, value);
    }

    load(key) {
        return sessionStorage.getItem(key);
    }
}

// App.js
function App(storage) {
    storage.save("username", "JohnDoe");
    console.log(storage.load("username"));  // "JohnDoe"
}

// 사용할 스토리지를 선택해서 앱 실행
const localStorageInstance = new LocalStorage();
const sessionStorageInstance = new SessionStorage();

App(localStorageInstance);  // LocalStorage를 사용
App(sessionStorageInstance);  // SessionStorage를 사용
```

## 범용적인 컴포넌트 만드는 법
데이터를 중심으로 애플리케이션을 보게되면, 컴포넌트는 크게 2가지로 볼 수 있다.
1. 어떤 도메인과도 상관없는 범용적인 컴포넌트
2. 특정 도메인에 종속된 비즈니스 컴포넌트.

### 범용적인 컴포넌트 구조의 특징
1. 도메인에 종속되지 말기
2. 합성 패턴 이용하기
3. 커스텀 훅 이용하기