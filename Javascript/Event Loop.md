
> ### 목차
>*이벤트 루프?*  
>*호출 스택*  
>*이벤트 루프! (feat. 이벤트 큐)*  

JavaScript는 싱글 스레드 언어입니다. 즉, 한번에 하나의 작업만을 처리할 수 있습니다.
하지만 개발하다보면 비동기 코드를 작성하게 되는데 이 코드는 동시적으로 여러 개의 작업이 수행됩니다. 싱글 스레드 언어인 JavaScript에서 이런 동시성을 어떻게 처리하는걸까요.


### 이벤트 루프
***
JavaScript 동시성에 대해 검색을 하다보면 이벤트 루프라는 용어가 항상 등장합니다.  
[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)에서도 이벤트 루프를 JavaScript의 동시성 모델의 기반이라고 설명하고 있습니다.

반면 [ECMAScript의 공식 스펙 문서](https://www.ecma-international.org/ecma-262/10.0/index.html)에 이벤트 루프라는 용어는 등장하지 않습니다.  
도대체 이벤트 루프가 무엇일까요? 이벤트 루프를 이해하려면 먼저 호출 스택을 알아야 합니다.

### 호출 스택
***
호출 스택은 JavaScript에서 실행되는 함수의 현재 위치 및 상태를 기록하는 자료 구조입니다.
JavaScript를 단일 호출 스택을 가진 언어이며 이것이 싱글 스레드 언어라고 부르는 이유입니다.
MDN에서는 호출 스택을 다음과 같이 정의하고 있습니다.
> *호출 스택은 여러 함수들(functions)을 호출하는 스크립트에서 해당 위치를 추적하는 인터프리터 (웹 브라우저의 자바스크립트 인터프리터같은)를 위한 메커니즘입니다. 현재 어떤 함수가 동작하고있는 지, 그 함수 내에서 어떤 함수가 동작하는 지, 다음에 어떤 함수가 호출되어야하는 지 등을 제어합니다.*

호출 스택의 동작을 간단하게 예제로 보겠습니다.
```javascript
function first () {
  second();
  console.log('first'); // C
}

function second () {
  console.log('second'); // B
}

setTimeout(function () {
  console.log('async'); // D
}, 0); // A

first();
```
```
[실행 결과]
second
first
timeout
```
위 코드가 실행될 때 호출 스택의 상태는 다음과 순서로 변경됩니다.

![callstack](https://user-images.githubusercontent.com/48500660/66096893-6367ff00-e5d7-11e9-9b33-dbae81d84a1b.png)


main함수는 전역 환경에서 실행되는 코드에 대한 함수입니다.
C에서 모든 코드가 실행되었기 때문에 main함수도 호출 스택에서 제거됩니다.
그 후 비동기 함수 setTimeout으로 호출한 익명 함수가 호출 스택에 쌓이며 실행됩니다.

여기서 주목할 점은 setTimeout의 지연시간을 분명 0으로 설정해놓았지만 바로 실행되지 않고 모든 코드의 실행이 종료된 뒤에 실행되었다는 것입니다. 이 현상은 JavaScript가 비동기 이벤트를 처리하는 방식에 기인합니다.

### 이벤트 루프! (feat. 이벤트 큐)
***
JavaScript에는 런타임 시점에 처리해야 할 Task를 담아놓는 이벤트 큐(Task Queue)가 존재합니다.
비동기 이벤트(DOM Event, Ajax, setTimeout 등)들의 작업이 완료된 뒤 실행될 Task(콜백 함수)들은 이벤트 큐에 쌓이게 되며 호출 스택이 비워졌을 때 이벤트 루프를 통해 하나씩 쌓인 순서대로 처리됩니다.

MDN에서는 이벤트 루프를 다음과 같은 임의 코드로 설명하고 있습니다.
```javascript
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```
위 코드의 waitForMessage() 함수는 현재 실행중인 Task가 있는지, 이벤트 큐에 Task가 있는지 체크합니다. 만약 실행 중인 Task가 없고 이벤트 큐에 대기 중인 Task가 있다면 이벤트 큐에서 꺼내 호출 스택에 쌓고 실행합니다. 이 반복적인 작업을 이벤트 루프라고 합니다. 그림으로 표현하면 다음과 같습니다.

![eventloop](https://user-images.githubusercontent.com/48500660/66096899-67941c80-e5d7-11e9-8f7c-175788e29327.png)
