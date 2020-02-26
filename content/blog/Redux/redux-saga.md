---
title: Redux-saga
date: 2020-02-26 11:02:87
category: Redux
---

`다음 글은 번역글을 읽고 필자가 보기 위해 옮겨 적은 내용입니다.`


`redux-saga` 는`TASK` 라는 개념을 `Redux` 로 가져오기 위한 지원 라이브러리입니다.

여기서 말하는  `Task` 란 일의 절차와 같은 독립적인 실행 단위로써, 각각 평행적으로 작동합니다.

`redux-saga` 는 이 `Task` 의 실행환경을 제공합니다.

더불어 비동기 처리를 `TASK` 로써 기술하기 위한 준비물인 `Effect` 와 비동기처리를 동기적으로 표현하는 

방법을 제공하고 있습니다. 

`Effect` 란 `TASK` 를 기술하기 위한 커맨드(명령)와 같은 것으로, 다음과 같은 것들이 있습니다.

- `select`: State로부터 필요한 데이터를 꺼낸다.
- `put`: Action을 dispatch한다.
- `take`: Action을 기다린다. 이벤트의 발생을 기다린다.
- `call`: Promise의 완료를 기다린다.
- `fork`: 다른 Task를 시작한다.
- `join`: 다른 Task의 종료를 기다린다.
- ...

`TASK` 안에 직접 실행할 수 있는 것도 있지만, `redux-saga` 에게 부탁하여 간접적으로 실행합니다.

비동기처리를 `co`처럼 동기적으로 쓸 수 있게 하고, 복수의 `Task`를 동시평행적으로 실행하는 것이 가능합니다.

### 무엇이 좋아지나?

- Mock 코드를 많이 쓰지 않아도 된다.
- 작은 코드로 더 분할할 수 있다.
- 재이용이 가능해진다.

### redux-thunk → redux-saga(FetchAPI를 사용한 통신처리)

데이터를 읽어들이는건 간단하지만, `Redux` 로 제대로 생각해야할게 적지 않습니다.

다음과 같은 점들입니다.

- 어디서 통신처리를 적을 것인가
- 어디로부터 통신처리를 불러올 것인가
- 통신처리의 상태를 어떻게 가지게 할 것인가.

3번째 요소는 `redux-thunk`나 `redux-saga` 둘다 공통적인 부분입니다.

통신처리가 완료되기까지 "읽어들이는 중..."이라는 메세지를 표시하기 위해서는, 

통신상태를 Store로 가지게 한 다음에, 통신의 `개시/성공/실패`의 3 타이밍에 Action을 dispatch하여 

상태를 바꿀 필요가 있습니다.

### redux-thunk

```js
    // api.js
    export function user(id) {
      return fetch(`http://localhost:3000/users/${id}`)
        .then(res => res.json())
        .then(payload => ({ payload }))
        .catch(error => ({ error }));
    }

    //action.js
    export function fetchUser(id) {
      return dispatch => {
        dispatch(requestUser(id));
        API.user(id).then(res => {
          const { payload, error } = res;
          if (payload && !error) {
            dispatch(successUser(payload));
          } else {
            dispatch(failureUser(error));
          }
        });
      };
    }
```

전체의 흐름

1. 누군가가 `fetchUser` 함수의 리턴 값을 `dispatch` 한다.
2. `redux-thunk` 의 Middleware 가 `dispatch`된 함수를 실행한다.
3. 통신처리를 개시전 `REQUEST_USER` `Action`을 `dispatch` 한다.
4. `API.user` 함수를 불러내서 통신처리 개시한다.
5. 완료되면 `SUCCESS_USER` 혹은 `FAILURE_USER` `Action` 을 `dispatch` 한다.

본래의 `Action Creator`는 `Action 오브젝트`를 생성하여 돌려줄 뿐이었기에,생성한 `Action 오브젝트`를 

`dispatch`하는 데다가, 앞뒤로 이런저런 로직이 들어가는건 위험한 냄새가 납니다. 도입도 사용법도 간단한 반면, 

사태파악도 안된채로 편하니까 막쓰다간 나중에 지옥을 맞이할지도 모릅니다. 

소극적으로 사용한다면 문제가 없겠지만, 복잡한 통신처리는 정말로 쓰고 싶지 않습니다. 

쓰고 싶다는 생각도 안됩니다.

### redux-saga
```js
    //sagas.js
    function* handleRequestUser() {
      while (true) {
        const action = yield take(REQUEST_USER);
        const { payload, error } = yield call(API.user, action.payload);
        if (payload && !error) {
          yield put(successUser(payload));
        } else {
          yield put(failureUser(error));
        }
      }
    }
    
    export default function* rootSaga() {
      yield fork(handleRequestUser);
    }
```

전체적인 흐름

1. redux-saga의 Middleware가 `rootSaga` Task를 시작시킨다.
2. `fork` Effect로 인해 `handleRequestUser` Task가 시작된다.
3. `take` Effect로 `REQUEST_USER` Action이 dispatch되길 기다린다.
4. *(누군가가 `REQUEST_USER` Action을 dispatch한다.)*
5. `call` Effect로 `API.user` 함수를 불러와서 통신처리의 완료를 기다린다.
6. *(통신처리가 완료된다.)*
7. `put` Effect를 사용하여 `SUCCESS_USER` 혹은 `FAILURE_USER` Action을 dispatch한다.
8. while 루프에 따라 3번으로 돌아간다.

`redux-thunk`로 인한 코드와의 비교를 위해 일련의 흐름을 써보았지만, 실은 이 처리로는 동시병행하여 

움직이는 2개의 흐름이 있습니다. 그것이 `Task`입니다. 

`sagas.js`로 정의된 2가지 함수 모두 `redux-saga`의 `Task`입니다. 하나씩 살펴봅시다.

- `rootSaga Task`는 Redux의 Store가 작성 된 후, redux-saga의 Middleware가 기동 될 때 `1번만` 불러와집니다.
- `fork Effect`을 사용하여 `redux-saga`에게 다른 `Task`를 기동할 것을 요청
- `Task`내에는 실제 처리를 행하지 않습니다. 기동만 할 뿐입니다.
- `fork`함수로부터 생성된 것은 단순한 오브젝트
- Effect 오브젝트는 생성될뿐 아무것도 하지 않으므로 `redux-saga` 로 넘겨져서 실행될 필요가 있습니다.
- 그러기 위해서 `제너레이터` 함수의 `yield` 를 사용해 불러오는 코드 값을 넘겨주고 있습니다.
- `불러오는 쪽의 코드` 는 누구의 것인가? `redux-saga`가 제공하는 `Task`의 `실행환경`인 `Middleware`입니다.
- `rootSaga` 에서 넘겨진 새로운 `Task` 로써 기동시킵니다.
- 그 이후 `redux-saga`는 2개의 `Task`가 동시에 움직이고있는 상태가 됩니다.
- 새롭게 기동된 `handleRequestUser Task`의 설명으로 넘어가기 전에 `rootSaga Task`의 `그 이후`를 따라갑니다.
- `fork Effect`는 지정한 Task의 완료를 기다리지 않습니다.
- 그러므로 `yield`는 블록되지않고 곧 제어로 돌아옵니다.
- `rootSaga Task`는 `handleRequestUser Task`의 기동 이외에 할 일이 없습니다.
- 그때문에 `rootSaga Task`내에는 `fork`를 사용하여 기동된 모든 Task가 `종료 될 때까지 기다립니다.`
- `handleRequestUser Task`를 기동시키면 금방 `REQUEST_USER` Action을 기다리기 위해 `take Effect`가 불러와집니다.
- 이 `기다림`이라는 행동이 비동기처리를 동기적으로 쓴다 라는 특징적인 Task의 표현
- `redux-saga`의 `Task`를 `Generator`함수로 쓰는 `이유`는 `yield` 에 따라 처리의 흐름을 `일시정지`하기 때문입니다.
- 이러한 체계 덕분에 `싱글 스레드`의 Javascript로 복수의 Task를 만들어, 각각 특정한 Action을 기다리거나, 통신처리의 결과를 기다려도 처리가 밀리지 않게 됩니다.
- 그리고 그 처리가 완료되면 일시정지된 코드가 제개되고, dispatch된 Action 오브젝트를 돌려줍니다
- 그리고 곧 `API` 불러냅니다.
- 여기서 `call` Effect를 사용합니다. 지정된 함수가 `Promise`를 돌려줄 경우, 그 Promise 가 `resolve`되고 나서 제어를 돌려줍니다.
- 통신처리가 완료하면 다시 한번 `handleRequestUser Task`로 `제어`를 돌려줍니다.
- 결과에 따라 Action을 dispatch합니다. Action의 dispatch에는 `put` Effect를 사용합니다.

**완료된 Task의 의문점: whil문의 존재**

- `handleRequestUser Task`는 전체가 while문으로된 무한 루프로 감싸여 있습니다.
- 그 결과 put Effect로 Effect로 Action을 dispatch한 후, `루프의 처음`으로 돌아가서 다시 한번 take Effect로 REQUEST_USER Action을 기다리게 됩니다.
- 즉, `Action`을 기다려 통신처리를 할 뿐인 `Task`가 됩니다. (재사용 가능)
    - 재사용 가능해지므로써 버그가 준다.
    - 단순해진다. (콜백지옥, 깊은 구조, 뜬근없는 Promise가 사라지게 됨)

### 어떻게 바뀌었나?

 처음에 얘기한 `어디에 쓸 것 인가`, `어디에서 불러올 것인가`에 대한 말을 해보겠습니다.

- `redux-thunk`는 `Action Creator`가 함수를 넘겨주기에 필연적으로 `Action Creator`에 `비동기처리`의 코드나 관련된 로직이 (약간 무리하게) 들어갑니다.
- `redux-saga`는 `비동기 처리`를 기술하는 전용의 방식인 `Task`로 쓰여있습니다.
- 그 결과, `Action Creator`는 본래의 모습을 되찾아, `Action 오브젝트`를 생성하여 돌려주는 `순수한 상태`로 돌아갑니다.

### 처리를 복잡하게 해보자( 다른 예제)

    어떤 통신처리가 끝난 이후, 그 결과에 따라 다시 통신처리를 개시하는 것입니다. 
    이번 예제에서는 유저 정보를 취득한 이후, 유저정보에 포함된 지역명을 사용하여 같은 지역에 살고 있는 
    다른 유저를 검색하여 제안하는 기능을 추가해봅니다.

**redux-thunk**

```js
    // api.js
    export function searchByLocation(name) {
      return fetch(`http://localhost:3000/users/${id}/following`)
        .then(res => res.json())
        .then(payload => { payload })
        .catch(error => { error });
    }
    
    // actions.js
    export function fetchUser(id) {
      return dispatch => {
        // 유저 정보를 읽어들인다
        dispatch(requestUser(id));
        API.user(id).then(res => {
          const { payload, error } = res;
          if (payload && !error) {
            dispatch(successUser(payload));
    
            // 체인: 지역명으로 유저를 검색
            dispatch(requestSearchByLocation(id));
            API.searchByLocation(id).then(res => {
              const { payload, error } = res;
              if (payload && !error) {
                dispatch(successSearchByLocation(payload));
              } else {
                dispatch(failureSearchByLocation(error));
              }
            });
          } else {
            dispatch(failureUser(error));
          }
        });
      };
    }
```

뭘 하고픈지는 압니다. 아마 보통 이렇게 되겠죠. 하지만 ...한(뭔가 찝찝한) 느낌이 드네요.

여기에 또 다른 체인을 늘리거나 체인시키는 위치를 바꾸게 된다면 곤란해 질듯 합니다.

무엇보다도 기분 나쁜건 fetchUser라는 Action Creator를 불러내서 왜 `유저 검색`까지 실행하고 있느냐 라는 점입니다.

**redux-saga**

```js
    // sagas.js
    
    // 추가
    function* handleRequestSearchByLocation() {
      while (true) {
        const action = yield take(SUCCESS_USER);
        const { payload, error } = yield call(API.searchByLocation, action.payload.location);
        if (payload && !error) {
          yield put(successSearchByLocation(payload));
        } else {
          yield put(failureSearchByLocation(error));
        }
      }
    }
    
    // 변경없음!
    function* handleRequestUser() {
      while (true) {
        const action = yield take(REQUEST_USER);
        const { payload, error } = yield call(API.user, action.payload);
        if (payload && !error) {
          yield put(successUser(payload));
        } else {
          yield put(failureUser(error));
        }
      }
    }
    
    export default function* rootSaga() {
      yield fork(handleRequestUser);
      yield fork(handleRequestSearchByLocation); // 추가
    }
```

- `handleRequestUser Task`는 변경점이 없습니다.
- 새롭게 추가된 `handleRequestSearchByLocation Task`는 `handleRequestUser Task`와 거의 `동일`한 처리입니다.
- `rootSaga Task`는 `handleRequestSearchByLocation Task`를 기동하기위해 fork 를 하나 더 추가하고 있습니다.

처리의 흐름을 다음과 같습니다.

1. redux-saga의 Middleware가 `rootSaga` Task를 기동시킨다.
2. `fork` Effect에 따라 `handleRequestUser` 와 `handleRequestSearchByLocation` Task가 기동된다.
3. 각각의 Task에 대해 `take` Effect로부터 `REQUEST_USER` 와 `SUCCESS_USER` Action이 dispatch되는 것을 기다린다.
4. *(누군가가 `REQUEST_USER` Action을 dispatch한다.)*
5. `call` Effect으로 `API.user` 함수를 불러와서, 통신처리가 끝나길 기다린다.
6. *(통신처리가 완료된다.)*
7. `put` Effect를 사용하여 `SUCCESS_USER` Action을 dispatch한다.
8. `handleRequestSearchByLocation` 태스크가 다시 시작되어, `call` Effect로 `API.searchByLocation` 함수를 불러와서, 통신처리가 끝나길 기다린다.
9. *(통신처리가 완료된다.)*
10. `put` Effect를 사용하여 `SUCCESS_SEARCH_BY_LOCATION` Action을 dispatch한다.
11. 각각의 Task에서 while 루프가 처음으로 돌아가 `take` 로 Action의 dispatch를 기다린다.

- 각각의 태스크를 주의해서 보면 단순한 것 밖에 하지 않기 때문에 이해하기 쉬워집니다.
- 게다가 이 코드를 확장하여 체인을 늘리거나, 체인을 순서를 바꾸거나 무엇을 하더라도 Task는 하나만 집중하고 있으므로 다른 무엇을 해도 그다지 영향을 받지 않습니다.
- 이러한 성질을 적극적으로 이용하여 Task가 너무 비대해지기 전에 계속해서 잘라 나누는 것으로 `코드의 건전성을 유지`할 수 있습니다.

### 테스트를 써보자.

- `redux-saga`를 적극적으로 쓰고싶은 이유로서 `테스트의 편리함`
- 테스트 대상은 최초의 통신처리의 코드로 합니다.
- 먼저 단순해보이는 `rootSaga Task`의 `테스트`부터 써보겠습니다.
- 그리고, 테스트코드는 `mocha + power-assert` 입니다.

    // sagas.js
    export default function* rootSaga() {
      yield fork(handleRequestUser);
    }

여기에 대한 테스트 코드는 다음과 같습니다.

```js
    // test.js
    describe('rootSaga', () => {
      it('launches handleRequestUser task', () => {
        const saga = rootSaga();
    
        ret = saga.next();
        assert.deepEqual(ret.value, fork(handleRequestUser));
    
        ret = saga.next();
        assert(ret.done);
      });
    });
```

- Task를 포크하고 있는지 테스트를 한다. 라고 하면 어렵게 들립니다.
- 여기서 Task라는 건 단순한 Generator 함수로, Task가 돌려주는건 모두 단순한 오브젝트라는걸 떠올립시다.
- 고로 redux-saga에 두어진 Task의 테스트는 단순히 오브젝트를 비교하는 것으로 대부분 충분합니다.
- `rootSaga Task`가 포크하고 있는지를 확인하기 위해 `fork Effect`로 오브젝트를 생성하여 비교하는 것으로 OK입니다.
- **테스트해야 할 것은 이 Task가 무엇을 하고 있는 가로써, 그에 앞서 무엇을 하는가는 알 필요 없습니다.**

이것만이면 테스트한 느낌이 안나므로 좀 더 복잡한 handleRequestUser Task도 테스트로 써봅니다.

```js
    // sagas.js
    function* handleRequestUser() {
      while (true) {
        const action = yield take(REQUEST_USER);
        const { payload, error } = yield call(API.user, action.payload);
        if (payload && !error) {
          yield put(successUser(payload));
        } else {
          yield put(failureUser(error));
        }
      }
    }
```

통신처리가 성공했는지 실패했는지로 분기됩니다. 그러므로 테스트도 각각의 케이스를 적습니다.

```js
    // test.js
    describe('handleRequestUser', () => {
      let saga;
      beforeEach(() => {
        saga = handleRequestUser();
      });
    
      it('receives fetch request and succeeds to get data', () => {
        let ret = saga.next();
        assert.deepEqual(ret.value, take(REQUEST_USER)); // (A')
    
        ret = saga.next({ payload: 123 }); // (A)
        assert.deepEqual(ret.value, call(API.user, 123)); // (B')
    
        ret = saga.next({ payload: 'GOOD' }); // (B)
        assert.deepEqual(ret.value, put(successUser('GOOD')));
    
        ret = saga.next();
        assert.deepEqual(ret.value, take(REQUEST_USER));
      });
    
      it('receives fetch request and fails to get data', () => {
        let ret = saga.next();
        assert.deepEqual(ret.value, take(REQUEST_USER));
    
        ret = saga.next({ payload: 456 });
        assert.deepEqual(ret.value, call(API.user, 456));
    
        ret = saga.next({ error: 'WRONG' });
        assert.deepEqual(ret.value, put(failureUser('WRONG')));
    
        ret = saga.next();
        assert.deepEqual(ret.value, take(REQUEST_USER));
      });
    });
```

- 접근하는 방법은 `next()`를 부를때 처음의 `yield`까지 실행되는 그때의 우변의 값을 랩핑한 것이 `리턴값`으로 나옵니다.
- 우변 값 자체는 value 프로퍼티에 격납되어있으므로 거기서 확인합니다.
- 지금, Task는 정지되어있습니다. 이를 재개하기 위해서는 더욱이 `next()`를 불러냅니다.
- 이 `next()`의 인수로써 넘겨준 것은, Task가 재개될 때에 `yield`로부터 나온 `리턴 값`이 됩니다.
- 즉 코드중의 `(A)`로 넘겨지는 것이 `(A')`로부터 나올거라 기대되는 `리턴 값`이라는 것이죠.
- 마지막으로, 통신처리가 끝나면, 다시 리퀘스트를 기다리는 상태가 되었는지 확인합니다. Task를 동기적으로 썻기에, 테스트 코드도 동기적으로 되었습니다.
- 왜 redux-saga의 실행모델로 제공된 커맨드를 사용하여 Task를 만드는 것인가가 이해되었다고 생각합니다. 모든것이 `예측 가능한` 테스트가 간단하게 되어, 복잡한 목을 만들 필요성도 최소한으로 되기 때문입니다.

## 참고

- [https://github.com/reactkr/learn-react-in-korean/blob/master/translated/deal-with-async-process-by-redux-saga.md](https://github.com/reactkr/learn-react-in-korean/blob/master/translated/deal-with-async-process-by-redux-saga.md)