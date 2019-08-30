---
title: 행렬의 덧셈 - 프로그래머스 알고리즘 문제풀이 
date: 2019-08-30 10:08:60
category: Algorithm
---

## 문제
> 행렬의 덧셈은 행과 열의 크기가 같은 두 행렬의 같은 행, 같은 열의 값을 서로 더한 결과가 됩니다. 2개의 행렬 arr1과 arr2를 입력받아, 행렬 덧셈의 결과를 반환하는 함수, solution을 완성해주세요.

처음에는 중첩 반복문을 이용해 풀었다.

```js
function solution(arr1, arr2) {

		let result = [];
	
		for (let i = 0; i < arr1.length; i++) {
			result[i] = [];
			for (let j = 0; j < arr1[i].length; j++) {
				result[i].push(arr1[i][j] + arr2[i][j])
			}
		}

	return result

}
```

그 다음으로는 배열의 map 함수를 이용해 리팩토링 했다.

```js

describe('배열 더하기 테스트', () => {
	it('테스트', () => {
		const arr1 = [[1, 3], [2, 4]];
		const arr2 = [[3, 2], [5, 2]];
		const result = [[4, 5], [7, 6]];
		expect(solution(arr1, arr2)).toEqual(result);
	})
})

function solution(arr1, arr2) {
	return arr1.map((a, i) => a.map((v, j) => v + arr2[i][j]))
}

```
map함수의 첫번째 파라미터는 배열의 반복되는 인덱스의 값이고, 두번째 파라미터는
그 반복되는 배열의 인덱스이다. 
각 배열을 반복하며 로직을 처리하고 리턴되는 값은 새로운 배열을 돌려준다.