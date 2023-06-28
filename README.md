
## react-redux 사용법



### 1. state 종속성 탈피

기존의 useState를 사용할 경우 컴포넌트 내부에 state를 만들고, 함수로 state를 바꿈.

그렇기 떄문에 state는 컴포넌트에 종속되는 결과를 낳는다.

redux는 컴포넌트에 종속되지 않고, 상태관리를 컴포넌트 바깥에서 함.

프로젝트 루트레벨에서 store라는 곳에 state를 저장하고, 모든 컴포넌트는 store에 subscribe를 하면서 state와 그 state를 바꾸는 함수를 전달 받게 된다. 해당 함수를 바꿈으로 state가 바뀌면 해당 state를 바라보고 있는 컴포넌트는 모두 리렌더링 됨.

### 2. props -> props -> props 지옥 탈출

개발자가 원하는 state가 자식의 자식의 자식에서 사용이 되어서 props를 내리고 또 내리고 또 내리면 해당 state 사용에 있어서 혼동이 오고 실수가 잦아진다.

위에서 본 대로, redux의 store는 프로젝트 루트레벨에 위치하고, 해당 store를 subscribe하는 컴포넌트는 모두 state와 state를 handling하는 함수를 받을 수 있게 된다.

즉 어느 위치에 있든 상관 없이 단 한번에 상태를 받을 수 있게 된다.

### 3. redux 기본 원리

redux는 기본적을 flux 패턴을 따른다.

### 4. react에서 redux 사용하기

- 사용 할 패키지를 설치한다.
- **redux-devtools-extension**은 크롬 확장 프로그램에서 **redux dev tools**를 통해 설치 할 수 있고, redux의 데이터 흐름을 알아보기 쉽게 하기 위해 사용한다.
- **redux-logger**는 redux를 통해 바뀔 이전 state, dispatch 실행으로 인해 바뀐 state가 콘솔에 찍혀 디버깅 쉽게 해주는 러이브러리이다.

    npm install redux react-redux redux-devtools-extension redux-logger

### 5. reducer 정의

- reducer는 **store**에 들어갈 state와 state를 handling할 함수를 정의하는 곳이다.
- 기본적으로 순수함수로 코딩하고, 불변성을 지켜야 한다.

#### 중요 ! - 불변성을 지켜야하는 이유

- **불변성을 지켜야하는 이유는** redux는 이전 state와 바뀐 state를 구분하는 방법이 참조값이 바뀌었는지 확인하고, 참조값이 바뀌면, state가 바뀌었다고 redux가 인식하여, 해당 state를 사용하는 컴포넌트에게 리렌더링을 요청하기 떄문이다.
    - 그렇기 떄문에, *state.test = action.test*와 같이 직접적으로 state를 변경하면 참조값이 변하지 않아 redux는 값이 바뀌었다고 인식하지 않고 리렌더링 되지 않는다.
    - *state.test = { …test, action.test }*
    - 또는 *immer*라는 라이브러리를 사용하여 쉽게 불변성을 유지한다.

다시 reducer 함수 정의로 돌아와서 

##  *rootReducer를 정의한다.*

    // reducers/index.js

    /** root reducer */
    import { combineReducers } from "redux";
    import counter from "./counter";

    // 여러 reducer를 사용하는 경우 reducer를 하나로 묶어주는 메소드입니다.
    // store에 저장되는 리듀서는 오직 1개입니다.
    const rootReducer = combineReducers({
      counter
    });

    export default rootReducer;

## *세부 reducer를 정의한다.*

    // reducers/counter.js

    // reducer가 많아지면 action상수가 중복될 수 있으니
    // 액션이름 앞에 파일 이름을 넣습니다.
    export const INCRESE = "COUNT/INCRESE";

    export const increseCount = count => ({ type: INCRESE, count });

    const initalState = {
      count: 0
    };

    const counter = (state = initalState, action) => {
      switch (action.type) {
        case INCRESE:
          return {
            ...state,
            count: action.count
          };

      // default를 쓰지 않으면 맨처음 state에 count값이 undefined가 나옵니다 꼭! default문을 넣으세요
          default:
          return state;
      }
    };
 
## *app에 store 넣고 , 만든 reducer 반영*

    // index.js
    import React from "react";
    import ReactDOM from "react-dom";
    import { createStore, applyMiddleware, compose } from "redux";
    import { Provider } from "react-redux";
    import logger from "redux-logger";
    import { composeWithDevTools } from "redux-devtools-extension";

    import App from "./App";
    import rootReducer from "./reducers";

    // 배포 레벨에서는 리덕스 발동시 찍히는 logger를 사용하지 않습니다.
    const enhancer =
      process.env.NODE_ENV === "production"
        ? compose(applyMiddleware())
        : composeWithDevTools(applyMiddleware(logger));

    // 위에서 만든 reducer를 스토어 만들때 넣어줍니다
    const store = createStore(rootReducer, enhancer);

    ReactDOM.render(
      // 만든 store를 앱 상위에 넣어줍니다.
      <Provider store={store}>
        <App />
      </Provider>
      document.getElementById('root'),
    );

## *컴포넌트에서 redux 사용하기*

    import { useSelector, useDispatch } from "react-redux";
    import { increseCount } from "reducers/count";

    // dispatch를 사용하기 위한 준비
    const dispatch = useDispatch();

    // store에 접근하여 state 가져오기
    const { count } = useSelector(state => state.counter);

    const increse = () => {
      // store에 있는 state 바꾸는 함수 실행
      dispatch(increseCount());
    };

    const Counter = () => {
      return (
        <div>
          {count}
          <button onClick={increse}>증가</button>
        </div>
      );
    };

    export default Counter;

- 위 처럼 store에서 **useDispatch**, **useSelector**로 state와 함수 가져와서 적재적소에 호출해주면 된다.


### 6. 보완점

redux의 함수는 **무조건** 동기적으로 데이터가 흘러간다. 

하지만 웹은 언제나 비동기로 사용자 경험을 높이는 것이 중요하다.

그래서 나온것이 *redux-saga*이다.

redux를 사용하면서 redux-saga도 동시에 사용함으로 비동기의 유연함도 같이 가져갈 수 있다.
