> ## 목차
>   
> 클로저란?  
> 이해하기  
> 클로저의 특징

<br/>

## **클로저(Closure)란?**

Javascript로 개발을 하다보면 알든 모르든 자주 쓰게 되는 기법입니다.

-   클로저(Closure)의 사전적 정의는 폐쇄입니다.
-   MDN 에서는 \`함수와 함수가 선언된 어휘적 환경(Lexical Environment)의 조합\`이라고 말합니다.
-   생활 코딩에서는 클로저(Closure)가 \`내부 함수가 외부 함수의 실행 문맥에 접근할 수 있는 것\`이라고 합니다.

**※실행 문맥(Execution Context)이란?**

JS가 실행되는 환경을 말하며 각 실행 문맥은 식별자(변수, 객체, ...) 및 this, outer 등JS 코드가 실행되는데 필요한 모든 상태들을 가지고 있습니다.함수가 실행되면 해당 함수의 실행 문맥이 생성되며 종료될 때 소멸합니다.

자세한 내용이 궁금하다면[여기](https://jee-goo.tistory.com/entry/JavaScript-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8Execution-Context%EB%9E%80)에서 확인하세요.

<br/>

## 이해하기

위 설명으로만 봐서는 이해가 잘 가지 않습니다. 바로 예제를 통해 알아봅니다.

```javascript
function makePrint () {
  let name = 'javascript';
  
  return function print() {
    console.log(name);
  }
}

// (1)
var fnPrint = makePrint();

```

makePrint함수는 print함수를 생성하여 반환하는 함수입니다. 또한 print함수는 makePrint함수의 내부 함수이죠.

print함수를 보면 name식별자에 접근하고 있습니다. 하지만 print함수에는 해당 식별자가 선언된 적이 없기 때문에 스코프 체인을 통해 외부 함수인 makePrint함수의 name식별자를 참조하게 됩니다.

(1)에서 makePrint함수가 호출될 때 makePrint함수의 실행 문맥이 생성됩니다. 그 후 print함수를 생성하여 반환하고 fnPrint변수에 할당한 뒤 함수가 종료됩니다.

일반적으로 함수가 종료 될 때, 실행되면서 할당되었던 지역 변수 등은 전부 GC에 의해서 메모리가 해제됩니다.

하지만 이 예제에서는 print함수가 makePrint의 실행 문맥에 존재하는 name식별자를 참조하기 때문에 makePrint함수는 종료가 되어도 name식별자는 메모리에서 해제되지 않고 계속 살아있게 됩니다. 여기서 print함수를 클로저 함수라고 합니다.

<br/>

## 클로저의 특징

첫 번째는 방금 위에서 언급한 것처럼, 참조하고 있는 외부 함수의 식별자가 메모리에서 해제되지 않는다는 점입니다. 이 점을 고려하지 않고 무분별하게 사용한다면 쓸데없이 메모리를 낭비할 수 있습니다.

두 번째는 캡슐화 및 은닉화입니다. 위 예제에서 name식별자에 접근하려면 오로지 print함수를 통해서만 가능합니다. 즉, 식별자에 대한 직접적인 접근을 제한할 수 있는 것이죠. 이 특성 때문에 Closure(폐쇄)라고 합니다.
