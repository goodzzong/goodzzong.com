---
title: 프린터(스택/큐) - 프로그래머스 알고리즘 문제풀이 
date: 2019-08-30 10:08:60
category: Algorithm
---

## 문제 설명

일반적인 프린터는 인쇄 요청이 들어온 순서대로 인쇄합니다. 그렇기 때문에 중요한 문서가 나중에 인쇄될 수 있습니다. 이런 문제를 보완하기 위해 중요도가 높은 문서를 먼저 인쇄하는 프린터를 개발했습니다. 이 새롭게 개발한 프린터는 아래와 같은 방식으로 인쇄 작업을 수행합니다.

```text
	1. 인쇄 대기목록의 가장 앞에 있는 문서(J)를 대기목록에서 꺼냅니다.
	2. 나머지 인쇄 대기목록에서 J보다 중요도가 높은 문서가 한 개라도 존재하면 J를 대기목록의 가장 마지막에 넣습니다.
	3. 그렇지 않으면 J를 인쇄합니다.
```

예를 들어, 4개의 문서(A, B, C, D)가 순서대로 인쇄 대기목록에 있고 중요도가 2 1 3 2 라면 C D A B 순으로 인쇄하게 됩니다.

내가 인쇄를 요청한 문서가 몇 번째로 인쇄되는지 알고 싶습니다. 위의 예에서 C는 1번째로, A는 3번째로 인쇄됩니다.

현재 대기목록에 있는 문서의 중요도가 순서대로 담긴 배열 priorities와 내가 인쇄를 요청한 문서가 현재 대기목록의 어떤 위치에 있는지를 알려주는 location이 매개변수로 주어질 때, 내가 인쇄를 요청한 문서가 몇 번째로 인쇄되는지 return 하도록 solution 함수를 작성해주세요.


## 풀이 과정

- 일단 문제를 확인하고 간단하게 어떤식으로 풀이과정을 진행할지 생각해 보고 적어보았다.
- TDD로 작성을 하기 위해 간단한 테스트 케이스를 작성하였다.
- 그 후에 테스트를 통과하기 위해 문제를 풀고, 최대한 리팩토링 가능하게 작성해 보았다.
- 처음에 문제를 잘못 이해하여 문제 제출을 했을때 자꾸 실패를 하였는데 문제를
	잘 인지해야겠다고 생각했다.

```js

function print(priorities = undefined, location = undefined) {
	if (!priorities || !priorities.length) return null;

	const printList = priorities.map((v, i) => ({
		check: i === location,
		value: v
	}));
	const result = [];
	const printMe = x => x['check'];

	while (printList.length > 0) {
		let first = printList.shift();
		if (printList.some(val => val['value'] > first['value'])) {
			printList.push(first);
		} else {
			result.push(first);
		}
	}
	//console.log(result);
	return result.findIndex(printMe) + 1;
}

test("Print", () => {

	expect(print([])).toBeNull();
	expect(print([1, 2, 3, 4], 0)).not.toBeNull();
	expect(print([2, 1, 3, 2], 2)).toEqual(1);
	expect(print([1, 1, 9, 1, 1, 1], 0)).toEqual(5);
	expect(print([1, 2, 3, 4, 5], 2)).toEqual(3);

});
```
해당 location을 체크하기 위해 인자로 넘어온 배열을 객체형태로 변경했다.
그리고 나서 1순위로 프린트 되야할 값을 뽑아 새로운 배열에 저장한다.
이 방법을 인자로 넘어온 배열의 값이 하나도 안남을때 까지 반복한다.
완료되고 나면 새로운 배열에 들어간 값중에 location의 위치를 확인후 리턴해준다.


[문제링크](https://programmers.co.kr/learn/courses/30/lessons/42587#)