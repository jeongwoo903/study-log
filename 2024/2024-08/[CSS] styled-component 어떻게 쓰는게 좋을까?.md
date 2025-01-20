# [CSS] styled-component 어떻게 쓰는게 좋을까?
## 서론
난 styled-component와 아직 친해지지 못했다.  
왜냐면 html 태그에 직접 스타일을 삽입하는게 아니라 자체적인 컴포넌트를 만들어서 사용한다는 점 때문에 어떻게 사용하는 것이 좋은 방법인지 잘 모르겠기 때문이다.

하지만.. 회사에서 주로 사용하는 css 라이브러리라고 하니 이번 기회에 좋은 방법들과 레거시들을 참고해 정리해 보려고 한다.

## styled-component란?
CSS in JS 방식 이라고 하여서, 말 그대로 css를 js 파일 안에 작성한다는 것이다.  
html+css+js를 묶어서 하나의 js파일에 넣은 뒤 컴포넌트 단위로 개발할 수 있게 만들어 준다.  

**js 파일에서 css를 관리하면 좋은점**은 **네이밍 문제**와 **코드 유지보수 문제**가 해결된다.  

직접 class를 정의해 css를 관리하는 방법은 차후 class 네임이 겹치거나 글로벌 네이밍을 정하는데 고민을 하게 된다.  
하지만 js파일에서 css를 관리하면 특정 js 파일 내에서만 사용하거나, js처럼 의존성을 관리할 수 있으므로 네이밍 문제와 유지보수 문제를 해결할 수 있다.  

하지만 단점이 있다면 단순 css를 사용하는 것보다 js 런타임 작업이 들어가다 보니 조금 느리다는 단점이 있다.  

## 설치법
```shell
# 일반적인 설치법
$ yarn add styled-components

# 버전이 안 맞아서 에러가 날 경우 가장 최신 버전으로 설치
$ yarn add styled-components@latest
```
만약 여러 버전의 styled-component가 설치되어 에러가 발생하는 문제를 방지하고 싶다면 다음 코드를 추가하자.
```json
{
  "resolutions": {
    "styled-components": "^5"
  }
}
```
사용할때는 파일에 다음 코드를 import 하자.
```js
import styled from "styled-components"
```

## 사용법
### 기본
일반적인 사용법은 컴포넌트의 이름은 대문자로 시작하게, `styled.태그네임`과 같은 형태로 작성해서 쓴다.
```typescript jsx
const List = styled.div`
    display: flex;
    ...
`
```
### 재활용
기존의 만들어둔 컴포넌트를 상속해서 쓰고 싶다면 `styled(컴포넌트 이름)`과 같은 형태로 사용한다.
```typescript jsx
const AlbumList = styled(List)`
    display: flex;
    ...
`
```
### props 사용
CSS in JS 방식이라 props를 활용할 수 있다는 장점이 있다.
```typescript jsx
const List = styled.div`
    background-color: ${(props) => props.color ? props.color : "white" }
    ...
`

또는

const List = styled.div`
    background-color: ${(props) => props.color || "white" }
    ...
`
```

### 다른 컴포넌트 조작
`${컴포넌트}` 다음과 같은 형식으로 컴포넌트를 가져와 다른 컴포넌트 내에서 속성을 변경하거나 의존되어 변경되게 할 수도 있다.
```typescript jsx

const Text = styled.div`
  display: none;
  ...
`;

const Link = styled.a`
  ...

  //Link hover 할때 하위컴포넌트인 Text가 나타나게 함
  &:hover {
    ${Text} {
      display: block;
    }
  }
`;

const Icon = styled.svg`
  fill: none;
  
  //상위 컴포넌트인 Link를 hover 할때 변화
  ${Link}:hover & {
    fill: revert;
  }
`;
```

## 전역 스타일 설정
styled-component에서는 전역 스타일을 지정하기 위한 함수를 제공한다.  
`createGlobalStyle` 함수를 사용하면 전역 스타일 지정이 가능하다.
```typescript jsx
import { createGlobalStyle } from "styled-components";

const GlobalStyle = createGlobalStyle`
    * {
        padding : 0;
        margin : 0;
    }
`
```
이렇게 만든 전역 스타일 컴포넌트를 최상위에 배치해주면 적용이 된다.
```typescript jsx
function App() {
	return (
        <>
            <GlobalStyle />
            ...
        </>
    );
}
```

## as 태그 사용
`as`속성을 통해서 스타일을 받는 element를 동적으로 swap할수도 있다.  
`as`로는 태그 뿐만 아니라 컴포넌트도 상속받을 수 있다.

```typescript jsx
const Button = styled.button`
  display: inline-block;
  color: #BF4F74;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid #BF4F74;
  border-radius: 3px;
  display: block;
`;

const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <Button as="a" href="#">Link with Button styles</Button>
    <TomatoButton as="a" href="#">Link with Tomato Button styles</TomatoButton>
  </div>
);
```

## css 변수 사용하기
styled-component에는 `styled` 변수 외에 `css`라고 하는 변수가 존재한다.  
사용 방법은 아래와 같이 import 해서 사용하면 된다.
```js
import { css } from 'styled-components';
```
### 조건부 css
위에서 props를 사용해서 상황에 따라 값을 달리 보여주던 것과 비슷하게 조건에 따라 다른 css를 보여줄 수도 있다.
```typescript jsx
import { css, styled } from 'styled-components';

const Button = styled.button`
    width: 50px;
    background-color: white;
    ${(props) =>
    	props.isClicked ?
        css`
        	background-color: purple;
        `;
    }
`;
```

### 주로 사용하는 css 분리
scss의 `@mixin & @include` 와 같이 자주 사용하는 css를 변수화 해둘 수 있다.
```typescript jsx
const FLEX_CENTER = css`
    display: flex;
    align-items: center;
    justify-content: center;
`

const CenterBox = css`
    ${FLEX_CENTER}
`
```

## 속성(attrs) 추가
attrs 함수를 사용하면 styled component에 속성(attribute)을 추가할 수 있다.  
이를 통해 styled component에 동적인 속성을 전달하고, 해당 속성에 기반한 스타일을 정의할 수 있다.

attrs 함수는 첫 번째 매개변수로 속성을 포함한 객체를 받는다.
각 속성은 이름과 값을 가지며, 해당 속성을 스타일 컴포넌트에 전달할 때 사용된다.

```typescript jsx
import styled from 'styled-components';

const Input = styled.input.attrs(
    {
        type: 'text',
        placeholder: "Enter text"
    }
)`
  padding: 10px;
  border: 1px solid #ccc;
`;

const App = () => {
    return <Input />;
};
```
명시한 속성들은 컴포넌트를 선언할때 따로 attribute를 선언 하지 않아도 된다.
또한 attrs 함수는 함수의 인자로 props를 받아서 동적인 속성을 정의 할 수 있다.
```typescript jsx
import styled from 'styled-components';

const Button = styled.button.attrs(props => ({
    disabled: props.disabled ? true : undefined,
}))`
`;

const App = () => {
    const isDisabled = true;
    return <Button disabled={isDisabled}>Submit</Button>;
};
```

## css 선택자
css 선택자(selector)를 통해서 다른 컴포넌트의 영향을 미칠수도 있다.
```css
  // 마우스가 올라갈때
  &:hover {
    color: red;
  }

  // 형제요소일 때
  & ~ & {
    background: tomato;
  }

  // 현재 요소 바로 옆에 현재 요소가 붙어있을 때
  & + & {
    background: lime;
  }
  
  // add 라는 클래스를 갖고있을 때
  &.add {
    background: orange;
  }
  
  // container 라는 클래스를 가진 부모안에 있을 때
  .container & {
    border: 1px solid;
  }
```

## + 모든 것을 다 컴포넌트화 해서 사용해야 하는가?
이 글을 쓰게된 주된 고민이다.

사실 모든 것을 다 컴포넌트화 하지 않아도 된다는 사실을 알고 있지만, 어떤 부분에서 컴포넌트를 쓰고 어떤 부분에서 일반 태그를 써야하는지 나한테 명확한 근거가 없다.  

우선, 모든 것을 컴포넌트화 해서 사용하면 코드의 통일감이 생겨서 좋다.
또한 네이밍이 확실해서 의도가 잘 전달되고 컴포넌트 하나하나 만들기가 신중해진다.  
간단한 div 태그라도 의미를 신중하게 생각하게 되는 것 같다.

하지만 아래와 같은 코드를 짰다고 생각해보자.
```typescript jsx
const TitleContainer = styled.div`
  width: 100%;
  display: flex;
`;
const TitleText = styled.h1`
  color: black;
`;
const TitleLink = styled.a`
  color: blue;
`;
...
<TitleContainer>
  <TitleText>제목</TitleText>
  <TitleLink>링크</TitleLink>
</TitleContainer>
```
일반적인 html 코드로 구현한다면 보다 빠르고 쉽게 할 수 있었을 것을 컴포넌트화 하다 보니 작성하는 입장에서 피로감이 느껴진다.

이 부분에 있어서 여러 글들을 찾아보니 동일한 문제를 느끼신 분이 계셨고 다음과 같이 처리하고 계셨다.
- 구조에 대한 큰 틀은 컴포넌트로 만든다.
- 내부적으로 자잘한 부분은 따로 컴포넌트화 하지않고 상위 컴포넌트에서 css를 작성해 관리한다.

위 내용들을 코드로 나타내면 아래와 같다.
```typescript jsx
const Title = styled.div`
  width: 100%;
  display: flex;
  .h1 {
    color: black;
  }
  .a {
    color: blue;
  }
`;
<Title>
  <h1>제목</h1>
  <a>링크</a>
</Title>
```
html 태그와 stled component가 섞인 형태지만 다음과 같은 식으로 작성한다면 css에서 제공하는 기능과 styled component의 특징을 잘 섞어서 사용할 수 있을 것 같아보인다.

이런 부분에 있어서 best practice는 없다고 생각한다.  
그래서 나에게 잘 맞는 컨벤션을 생각해 사용하는 것이 중요해 보인다.

---
이 외에도 keyframe, `&&`, ThemeProvider 등등 여러 문법들이 있으니 알아보자.

## 참고
- [styled-component 공식문서](https://styled-components.com/docs/basics)
- [Web: styled-components와 사용기](https://medium.com/hcleedev/web-styled-components%EC%99%80-%EC%82%AC%EC%9A%A9%EA%B8%B0-bb7bac75b569)
- [[Styled Component] 정리글 1-7](https://white120.tistory.com/62)
- [[Front-End] 복잡한 styled-components 구조 개선해보기](https://velog.io/@hayoung474/Front-End-%EB%B3%B5%EC%9E%A1%ED%95%9C-styled-components-%EA%B5%AC%EC%A1%B0-%EA%B0%9C%EC%84%A0%ED%95%B4%EB%B3%B4%EA%B8%B0)

