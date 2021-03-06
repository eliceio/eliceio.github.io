---
layout: post
title:  "Progressive Web App (1)"
author: "Suin Kim"
date:   2021-08-30 17:25:38 +0900
categories: JavaScript
---

PWA란 무엇인가?
==========

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/89ce248145c5404ab5959f16c377343c/image.png" description="PWA 기술을 이용해 Minimal-ui 상태로 실행된 엘리스 웹 앱" %}

PWA (Progressive Web App) 은 Google I/O 2016에서 발표한 기술로, 웹과 네이티브 앱의 기능 중 이점만을 가져올 수 있도록 수 많은 기술과 패턴을 사용해 개발된 웹앱을 총칭합니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/d30002b070c0409ba623e641628ffb22/image.png" description="https://web.dev/what-are-pwas/" width="385" height="365" %}

위 그림에서 볼 수 있는것과 같이 PWA는 웹 앱의 발견 용이성과 네이티브 앱의 강력함을 모두 제공하는 것을 목표로 합니다. 웹 앱의 경우 네이티브 앱에 비해 발견이 쉬운 점에는 다음과 같은 특성이 있습니다.

*   웹 사이트에 방문하는 것이 훨씬 쉽다.
*   설치해야하는 네이티브 앱과는 달리 웹 사이트에 방문하면 바로 사용할 수 있다.
*   URL 링크로 웹앱을 공유할 수 있다.

네이티브 앱의 경우 웹 앱보다는 발견의 용이성 (Reach of Platform) 이 떨어지지만 기능 면에서 훨씬 강력합니다.

*   부드러운 사용자 경험을 제공
*   내가 필요로 하는 정보에 대해 푸시 알림을 받을 수 있음
*   홈 화면 아이콘을 탭하여 웹보다 더 쉽게 접근할 수 있음

PWA의 특징
=======

웹 앱과 네이티브 앱이 가지는 장점만을 취하기 위해서 PWA는 새로운 기술 (Service Worker; 후술) 을 같이 사용합니다. 발견의 용이성을 위해서, PWA는 Android나 iOS 운영체제에 종속되지 않고 양쪽 플랫폼에서 동시에 실행될 수 있는 웹 기반 플랫폼을 사용합니다. 따라서 플랫폼에 관계 없이 어떤 운영체제에서도 실행될 수 있는 하나의 앱만을 제작해도 됩니다.

웹 기반 플랫폼의 한계를 벗어나, PWA는 네이티브 앱처럼 동작하여 브라우저 레벨에서 접근할 수 없었던 시스템 하드웨어나 소프트웨어에도 접근할 수 있습니다. PWA의 목표 중 하나는 성능이 뛰어날 경우 유저가 웹 기반 앱을 사용하고 있는지, 네이티브 앱을 사용하는 것인지 헷갈릴 정도의 사용자 경험을 제공하는 것입니다. 잘 설계된 PWA는 웹앱과 네이티브 앱의 장점을 결합한 UX를 제공할 수 있습니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/24ba0242b7964dc7a3a44d0e33e35860/image.png" description="https://medium.com/dev-channel/a-pinterest-progressive-web-app-performance-case-study-3bd6ed2e6154" %}

Pinterest 는 PWA 기반의 웹을 사용하여 앱은 아니지만 유사한 경험을 제공합니다. 알림, 앱 설치, 오프라인 모드 등을 제공합니다. PWA 기술을 사용하여, Pinterest 는 FMP (First Meaningful Paint) 시간을 4.2초에서 1.8초로, TTI (Time to Interactive) 시간을 23초에서 5.6초로 크게 줄일 수 있었습니다.

PWA의 장점
=======

PWA의 가장 큰 장점은 React Native, Flutter 와 같은 하이브리드 프레임워크의 장점과 일치하는데 바로 개발비용이 저렴하고 코드베이스가 적다는 것입니다. 또한 약간의 작업만으로 기존의 static 한 웹 사이트를 앱으로 만들 수도 있습니다.

PWA에서 쓰이는 기술을 사용하면, 일반적인 웹 사이트와 다르게 오프라인에서도 동작할 수 있습니다 (당연히 모든 기능이 동작하는 것은 아니고, asset 들과 일부 API call 들에 대한 캐싱이 필요합니다). 오프라인 동작은 모바일 디바이스에서 매우 중요한데, 불안정한 연결 속에서도 offline-first content 를 보여줄 수 있다면 사용자의 retention은 크게 증가할 것이기 때문입니다.

또한 앱만이 가지고 있던 특권인 푸시 알림은 브라우저가 닫혀 있더라도 가능하며, 설치가 가능하여 재방문율을 높일 수 있습니다. 마지막으로 검색 엔진에서 Application 으로 인식하여 검색이 가능합니다. 이러한 장점들은 유저의 전환율과 세션 수, 이탈률에 긍정적인 영향을 미칩니다. [pwastats.com](http://pwastats.com) 이 조사한 성공 사례 중 일부는 다음과 같습니다.

1.  Flipkart Lite: 2015년에 PWA로 재구축하여, 전환율을 70% 이상 상승
2.  AliExpress: 전환율 104%상승
3.  Twitter 세션당 페이지수가 65% 증가, 트윗이 75%증가, 이탈률 20% 감소, 앱 크기 97% 이상 감소
4.  Nikkei: 트래픽이 2.3배, 구독이 58%, 일일 활성 사용자 49% 증가
5.  Pinterest: 평균 접속 시간 40% 증가, 광고 수익 44% 증가, 핵심 유저 참여율 60% 증가

PWA의 단점
=======

아직까지는 모든 플랫폼에서 PWA가 완벽하게 지원되지는 않습니다. PWA는 Chrome 기반의 웹 플랫폼에서 가장 잘 동작하나 iOS 플랫폼에서는 일부 기능이 동작하지 않습니다. [CrustLab](https://crustlab.com/blog/progressive-web-apps-state-2020-2021/) 의 조사에 의하면, iOS (Safari) 플랫폼에서는 푸시 알림, 진동, 블루투스 기능이 Android (Chrome) 플랫폼과 비교해서 지원되지 않습니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/feaf151173164cd599f0c27e7b4d3bc2/image.png" description="https://caniuse.com/?search=PWA" %}


{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/451b9b56ffe64927a6a68b3c5b2227dd/image.png" description="https://crustlab.com/blog/progressive-web-apps-state-2020-2021/" width="385" height="686" %}

Service Worker
==============

[Service Worker](https://developers.google.com/web/fundamentals/primers/service-workers) (앞으로 SW로 통칭) 란 브라우저가 백그라운드에서 실행하는 스크립트입니다. 이 스크립트는 웹 페이지나 유저 인터랙션과 완전히 분리되어 실행됩니다. SW는 다음과 같은 특성을 가지고 있습니다.

*   SW는 자바스크립트 기반의 worker 로서, DOM에 직접 접근할 수 없습니다.
*   대신, [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) 인터페이스를 이용해 페이지에 있는 메인 스레드와 통신할 수 있습니다.
*   SW는 프로그래밍이 가능한 네트워크 프록시로, 페이지에서 보내는 네트워크 요청들을 어떻게 처리할지 커스텀하게 프로그래밍 할 수 있습니다.
*   SW는 쓰이지 않을때 종료되고, 필요할 때 재시작되기 때문에, SW의 `onfetch` 혹은 `onmessage` 핸들러에서 유추할 수 있는 global state 는 언제나 최신 상태로 유지된다고 확신할 수 없습니다. SW에서 계속해서 유지하며 사용해야 하는 정보를 담기 위해서는 [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) 를 사용할 수 있습니다.

Service Worker의 Life Cycle
==========================

SW는 웹 페이지와 완전히 분리된 life cycle을 가지며 여러 단계로 이루어져 있습니다.

SW를 웹 사이트에서 사용하기 위해서는 먼저 SW를 등록 (register) 해야 합니다. 이것은 페이지 내 자바스크립트 코드를 통해서 진행할 수 있습니다. SW의 등록을 시작하고 나면 브라우저가 백그라운드에서 SW의 설치를 시작합니다.

`설치` 단계에서는, SW에 등록된 static assets 들의 캐싱을 진행합니다. 등록된 모든 파일들에 대해 캐싱이 완료되면, SW는 `설치됨` 단계로 이동합니다. 만약 등록된 파일 중 하나라도 캐싱에 실패한다면, 설치는 실패하고 SW는 활성화되지 않습니다. 이 경우, SW는 추후 설치를 재시도합니다. 다만, 이 때 다른 SW를 사용하는 클라이언트가 이미 존재한다면 (예를 들어 다른 탭에 같은 SW를 사용하는 웹 사이트가 열려 있다거나) 활성화는 바로 이루어지지 않으며 모든 이전 버전의 SW가 닫힐 때까지 대기하게 됩니다. 이 대기는 `skipWaiting` 함수를 이용해 생략할수도 있습니다.

SW가 설치된 경우, `활성화` 단계가 바로 뒤따르게 되며 이 때 이전 캐시를 어떻게 처리할지 수동으로 처리할 수도 있습니다. 활성화된 SW는 이전 SW에 의해 로드된 페이지를 제외한 새로운 페이지에 대한 모든 제어를 진행하게 됩니다. 이 상태의 SW는 1) fetch 및 메시지 이벤트를 핸들링하거나 2) 멈춰있게 됩니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/2fcd29cfc79943738dfbf0f5ce2eeddf/image.png" description="https://developers.google.com/web/fundamentals/primers/service-workers" width="351" height="342" %}

Service Worker 의 실행 조건
======================

Service Worker 를 설치하고 실행하기 위해서는 첫 번째로 브라우저가 이를 지원해야 합니다. SW는 Firefox, Chrome, Opera 에서 지원되고 있으며 Chrome 기반의 웹 브라우저인 Microsoft Edge와 Naver Whale 에서도 가능합니다. Safari 에서는 일부 기능이 지원됩니다.

추가로 SW는 HTTPS 에서만 실행이 가능하나 개발을 위해 `localhost (http://localhost)` 에서는 이 조건이 생략됩니다.

마지막으로 `manifest.json` 의 유무가 있습니다. [Web app manifest](https://web.dev/add-manifest/) 는 JSON 파일로 PWA에 대한 정보와 현재 웹 사이트가 유저의 데스크탑이나 모바일 장치에 어떻게 설치되어야 하는지에 대한 정보를 저장한 파일입니다. 이 파일은 앱 이름, 아이콘, 그리고 앱의 URL 정보 등을 포함하고 있습니다. 아래는 엘리스가 PWA 앱을 위해 사용중인 파일의 예시입니다.

```js
{  
  "short_name": "Elice", // 유저의 홈 스크린에 나타날 이름  
  "name": "Elice", // 앱의 전체 이름  
  "start_url": ".", // 앱 실행시 이동할 route  
  "icons": [ // 아이콘 asset 들  
    {  
      "src": "logo512.png",  
      "sizes": "512x512",  
      "type": "image/png"  
    },  
    {  
      "src": "logo192.png",  
      "sizes": "192x192",  
      "type": "image/png"  
    },  
    {  
      "src": "favicon.ico",  
      "sizes": "64x64 32x32 16x16",  
      "type": "image/x-icon"  
    }  
  ],  
  "display": "minimal-ui", // fullscreen, standalone, minimal-ui 중 하나  
  "theme_color": "#524fa1", // 툴바에 표시될 색상  
  "background_color": "#f6f7f8" // Splash 스크린에 보일 배경 색상  
}
```

React 에서 PWA 지원하기
=================

HTML/CSS/JS 기반의 static 웹 사이트와는 달리 React 기반의 SPA 에서는 PWA 를 적용하기 상대적으로 조금 더 까다로운 편입니다 (하지만 설정하고 나면 그리 어렵지 않습니다!). [다음 포스트](/javascript/2021/10/24/progressive-web-app-2.html)에서는 실제로 기존에 PWA가 적용되어 있지 않은 프로젝트에서 PWA를 적용하는 방법에 대해 알아보겠습니다.