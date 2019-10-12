
> ### 목차
> 
> CRP  
> 문제를 어떻게 찾지?  
> 최적화

<br/>

## CRP(Critical Rendering Path)

---

CRP은 문서(HTML, CSS 및 JavaScript)를 화면에 표현하기 위해 거치는 일련의 과정을 말한다.

이 과정을 간단하게 설명하면 다음과 같다.  
HTML문서가 브라우저에 의해 구문 분석될 때 DOM트리와 CSSOM트리를 구성한다. 그 후 DOM, CSSOM 트리를 결합하여 Render 트리를 구성하고 배치(Layout)를 통해 페이지에 그려질 요소들의 크기와 위치를 결정한다. 마지막으로 그리기(Painting)과정을 통해 픽셀이 화면에 그려진다. 자세한 과정이 궁금하다면 [\[WEB\] 웹 브라우저 렌더링과 최적화](https://github.com/eonnine/Programming-Study/blob/master/Web/Browser%20Rendering%20and%20Optimization.md)를 참고.

더 나은 성능과 우수한 사용자 경험을 제공하기 위해서 CRP을 최적화하는 것은 중요하다.

<br/>

## 문제를 어떻게 찾지?

---

CRP 중 어느 부분을 최적화해야 하는지 어떻게 찾을까? 그 방법들을 살펴본다.

### \- Lighthouse

Lighthouse는 웹 앱의 품질을 개선하는 오픈 소스 자동화 도구이며 크롬의 확장 프로그램이나 Node의 패키지를 통해 사용할 수 있다.  
이 도구는 특정 페이지에 대해 테스트를 수행하고 그 결과를 보고서로 보여준다. [Lighthouse 설치 및 사용법](https://developers.google.com/web/tools/lighthouse/?hl=ko)  
아래는 이 도구를 이용하여 해당 포스팅을 테스팅한 결과의 일부다.
![lighthouse](https://user-images.githubusercontent.com/48500660/66695003-22b66700-ecf6-11e9-9f1a-08d8c5c25d5d.png)﻿

<br/>

### \- Performance API와 HTML Events

Performance API는 특정 페이지에 대한 타이밍 관련 성능 정보를 나타낸다.

![CPR](https://user-images.githubusercontent.com/48500660/66693660-3659d180-ece6-11e9-9a08-f402a9044c2d.png)

위 이미지는 페이지가 로드될 때 브라우저가 추적하는 타임스탬프 중 일부 순서를 표현한다.

1.  **domLoading**: 전체 프로세스의 시작. 브라우저가 처음 수신한 HTML 문서 바이트의 파싱을 시작하는 시점이다..
2.  **domInteractive**: 브라우저가 파싱을 완료. 모든 HTML 및 DOM 생성 작업이 완료되는 시점이다..
3.  **domContentLoaded**: DOM이 준비되고 자바스크립트 실행을 차단하는 스타일시트가 없는 시점이며 이제 렌더링 트리를 생성할 수 있다.
4.  **domComplete**: 이름이 의미하는 바와 같이, 모든 처리가 완료되고 페이지의 모든 리소스(이미지 등)의 다운로드가 완료된 시점이다.
5.  **loadEvent**: 각 페이지 로드의 최종 단계로, 브라우저가 onload 이벤트를 발생시킨다.

```javascript

function loggingCRP () {
    const t = window.performance.timing;
  const interactive = t.domInteractive - t.domLoading;
  const dcl = t.domContentLoadedEventStart - t.domLoading;
  const complete = t.domComplete - t.domLoading;
  const log = 
  'interactive: ' + interactive + 'ms, ' 
  + 'dcl: ' + dcl + 'ms, ' 
  + ' complete: ' + complete + 'ms';

  console.log(log);
}

loggingCRP();

```

각 HTML 이벤트의 타임스탬프를 통해서 체크할 수도 있다.

```javascript

document.addEventListener('DOMContentLoaded', function(event) {
    console.log('DOMContentLoaded', event);
});


document.addEventListener('readystatechange', function(e){
  if( document.readyState === 'interactive' ) console.log('interactive', event);
  if( document.readyState === 'complete' ) console.log('complete', event); 
});


window.addEventListener('load', function(e){
  console.log('load', event);
});


window.addEventListener('beforeunload', function(event) {
  console.log('beforeunload', event);
});


window.addEventListener('unload', function(event) {
  console.log('unload', event);
});

```

<br/>

### \- DevTools

크롬의 개발자 도구에서 Performance-EvnetLog를 이용하여 CRP 각각의 소요 시간을 체크할 수 있다.

![devtools](https://user-images.githubusercontent.com/48500660/66694342-4cb75b80-eced-11e9-86bf-3ab95fe9a932.png)

<br/>

## 최적화
---
위 방법들을 이용하여 성능을 개선해야 할 부분을 찾았다. 그럼 이제 최적화를 해야 한다.

### \- HTTP Request 줄이기

웹 사이트 최적화의 가장 기본적이면서 중요한 부분이다. 일반적으로 웹 문서를 로드할 때 js, css, 이미지, 마크업 등을 웹서버에서 가져오며 이 횟수가 많아질수록 소요되는 비용이 크게 증가한다. 이 문제는 아래 방법들을 통해 최적화할 수 있다.

1.  리소스 파일(js, css, ...)들을 bundling한다.  
    10개의 js파일을 로드해야 하는 페이지가 있다. 이 파일들을 번들링해서 1개의 파일만 로드하게 하는 것이다. 파일의 크기가 더 커지더라도 파일 각각을 요청한 뒤 응답받는 통신 비용이 감소하기 때문에 큰 성능 향상을 이룰 수 있다.
2.  CSS Sprites기법을 사용한다.  
    10개의 이미지를 사용하는 페이지가 있다. 이 때 10개의 이미지를 합쳐 한 이미지로 만든 뒤 페이지에서는 1개의 이미지만 로드한다.  
    그 후 background-position 스타일 속성을 이용해 사용할 이미지만 보이게 하는 기법이다.

<br/>

### \- 파일 크기 줄이기

이것도 웹 사이트에서 최적화하는 기본적이며 효과적인 방법이다. 웹 사이트에서 리소스를 로드할 때 파일의 크기가 크면 그만큼 시간이 더 소요된다. 그래서 로드될 파일들을 minify하면 성능을 향상 시킬 수 있다.

<br/>

### \- 초기 렌더링 최적화
처음 문서가 렌더링될 때 최적화하는 방법이다.

1.  처음 문서가 로드될 때 실행되는 Ajax의 요청을 최소화한다.
2.  css파일은 header에서 로드한다.  
    브라우저가 JS를 로드하고 분석할 때 스타일이 필요한 경우가 있다. 이 때 스타일 정보가 완전히 로드되지 않았다면 DOM 트리의 구성을 멈추게 된다. 때문에 css를 header에서 로드하여 전부 완료된 후 DOM을 구성하게 해야 한다.
3.  js파일은 body의 마지막 부분에서 로드하거나 지연 로드(Lazy Load)한다.  
    HTML파서가 HTML 구문 분석을 진행하다가 스크립트 태그를 만나면 JS 해석기가 제어권을 넘겨받고 스크립트를 실행한다. 이 때 HTML 파싱은 멈춰있게 되며 사용자 입장에서는 그만큼 체감 속도가 느려지게 된다.
