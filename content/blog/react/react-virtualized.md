---
title: react-virtualized
date: 2020-02-26 10:02:35
category: react
---

## 간단 소개
리액트 성능 최적화 방법을 위한 라이브러리 중 하나이다.

데이터가 많은 리스트와 표를 효율적으로 랜더링 할 수 있도록 도와주는 라이브러리이다.

만약에 데이터가 2000개고 노출은 9개만 되고 나머진 스크롤 할 때 보여지는 화면일 때 첨부터 2000개를 모드 렌더링 하는건 낭비이다. 

스크롤되기 전에 보이지 않는 컴포넌트는 렌더링하지 않고 크기만 차지하게끔 하고 스크롤 되면 해당 스크롤 위치에서 보여 주는 방식으로 최적화 할 수 있다.

## Getting started

npm을 이용하여 사용하여 설치해 준다.
```js
npm install react-virtualized --save
```
그리고 나서 다음과 같이 가져 올 수 있다.
```js
import List from 'react-virtualized/List';

<List {...props} />
```
그 후에는 상황에 맞게 사용하면 된다.

지금은 제공해주는 것 중에 List 컴포넌트를 사용 하였기 때문에 그걸 예시로 쓰지만 상황에 맞게 Table, Grid 등등을 사용하면 된다.

List 컴포넌트를 사용할 때는 해당 리스트의 전체 크기와 각 항목의 높이, 각 항목을 랜더링할 때 사용해야 하는 함수, 그리고 배열을 props로 넣어 주어야 한다.


예제는 TdoList를 사용하였고 각각의 항목 컴포넌트는 TodolistItem이다.
```js
// TodoList.js

import React, { useCallback } from 'react';
import { List } from 'react-virtualized';
import TodoListItem from './TodoListItem';

const TodoList = ({ todos, onToggle, onRemove }) => {
	const rowRenderer = useCallback(
		({ index, key, style }) => {
			const todo = todos[index];
			return (
				<TodoListItem
					todo={todo}
					key={key}
					onRemove={onRemove}
					onToggle={onToggle}
					style={style}
				/>
			);
		}, [onRemove, onToggle, todos],
	);

	return (
		<List
			className="TodoList"
			width={512}
			height={513}
			rowCount={todos.length}
			rowHeight={57}
			rowRenderer={rowRenderer}
			list={todos}
			style={{ outline: 'none' }}
		/>
	);
};

export default React.memo(TodoList);
```

## 참고

- 리액트를 다루는 기술
- [https://github.com/bvaughn/react-virtualized](https://github.com/bvaughn/react-virtualized)