---
title: Spread 와 Rest 에 대해 알아보자.
date: 2019-09-09 21:09:21
category: javascript
---

리액트 프로젝트를 진행하거나 어떤 자바스크립트 프로젝트를 진행 할 때 상태관리를 위해 리덕스를 자주 활용한다.  그럴때 리덕스에서 리듀서를 작성하게 되는데, 그때 Spread 문법을 활용하여 불변성을 유지하면서 리듀서를 작성한다. 

그때마다 Spread를 활용하는게 헷갈려서 이번에 개념에 대해 간단하게 정리하려 한다.

### Spread 문법

es6에서  추가된 문법이고 객체 or 배열을 펼칠수 있다. 

기존에 썼던 방식과 비교해보자.
```js
  const number = [1, 2, 3, 4, 5]
```
기존에 위의 배열에 값을 추가한다면
```js
	const newNumber = [];
										
	for ( let i = 0 ; i < number; i ++) {
		newNumber.push(number[i]);
	}
	newNumber.push(6) // [1, 2, 3, 4, 5, 6]
```
이런식으로 새로운 배열을 생성하여 그곳에 push 함수를 활용하여 값을 추가 했을 것이다.

Spread 문법을 사용하면 이를 쉽게 할 수 있다.
```js
  const newNumber = [...number, 6] // [1, 2, 3, 4, 5, 6]
```
이렇게 하면 새로운 배열형태로 리턴한 값을 변수에 담을 수 있다. 기존의 배열은 변하지 않는다.

이런식으로 객체를 손쉽게 병합 또는 변경 할 수 있다. 

### Rest 문법

es6에 추가된 문법이고 객체 or 배열 그리고 함수의 파라미터에서 사용 가능하다.

함수에 전달된 인수들의 목록을 배열로 전달 받는다.

이름 그대로 먼저 선언된 파라미터나 변수들을 제외하고 나머지 항목들이 배열 형태로 저장된다.

객체에서 rest 활용
```js
  const person = {
    name: '아무개',
    nickname: '멋쟁이',
    age : 20,
  }
  
  const { nickname, ...rest} = person;
  console.log(nickname) // "멋쟁이"
  console.log(rest) // {name: "아무개", age: 20}
```  

비구조화 할당 문법과 함께 사용 하여 표현 할 수 있다.

이름은 굳이 rest로 안하고 다르게 해도 상관은 없다.

배열에서 rest 활용
```js
  const number = [1, 2, 3, 4, 5];
	const [ one, two, ...rest] = number;
    
  console.log(one) // 1
  console.log(two) // 2
  console.log(rest) // [3, 4, 5]
```

rest 문법은 나머지 항목들을 저당 되기 때문에 다음과 같이 할 수 없다.
```js
  const number = [1, 2, 3, 4, 5];
  const [ ...rest, five ] = number;
```