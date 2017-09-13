# redux-saga
- 비동기 통신의 제어를 좀 더 깔끔하고 쉽게 할 수 있도록 도와주는 라이브러리
- 동시성 패턴 Communicating Sequential Processes(CSP)에서 영감을 얻었다.

## redux-saga에서 사용할 수 있는 effect
- `put`: redux 스토어의 액션을 디스패치한다.
- `select`: 상태 일부를 가져온다.
- `call`: 다른 saga들이나 promise 등을 호출할 수 있다.
- `take`: 액션이 디스패치되기를 기다린다.
- `fork`: 서브 프로세스를 트리거한 뒤 완료를 기다리지 않고 이동한다.
- `cancel`: 포크되었던 서브 프로세스를 취소한다.
- `cancelled`: 현재 프로세스가 취소되었는지 확인한다.
- `delay`: 다음 구문으로 이동하기 전에 주어진 기간동안 대기한다. Promise를 반환한다.

## Directory
- redux-sagajs
  - src
    - internal
        - sagaHelpers
            - fsmlterator.js
            - index.js
            - takeEvery.js
            - tagkeLatest.js
            - throttle.js
        - buffers.js
        - channel.js
        - io.js
            - effect가 실행되는 부분
        - middleware.js
        - proc.js
        - runSaga.js
        - schedulers.js
        - utils.js
    - effects.js
    - index.js
    - utils.