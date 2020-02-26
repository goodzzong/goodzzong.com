---
title: Connected-React-Router
date: 2020-02-26 11:02:56
category: react
---

## Connected React Router 이란?

리덕스에서 주소를 변경 및 확인하기 위해 history 객체를 관리하며 리덕스 스토어에서 애플리케이션 히스토리를 읽을 수 있으며 스토어에 액션을 디스패치하여 애플리케이션의 다른 경로로 이동할 수 있다.

## 적용 방법

적용 방법은 깃헙페이지에 자세히 나와 있다. 간단하게 설정하는 법만 적어보자.

### Step 1

- 루트 리듀서파일
  - `history`를 인자 값으로 넘기는 함수를 만들자.
  - `history`를 `connectRouter`에 전달하여 `router`에 추가하자.
  - 참고 : key 는 `router`여야 합니다.

```js
  // reducers.js
  import { combineReducers } from 'redux'
  import { connectRouter } from 'connected-react-router'

  const createRootReducer = (history) => combineReducers({
    router: connectRouter(history),
    ... // rest of your reducers
  })
  export default createRootReducer
```

### Step 2

- Redux 스토어를 만들 때
  - `history` 객체를 가져오자.
  - 작성된 `history`를 `createRootReducer`에게 할당한다.
  - `history` 액션을 `dispatch` 하려면 `routerMiddleware (history)`를 사용하십시오 (예 : push ( '/ path / to / somewhere')로 URL을 변경).

```js
    // configureStore.js
    ...
    import { createBrowserHistory } from 'history'
    import { applyMiddleware, compose, createStore } from 'redux'
    import { routerMiddleware } from 'connected-react-router'
    import createRootReducer from './reducers'
    ...
    export const history = createBrowserHistory()
    
    export default function configureStore(preloadedState) {
      const store = createStore(
        createRootReducer(history), // root reducer with router state
        preloadedState,
        compose(
          applyMiddleware(
            routerMiddleware(history), // for dispatching history actions
            // ... other middlewares ...
          ),
        ),
      )
    
      return store
    }
```
### Step 3

- `ConnectedRouter`로 react-router v4 / v5 라우팅 을 감싸고 `history` 객체를 `prop`으로 전달하자. `BrowserRouter` 또는 `NativeRouter` 사용을 삭제하면 상태를 동기화 하는 데 문제가 발생할 수 있다.
- `ConnectedRouter`를 `react-redux`의 `Provider` 하위 항목으로 넣어주자.
- N.B. 서버 측 렌더링을 수행하는 경우 여전히 서버의 `react-redux`에서 `StaticRouter`를 사용해야 한다.

```js
    // index.js
    ...
    import { Provider } from 'react-redux'
    import { Route, Switch } from 'react-router' // react-router v4/v5
    import { ConnectedRouter } from 'connected-react-router'
    import configureStore, { history } from './configureStore'
    ...
    const store = configureStore(/* provide initial state if any */)
    
    ReactDOM.render(
      <Provider store={store}>
        <ConnectedRouter history={history}> { /* place ConnectedRouter under Provider */ }
          <> { /* your usual react-router v4/v5 routing */ }
            <Switch>
              <Route exact path="/" render={() => (<div>Match</div>)} />
              <Route render={() => (<div>Miss</div>)} />
            </Switch>
          </>
        </ConnectedRouter>
      </Provider>,
      document.getElementById('react-root')
    )
```

## 참고

- [[https://github.com/supasate/connected-react-router](https://github.com/supasate/connected-react-router)]([https://github.com/supasate/connected-react-router](https://github.com/supasate/connected-react-router))