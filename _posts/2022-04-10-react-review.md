---
layout: post
title: "React 한발자국"
categories: React
author: "Yongchan Hong"
---
# React 한발자국
어쩌다 보니 분석 플랫폼 Rivendell을 만들면서 React를 좀 많이 사용하게 되었다. 아무래도 이쁘게 Front를 만들고 배우기 쉬운게 React라 선택하게 되었는데, 후회 없이 사용하고 있다. 나 스스로에 대한 기록이 될겸 기본 컨셉 등을 기록해보고자 한다. 

## React란
React는 페이스북 (현 메타)에서 제공해주는 자바스크립트 라이브러리로, UI를 만들기 위해 사용되는 웹 인터페이스다. 웹/앱의 View를 만드는데 자주 사용된다.

### React의 특징
1. Declartive  
React는 Declartive (선언형) 성격에 맞게 컴포넌트를 얻기 위해 `<tag></tag>`문법으로 구현하고 얻기 위한 알고리즘은 구현하지않는다.  

2. Component-based  
React는 컴포넌트 단위로 개발이 된다. 여기서 컴포넌트란 독립적인 단위의 소프트웨어 모듈을 의미한다. React는 View를 여러 컴포넌트로 쪼개서 만드는데 이 말인 즉슨 한 페이지에 각 부분을 독립된 컴포넌트로 만들고 이를 조립해 화면을 채운다. 따라서 코드를 파악하기가 매우 쉽고 컴포넌트를 import하여 사용할 수 있기에 재사용성이 높으며 유지보수가 간편해진다. 

3. Virtual DOM  
DOM은 Document Object Model로서 html, css 등을 트리구조로 인식하고 랜더링 될때 쓰인다. React는 이벤트가 발생할때 가상의 DOM을 만들고 실제 DOM과 비교해 변경이 필요한 최소한의 변경사항만 실제 DOM에 반영하여 앱의 효율성과 속도를 증진 시킨다.

4. Props and State  
Props와 State는 React의 기본 개념 중 하나이다.  
- Props
부모 컴포넌트에서 자식 컴포넌트에 주는 데이터. 읽기 전용 데이터 같은 형태로 자식에게 전달이 된다.
- State
컴포넌트 내부에서 설정하며 값을 바꿀 수 있다. 동적인 데이터를 사용할때 필수적이다.

5. JSX  
Javascript를 확장한 문법을 사용한다.  
예시)  
const element = <h1>Hello, world!</h1>;

## React의 핵심 모듈  
React의 핵심 모듈은 크게 ReactDOM과 React라 할 수 있다.
ReactDOM은 `import ReactDOM from 'react-dom'`을 통해 import하며, ReactDOM.render를 통해 무엇을 어디에 그릴지 지정해준다. 보통 App을 Render하는데 자주 쓰인다.
React는 component를 만들때 쓰인다. 

## React 시작하기
지정한 폴더 내에서 `npx create-react-app .`을 통해 쉽게 시작할 수 있다. 그 이후 npm ci (npm install과 비슷하지만 package-lock.json을 기반으로 설치하고 package.json의 버전과 같아야함), npm run build (배포를 위해 build), npm run start로 시작 가능하다. 

## React Hook
Hook은 class를 작성할 필요 없이 state나 다른 feature를 사용할 수 있게 해준다. 대표적으로 있는 몇가지를 소개하자면...
1. useState  
```
const [count, setCount] = useState(0);
```
다음과 같이 component의 상태를 관리할 수 있다. 예시에서는 useState(0)을 통해 count를 0으로 초기화하고, 이후 setCount를 이용하여 변경할 수 있다.

2. useEffect  
useEffect는 componentDidMount와 componentDidUpdate, componentWillUnmount가 합쳐진 것으로 컴포넌트가 랜더링 될때마다 특정 작업을 수행하도록 한다.
```
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // count가 바뀔 때만 effect를 재실행합니다.
```
다음과 같은 코드에서는 처음 랜더링 될때와 count가 변경될때 랜더링이 된다. 


다음에는 Flask에 대해 다뤄보고, 이 둘을 통합하는 과정에 대해 이야기 해보겠다.

### Reference
https://velog.io/@kim-jaemin420/React-%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90  
https://velog.io/@jini_eun/React-React.js%EB%9E%80-%EA%B0%84%EB%8B%A8-%EC%A0%95%EB%A6%AC  
https://velog.io/@velopert/react-hooks  
https://ko.reactjs.org/docs/hooks-state.html  
https://ko.reactjs.org/docs/hooks-effect.html  
