# [React] React 유저가 Function 방식을 선호하는 이유

## 서론
내가 처음 React를 접한 20년도 쯤만 하더라도 Class 방식이 우세였다.  
하지만 React가 업데이트를 진행하면서 Class 방식에서만 사용가능했던 것들을 Functional 방식에서도 지원하면서 부터 Function 방식이 시장의 우세로 자리잡고 있는 것으로 알고 있는데.. 명확한 이유가 생각나지 않아 한 번 알아보려고 한다.

## React에서 Component를 만드는 2가지 방법
React에선 Component를 만들기 위해서 2가지 방식을 사용할 수 있다.
1. Class-based
2. Functional

집을 짓는 다고 가정할때 벽돌이 필요하듯이, React에서는 Component가 벽돌의 역할을 한다고 볼 수 있다.  
Component는 UI를 관리가 용이한 조각들로 나누어 새로운 UI를 만들 수 있도록 해주기도 한다.  

## Functional Component 란?
JSX를 return 하는 JS 함수라고 볼 수 있다.  

많은 상태나 로직을 필요로 하지 않는, 아주 작은 조각의 UI를 만들기엔 Functional 방식이 간편하다.

Functional components는 기존에는 **life-cycle event를 다루지 못했다.**  
그래서 이전엔 Class Component 방식이 선호되었으나 **hooks의 등장**으로 Class Component 처럼 life-cycle을 다룰 수 있게 되었다.

## Class Component 란?
Class 방식은 Functional 방식보다 조금 복잡하다.
이는 ES6 Class를 기반으로 하며 `React.Component`를 extend 한다.
```typescript jsx
class Human extends React.component {
    render() {
        return (<h1>Hello, {this.props.name}</h1>)
    }
}
```
Class 방식은 life-cycle을 handling 할 수 있는 built-in 메서드가 함께 제공된다.

## 조금 더 자세히 들여다보기
### JSX 렌더링 차이
Functional 방식은 return을 통해서 렌더링 하고 Class 방식은 render()이란 메서드를 통해서 렌더링한다.
```typescript jsx
//Functional Components
function DarkButton() {
    return (
        <button type={button}>버튼</button>
    )
}

//Class-baesd Component
class DarkButton extends React.component {
    render() {
        return (<button type={button}>버튼</button>)
    }
}
```

### props는 받는 방식 차이
```typescript jsx
//Functional Components

function DarkButton(props) {
    return (
        <button type={button}>{props.buttonText}</button>
    )
}

//Class-baesd Component
class DarkButton extends React.component {
    render() {
        return (<button type={button}>{this.props.buttonText}</button>)
    }
}
```

### state를 handling하는 방식 차이
역사적으론 Class를 통해서 state를 관리하는데 hooks를 통해 Functional 방식을 통해서도 state를 관리 할 수 있다.
```typescript jsx

const FunctionalComponent = () => {
    const [count, setCount] = React.useState(0);
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>Increment</button>
        </div>
    );
};

class ClassComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {count: 0};
    }

    render() {
        return (
            <div>
                <p>Count: {this.state.count}</p>
                <button onClick={() => this.setState({count: this.state.count + 1})}>
                    Increment
                </button>
            </div>
        );
    }
}
```

## Functional Component 방식이 game changer가 될 수 있었던 이유
1. React Hooks 
   - 이를 통해서 Class Component만 가능하던 일을 할 수 있게 됨.
2. Stateless No More 
   - 이전에는 Functional 방식은 상태를 가질 수 없었다. 그냥 props를 받아 렌더링만 할 수 있었기 때문에 'stateless'라고 불렀다.  
     하지만 `useState`를 통해 Functional 방식도 상태를 추적할 수 있게 되었다.
3. LifeCycle
   - hooks의 등장으로 Functional 방식에서도 LifeCycle을 다룰 수 있게 되었다. `useEffect` hook을 사용하면 LifeCycle을 모방하여 Component의 부작용을 관리할 수 있습니다.