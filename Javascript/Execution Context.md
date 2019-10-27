> ## 목차
>  
> 실행 문맥(Execution Context)이란?  
> Execution Context Stack  
> this

<br/>

## **실행 문맥(Execution Context)이란?**

말 그대로 JavaScript가 실행되는 환경을 일컫는 개념입니다.

모든 실행 문맥은 다음과 같이 구성되어 있습니다.

-   code evaluation state(해당 실행 문맥의 실행, 중단 등에 필요한 모든 상태를 가지는 컴포넌트)
-   Relam(활성 객체를 가진 컴포넌트)
-   Function(함수 실행 문맥일 때 해당 함수 객체를 가지는 컴포넌트)
-   ScriptOrModule(해당 실행 문맥의 스크립트 요약 정보와 연관된 모듈 정보를 가지는 컴포넌트)

그리고 ECMAScript Code의 실행 문맥은 추가적으로 아래의 컴포넌트를 가지며 해당 포스팅에서 중점적으로 알아볼 컴포넌트입니다.

-   LexicalEnvironment
-   VariableEnvironment

### **- LexicalEnvironment**

해당 실행 문맥에서 생성된 변수 및 함수의 식별자(변수나 함수를 선언할 때 명시하는 이름)를 정의하는데 사용됩니다.

보통 LexicalEnvironment는 함수 선언, 블록, try-catch문 등과 같은 ES의 특정 문법 구조와 관련되어 있으며 이러한 코드들이 실행될 때마다 새로 생성됩니다. 또한 Environment Record와 null참조를 가질 수 있는 outer로 구성됩니다.

LexicalEnvironment는 Global Environment, Function Environment, Module Environment가 있습니다.

-   Global Environment : 이 환경의 EnvironmentRecord는 빌트인 객체, 전역 객체의 바인딩 등 식별자가 미리 채워져 있을 수도 있습니다.
-   Function Environment :  ES 함수 객체의 호출에 해당하는 LexicalEnvironment입니다. 새로운 thisBinding을 생성할 수 있으며 super 메서드를 호출하기 위한 상태도 가지고 있습니다.
-   Module Environment : 모듈의 최상위 선언에 대한 바인딩을 포함하는 LexicalEnvironment입니다. 또한, 모듈에서 명시적으로 import된 바인딩을 포함하며 outer는 Global Environment를 참조합니다.

### **- VariableEnvironment**

LexicalEnvironment와 구조는 동일합니다. 다만 여기에는 var 키워드로 선언된 변수들이 저장됩니다.

또한, 실행 문맥이 생성될 때 LexicalEnvironment와 VariableEnvironment는 초기에 동일한 값을 갖습니다.

> ### **Environment Record**

스코프 안에서 생성된 바인딩된 식별자를 저장합니다 .즉, 해당 스코프 안에서 생성된 변수 및 함수를 저장합니다.

Environment Record는 다음과 같은 종류가 있습니다.

-   Declarative Environment Records : 함수 및 변수 선언과 같은 구문을 정의하는데 사용됩니다.
-   Object Environment Records : 전역 문맥과 with문에 나타나는 변수와 함수의 연관을 정의하는 데 사용됩니다.
-   Function Environment Records : 함수 내 최상위 선언에 사용됩니다.
-   Global Environment Records : ES에서 공유하는 가장 외부 범위이며 전역 식별자에 대한 ㅇclarative Environment Records와 빌트인 객체, 전역 객체에 대한 Object Environment Records가 있습니다.

> ### **outer**

LexicalEnvironment들의  논리적 중첩 구조(Logical nesting of Lexical Environment values)를 구현하는데 사용됩니다. 이 논리적 중첩 구조를 탐색해나가는 것을 스코프 체인이라고 합니다.

내부 LexicalEnvironment의 outer는 해당 Lexical Environment를 논리적으로 감싸고 있는 Lexical Environment를 참조합니다. 

그리고 outer LexicalEnvironment는 자체적으로 outer Lexical Environment를 가질 수 있습니다.

또한, LexcialEnvironment는 여러 개의 내부 LexicalEnvironment의 outer LexcialEnvironment로서 제공될 수도 있습니다.

---

지금까지의 내용들을 간단한 예제로 보겠습니다.

```
const name = 'e';

function exam () {
  const count = 0;

  function firstChild () {
  }
  
  function secondChild () {
  }
}
```

위 예제의 모든 함수가 차례대로 실행된다고 했을 때, 실행 문맥을 이해하기 쉽게 보자면 다음과 같습니다.

```
GlobalEnvironment {
  Environment Record: { name: 'e' },
  outer: null
}

examEnvironment {
  Environment Record: { count: 0 },
  outer: GlobalEnvironment
}

firstEnvironment {
  Environment Record: {},
  outer: examEnvironment
}

secondEnvironment {
  Environment Record: {},
  outer: examEnvironment
}


```

\`name\`은 전역에서 선언되었으므로 전역 환경에서 저장됩니다. 마찬가지로 count는 exam함수에서 선언되었으므로 exam환경에 저장됩니다. 또한, outer객체들을 잘 보면 각자 자신이 선언된 환경을 참조하고 있습니다.

만약 first함수에서 name값을 호출한다면 name 식별자를 찾을 때까지 outer객체의 참조를 따라 탐색합니다.

이것을 스코프 체인이라고 합니다.

<br/>

## **Execution Context Stack**

Execution Context Stack은 실행 문맥을 추적하는데 사용됩니다. LIFO 방식의 호출 스택이며 항상 Execution Context Stack의 가장 최상위 요소가 실행중인 실행 문맥입니다. 

  
현재 실행중인 실행 문맥과 관련된 실행 코드에서 해당 실행 문맥과 관련없는 실행 코드로 제어가 넘어갈 때마다 새로운 실행 문맥이 생성됩니다. 이렇게 새로 생성된 실행 문맥은 스택에 추가되고 현재 실행중인 실행 문맥이 됩니다.

예제를 통해 간단하게 알아보겠습니다.

```
function first () {
  second();
}

function second () {

}

first();
```

---

first함수가 호출되면서 호출 스택에 first함수의 실행 문맥이 쌓입니다.

```
호출 스택 => [first]
```

---

이후 first함수 내부의 코드를 실행하다가 second함수를 호출하게 되고 second함수의 실행 문맥을 호출 스택에 쌓습니다.

```
호출 스택 => [first, second]
```

---

second함수의 내부 코드를 전부 실행한 뒤 함수가 종료되면 second함수가 호출된 라인에서부터 first함수의 실행을 이어갑니다. 또한, second함수의 실행 문맥을 호출 스택에서 제거합니다.

```
호출 스택 => [first]
```

---

이후 first함수의 내부 코드가 모두 실행되면 함수를 종료하고 first함수를 호출했던 라인에서부터 실행을 이어갑니다. 물론 호출 스택에서 first함수의 실행 문맥은 제거됩니다.

```
호출 스택 => []
```

<br/>

## **this**

Global Environment Record는 전역 스코프를 가리키는 GlobalThisValue필드를 가지고 있습니다.

또한, 화살표 함수를 제외한 Function Environment Record는 this라는 필드가 제공되며 함수가 호출되는 부분이 어디냐에 따라 가리키는 값이 달라집니다. 만약 함수가 전역 실행 문맥에서 호출되었다면 this는 전역 객체를 가리킵니다. 하지만 Object나 생성자에서 호출되었다면 해당 인스턴스 자체를 가리킵니다.

```
function print () {
  console.log(this);
}
print(); // Window

function construct () {
  console.log(this);
}
new construct(); // construct {}

const object = {
  print: function () {
    console.log(this);
  }
};
object.print(); // {print: f}
```

---

엄격 모드인 'use strict'를 사용하면 전역 객체에 대한 this참조는 undefined가 됩니다.

```
'use strict'
function print () {
  console.log(this);
}
print(); // undefined
```

---

바인드 함수를 통해 this의 참조를 임의로 할당할 수도 있습니다.

```
const bindThis = function () {
  console.log(this);
}.bind('bindTest');

bindThis(); // String {"bindTest"}
```

---

화살표 함수는 실행될 때 this를 새로 정의하지 않습니다. 대신 코드에서 바로 바깥의 함수(혹은 class)의this값이 사용됩니다.

```
const bindThisArrowGlobalFn = () => console.log(this);

const bindThisTest = function () {
  console.log(this);
  
  const bindThisFn = function () {
    console.log(this);
  }
  
  const bindThisArrowFn = () => console.log(this);
  
  bindThisFn(); // Window
  bindThisArrowFn(); // String {"bindTest"}
  bindThisArrowGlobalFn(); //Window
  
}.bind('bindTest');

bindThisTest();
```

---

#### **\# 참고 문서**

[https://tc39.es/ecma262/](https://tc39.es/ecma262/)  
[ECMAScript® 2020 Language Specificationtc39.es](https://tc39.es/ecma262/)
