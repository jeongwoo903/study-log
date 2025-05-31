# [WEB] 서버 사이드 렌더링(SSR)에 대한 고찰

## 서론

서버 사이드 렌더링의 개념 그 너머를 보기 위한 글이다.

## SSR 과거와 현대

요즘 Next.js와 Nuxt.js 같은 프레임워크들이 SSR 프레임워크로 부상하지만, SSR의 개념 자체는 먼 과거부터 있어 왔다.

스프링, PHP 처럼 서버에서 렌더링을 한 후 브라우저에 뿌렸기 때문에 과거에는 오히려 CSR 형태보다 SSR이 더 자주 쓰이는 형태였다.

하지만 이는 서버의 부담이 많이 갔고 SPA의 유행을 따라가기에 SSR의 방식은 적합하지 못했다.  
그래서 React와 같은 CSR-SPA가 가능한 프레임워크들이 유행을 했었다.  
다만 고질적인 문제가 있다면,

1. 초기 로딩 느림
2. JS 꺼진 브라우저에서는 내용 없음 (SEO 불리)

이런 문제들이 있었다.

현대에 와서 하드웨어 기술이나 클라우드 인프라, CDN, 서버리스 환경이 발달함에 따라서 서버의 부담을 줄일 수 있는 기술이 많이 늘어났다. (서버에서는 REST API로 JSON이나 XML를 만들고, 웹 브라우저에서 Javascript로 UI를 그리는 현재의 모습이 된 것이다.)

그렇기 때문에 **좋은 SEO와 빠른 렌더링을 가졌던 SSR**과 **유저 인터렉션에 잘 대응했던 CSR**의 장점만 뽑아내 현재적인 SSR을 추구해 유연성을 극대화 하고 있다.

이를 통해 Next.js와 같은 프레임워크를 쓰면, 유저 경험과 비즈니스 목표에 따라 전략적으로 대응할 수 있게 되어 최근 많은 사랑을 받고 있다 생각한다.

| 구분        | 전통 SSR (PHP 등) | CSR (React 등) | 현대 SSR (Next.js 등) |
| ----------- | ----------------- | -------------- | --------------------- |
| 초기 로딩   | 빠름              | 느림           | 빠름                  |
| 사용자 경험 | 떨어짐            | 뛰어남         | 뛰어남                |
| SEO         | 강함              | 약함           | 강함                  |
| 서버 부담   | 큼                | 적음           | 중간 (분산 가능)      |
| 유연성      | 낮음              | 높음           | 가장 높음             |

### 조금 더 자세한 정리

1. 과거에는 원래 SSR이 자연스러웠다.
2. 스마트폰과 모바일 앱이 등장했다.
3. 모바일 앱은 개발 비용과 유지 비용이 많이 든다. (iOS + Android + 검수 + 업데이트)
4. 이를 해결하기 위해 앱에 웹을 불러오는 하이브리드 방식의 앱을 만들기 시작했다.
5. 다만 웹은 앱에 비해 무척 느리고 무거웠기 때문에, 앱과 유사한 사용성을 제공하기 위해 많은 연구가 이루어졌고, 브라우저가 발전해갔으며 자연스럽게 프론트엔드 개발자들이 생겼고, 프론트엔드 프레임워크도 생겼다.
6. 지금은 브라우저에서 오직 Javscript만 이용하여 UI를 만드는 것(CSR)이 자연스러운 모습으로 자리잡혔다.

출처: https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Server-Side-Rendering/#_2-csr%E1%84%8B%E1%85%B4-%E1%84%83%E1%85%B3%E1%86%BC%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5-%E1%84%81%E1%85%A1%E1%84%8C%E1%85%B5

## 왜 SSR이 필요한가

1. CSR을 하면 JS 번들의 사이즈가 늘어난다.
   - UI 조작에 의한 변화를 필요로 하지 않는 부분까지 한꺼번에 담겨서 전달이 되다 보니 JS 번들의 사이즈가 늘어나고 FCP가 느려지게 된다.
2. CSR에서는 서버로 부터 최초로 받아오는 html은 고작 `<div id="app"></div>`에 불과할건데, 이렇게 될 경우 검색엔진이 사이트의 내용을 파악하여 색인하는 것이 불가능 해진다.

## SSR 이란?

SSR은 서버에서 렌더링된 페이지를 클라이언트가 받아본다는 것이다.

이를 뜯어보면, 만약 React의 구조를 MVVM 패턴으로 고안해 보았을때
| MVVM 역할 | React 대응 요소 |
| --------- | ---------------------------------------- |
| Model | 상태(state), 비즈니스 로직 (예: Redux, API 호출 로직) |
| ViewModel | 컴포넌트 (UI + 상태 관리 및 로직 연결) |
| View | JSX를 통한 실제 UI 출력 (브라우저에서 DOM으로 랜더링) |

이렇다고 한다면, SSR이란 **React 컴포넌트를 서버에서 HTML 문자열로 렌더링한 후, 이를 클라이언트에 전달하는 것**이라 볼 수 있다.

SSR은 React 컴포넌트를 서버에서 HTML 문자열로 렌더링하고, 클라이언트에 전달하는 방식입니다. 이후 클라이언트는 Hydration을 통해 이 정적인 HTML에 인터랙션을 추가합니다.

### CSR과의 차이

SSR

- React 컴포넌트를 서버에서 HTML 문자열로 렌더링하고, 브라우저로 전송
- 브라우저는 이 HTML을 즉시 렌더링 → 초기 화면이 빠르게 보임
- 이후 JavaScript 번들이 로드되고, React는 해당 HTML을 hydrate해서 인터랙션 가능하게 만듦

CSR

- React 앱 전체를 브라우저에 JS 번들로 보내고, 브라우저가 초기 렌더링을 담당
- 서버는 거의 빈 HTML을 보내고 `<div id="root"></div>` 안에 JS가 렌더링함

## hydration

하이드레이션은 수화 라고도 하는데, 왜 이런 이름이내면 마른 static component에 이벤트라는 비를 뿌려주기 때문이다.. 라고 알고 있다.
뭐 정확히는 **서버에서 미리 렌더링된 HTML을 클라이언트 측에서 JavaScript가 "활성화"시키는 과정을 의미한다.**

js로 이런 hydration을 구현하기 위해서는 SSR로 구성한 서버에 컴포넌트를 static으로 등록해줄 필요가 있다.

```js
// static 파일 등록
app.use("/src", express.static("./src"));
```

node.js를 통해서 js로 SSR을 구현하면, 변경된 데이터를 불러오기 위해 새로고침을 해주어야 한다.

```js
// 대략 이런 형태를 띔
document.querySelector("#delete").onclick = () => {
  fetch("/api/items/0", { method: "delete" }).then(() => location.reload());
};
```

렌더링을 오직 Server에 의존하고 있기 때문에, 문제를 해결하기 위해선 Client에서도 데이터를 받아와서 렌더링할 수 있는 코드가 있어야 한다.

```js
// Before
function main() {
  document.querySelector("#add").onclick = () => {
    fetch("/api/items", {
      method: "post",
      body: JSON.stringify({ content: "추가된 음식" }),
      headers: {
        "Content-Type": "application/json",
      },
    }).then(() => location.reload());
  };

  document.querySelector("#delete").onclick = () => {
    fetch("/api/items/0", { method: "delete" }).then(() => location.reload());
  };
}

main();

// After
import { App } from "./components.js";
import { model } from "./model.js";

function render() {
  const $app = document.querySelector("#app");
  $app.innerHTML = App(model.items);

  $app.querySelector("#add").onclick = () => {
    model.addItem("추가된 음식");
    render();
  };

  $app.querySelector("#delete").onclick = () => {
    model.deleteItem(0);
    render();
  };
}

function main() {
  render();
}

main();
```

Before 구조에선 서버가 상태를 관리했기 때문에, 변경 시 전체 페이지를 새로고침 해주어야 브라우저가 상태가 바뀐것을 확인 할 수 있었다.

After 구조에서는 클라이언트의 model 객체가 상태를 관리 하게 되면서 페이지를 새로고침할 필요없이 브라우저에서 바로 바로 상태변경을 확인 할 수 있게 되었다.

이는 현대의 React와 같이 상태가 변경되면 해당 상태를 사용하는 UI만 다시 그리는 방식이다.

하지만 현재 방식은 SSR과 CSR의 데이터가 동기화 되지 않아 새로고침시 상태가 유지 되지 않는다.

### SSR과 CSR 데이터 동기화

### 정리

1. SSR 기반으로 컴포넌트를 string 화 해서 렌더링한다.
2. SSR로 뿌려진 UI중 이벤트가 필요한 부분은 CSR 처럼 동작하게 한다.
3. 서버와 브라우저간의 데이터를 양방향으로 동기화 해 상태가 유지되게 한다.
