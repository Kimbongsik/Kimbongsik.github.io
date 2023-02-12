---
title: "[웹/React] props로 컨포넌트에 값 전달"
last_modified_at: 2023-01-31T16:20:02-05:00
categories:
  - web

tags:
  - web
---

## 기본 사용법

### props

:properties의 줄임말. 어떤 값을 컴포넌트에게 전달할 때 props를 사용함. 

<br/>

- App.js

```jsx
import React from 'react'; #React 불러오기
import Hello from './Hello';

function App(){
	return (
		<Hello name="react" /> 
	);
}

export default App; # App이라는 컴포넌트를 내보내기. 다른 컴포넌트에서 불러와서 사용할 수 있음.
```

- Hello.js

```jsx
import React from 'react';

function Hello(props){
	return <div>안녕! {props.name}</div>
}

export default Hello;
```

→ Hello 컴포넌트에 name 값 전달

→ props는 파라미터를 통해 조회할 수 있음 

→props는 객체 형태로 전달

→ name 값 조회 시, props.name을 조회

<br/>

## 비구조화 할당

- App.js

```jsx
import React from 'react';
import Hello from './Hello';

function App(){
	return (
		<Hello name="react" color="red"/>
	);
}

export default App;
```

- Hello.js

```jsx
import React from 'react';

function Hello(props){
	return <div style={{color: props.color}}>안녕!{props.name}</div>
}

export default Hello;

```

→ props 내부 값을 조회할 때마다 props. 를 입력하고 있는데.. 비구조화 할당을 쓰면~ 아래와 같다. 

- Hello.js (그런데 이제 비구조화 할당을 곁들인..)

```jsx
import React from 'react';

function Hello({ color, name }){
	return <div style={{ color }}>안녕! {name}</div?
}

export default Hello;
```

<br/>

참고 사이트: 

[https://react.vlpt.us/basic/05-props.html]

[https://velopert.com/3480]
