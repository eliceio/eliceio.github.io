---
layout: post
title:  "Flutter 앱 성능 측정 및 개선 방법"
author: "Yechan Choi"
date:   2021-09-09 00:00:00 +0900
categories: Flutter
---

잘 만들어진 앱이란?
===========

잘 만들어진, 완성도가 높은 앱이란 어떤 앱일까요? 어떻게 하면 사용자에게 앱의 완성도가 높다는 평가를 들을 수 있을까요? 이 질문은 참 대답하기 어렵습니다. 사용자마다 저마다 다른 관점으로 바라볼 것이고, 모두를 만족시키기는 어려운 일이니까요. 하지만 ‘버벅이는 앱’이 완성도가 높다는 평가를 듣기란 쉬운 일이 아닙니다. 그럼 우리는 어떻게 우리 앱이 더 이상 버벅이지 않도록 할 수 있을까요?

Jank
====

Flutter는 Skia engine을 통해 Widget을 생성하고 제거합니다. 일반적으로 Skia engine은 60Hz로 동작하는 Ticker와 함께 화면을 업데이트 하므로, 우리는 16.7ms 안에 Rendering을 완료해야 합니다. 만약 우리가 Ticker의 주기에 맞추어 Rendering을 끝내지 못하게 된다면, 새로운 UI가 그려지지 않고, 화면은 업데이트 되지 않게 되며, 우리는 앱이 버벅인다고 느끼게 됩니다. 이렇게 앱이 주사율을 맞추지 못하고 버벅이는 것을 사용자가 보는 걸 Jank라고 합니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/68f533bff948406c9d1687fbe92c82ee/image.png" description="화면 업데이트 주기(점선)에 맞추어 Render가 완료되어야 하지만… 그렇지 못하면 Jank! ¹" %}

16.7ms 안에 Render가 완료되기만 하면 되는 건 아닙니다. Rendering에 소요되는 시간이 짧을 수록 배터리 소모와 발열이 줄어듭니다. 또한 Flutter는 60Hz 이상을 지원하는 앱에서는 가능한 한 주사율에 맞추어 더 빠르게 화면을 Render하기 때문에, Rendering에 걸리는 시간은 짧으면 짧을 수록 좋습니다.

Rendering Performance 측정
========================

Rendering Performance를 개선하기 위해서는 앱에서의 Jank의 발생을 감지하고, Rendering을 모니터링 해야 합니다. Flutter에서는 이를 위해 Performance Profiling 도구를 지원해줍니다. Flutter를 profile 모드로 실행하고, Dart DevTools를 켭니다. 화면 렌더링에 17ms를 초과하면 붉은 색 막대로 표현되며, 개선이 필요함을 알려줍니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/b1a4fb4a9e894158b5dfa1ab718534ec/image.png" description="Flutter DevTools — Performance" %}

Performance 탭에 표현되는 4개의 그래프에 대해 설명하면,

**UI**

Dart VM을 통해 Dart 코드를 실행하는 스레드입니다. Widget Tree를 Layer Tree로 변환하고, 이를 Raster 스레드로 보내는 역할을 합니다.

**Raster**

Layer Tree를 받아 GPU와 통신하여 UI를 업데이트 합니다. Skia engine이 이 스레드에서 실행됩니다. 개발자는 Raster 스레드나 스레드의 데이터에 접근할 수 없습니다.

**Platform**

각 플랫폼 별 스레드입니다. Performance Overlay에는 표시되지 않습니다.

**I/O**

입출력을 담당하고, Performance Overlay에는 표시되지 않습니다.

Jank가 나타나는 대부분의 경우는 UI와 Raster 스레드 모두에서 오랜 시간이 걸리는 경우와, UI 스레드는 아무런 문제가 없는데 Raster 스레드에서 문제가 나타나는 경우가 있습니다. 전자의 경우는 UI 스레드의 변경이 너무 잦거나, Widget Tree가 너무 자주 변경되거나, UI 스레드에서 무거운 작업이 수행되고 있는 경우입니다. 후자의 경우에는 saveLayer, Opacity, Shadow, Clip이 원인일 가능성이 높습니다. 정적이지 않은 이미지 캐싱도 많은 Cost를 소모합니다.² 자세한 내용은 [공식 문서](https://flutter.dev/docs/development/tools/devtools/performance)를 참고하세요.

Shader Compilation Jank
=======================

Shader compilation jank는 앱을 처음 실행할 때 animation이 버벅이는 현상을 의미합니다. Shader는 GPU에서 처리되는 코드 조각인데, 이 shader를 앱에서 그리기 위해서는 컴파일이 되어야 하고, 컴파일 되는 과정에서 jank가 유발됩니다. Flutter 1.20부터 SkSL을 사용해 shader compilation jank를 줄일 수 있습니다.

**SkSL warmup**

1) `--cache-sksl` 옵션으로 앱을 profile 모드로 실행합니다.

```
flutter run --profile --cache-sksl// --cache-sksl 옵션으로 처음 실행하는 경우   
flutter run --profile --cache-sksl --purge-persistent-cache
```

2) 가능한 모든 animation을 trigger합니다.

3) M을 눌러 캡쳐된 SkSL을 저장합니다.

4) 저장된 SkSL을 사용해 빌드합니다.

```
flutter build appbundle --bundle-sksl-path flutter\_01.sksl.json  
flutter build ios --bundle-sksl-path flutter\_01.sksl.json
```

이론적으로 저장된 SkSL이 다른 Device에서 도움이 된다는 보장은 없지만, 호환되지 않더라도 문제를 일으키지 않을 뿐더러 대부분의 경우 효과가 있다고 합니다.³ 자세한 내용은 [공식 문서](https://flutter.dev/docs/perf/rendering/shader)를 참고하세요.

성능 개선 방법
========

Rendering 성능을 개선할 수 있는 5가지 방법에 대해 소개해 드립니다.

build 메소드를 최대한 가볍게, 최대한 덜 호출되도록
-------------------------------

*   build 메소드는 UI 변경이 있을 때 언제든지 다시 호출될 수 있는 함수입니다. 그러므로 build에서 비용이 많이 드는 작업을 해서는 안 됩니다. FutureBuilder를 사용할 때, Future에 대한 caching을 하지 않았다면 매번 새롭게 future를 대기하게 됩니다.
*   하나의 큰 위젯보다는 작게 나누어진 여러 위젯이 낫습니다. 하나의 큰 위젯의 구현부를 메소드로 나누는 건 아무런 도움이 되지 않습니다. StatelessWidget과 StatefulWidget은 자체적인 caching 시스템을 이용하기 때문에 변경 없는 rebuild의 비용이 크지 않습니다.
*   build 메소드를 가능한 한 적게 호출되도록 하는 방법은 위젯을 const로 만드는 것입니다. const Widget은 상위 위젯이 rebuild 되어도, 변경이 없다면 다시 build되지 않습니다. 다음 DartPad에서 예제를 실행시켜서 Console 창을 확인해보세요.

<iframe src="https://dartpad.dev/58cce6e8028809b262c22c03ebe5f5ad" width="100%" height="400px" style="border: 0"></iframe>

가능한 Widget Tree는 변경되지 않도록
-------------------------

Flutter에서 실제로 위젯이 그려지는 과정은 다음과 같습니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/9ba04fc122184cb391f14df8e3042457/image.png" description="Widget Tree/Element Tree/Render Tree ⁴" %}

개발자가 구성한 Widget Tree는 Element Tree로 변환됩니다. Element Tree는 논리적 구조인 Widget Tree와 실제로 Rendering되는 구조인 Render Tree를 Mapping하는 Element의 Tree입니다. Widget은 `createElement()`를 통해 Element를 생성합니다. 이 때 생성되는 Element가 바로 BuildContext입니다. Element는 `createRenderObject()` 를 통해 RenderObject를 생성하고, Layer를 생성하게 됩니다. 이렇게 생성된 LayerTree가 Raster 스레드로 전달되어 위젯이 그려지게 됩니다.

따라서 rebuild 과정에서 Widget Tree에 변경이 없다면, Render Tree에서의 최소한의 변경 사항만 생기게 됩니다. 하지만 Widget Tree에 변경이 생긴다면 하위 Tree 전체를 다시 작성하게 되어 UI 스레드에 부하가 걸리게 됩니다.

가능하다면 lazy load
---------------

대부분의 경우에서 ListView보다는 ListView.builder가 낫습니다. ListView.builder는 화면에 표시되는 위젯만 동적으로 build하고, 화면에서 사라지면 (정확히는 cacheExtend 범위를 벗어나면) 메모리에 유지하지 않습니다. 하지만 ListView는 맨 처음 build될 때 모든 위젯을 빌드해 Jank를 유발합니다.

무거운 작업은 Isolate
---------------

먼저 Isolate와 Dart에서의 Future, Async, Await에 대해 알아보겠습니다. Dart는 기본적으로 단일 스레드 언어입니다. 다트는 오직 하나의 Isolate만을 가지고 시작합니다. Async와 Await은 병렬 작업이 아닙니다.

Isolate는 Memory와 하나의 스레드, EventLoop를 가진 독립적인 실행 공간입니다. eventLoop는 microTaskQueue와 eventQueue로 이루어져 있으며, 기본적으로 microTaskQueue가 우선권을 갖게 됩니다.

```
void eventLoop() {  
  while(microTaskQueue.isNotEmpty) {  
    fetchFirstMicroTaskFromQueue();  
    executeThisMicroTask();  
  }if (eventQueue.isNotEmpty) {  
    fetchFirstEventFromQueue();  
    executeThisEventRelatedCode();  
  }  
}//microTaskQueue에 있는 모든 task를 실행한 후에 eventQueue의 task를 실행합니다.\[5\]
```

모든 I/O, Gesture, Tap, Timer, Future, 다른 Isolates로부터의 message 등의 모든 Event는 eventQueue에 add된 후에, eventLoop에 의해 순차적으로 처리됩니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/ce568bddc2d04956a18024876862738a/image.png" description="EventQueue에 등록된 event가 eventLoop에 의해 순차적으로 실행되며, 각 event의 handler/task가 스레드에서 처리됩니다.⁶" %}


microtask와 eventloop에 대한 좀 더 자세한 정보는 [링크](https://developpaper.com/a-brief-introduction-to-the-concept-of-dart-asynchronous-programming/)를 참고하세요.

async, await은 다음과 같은 순서로 실행됩니다.

1) Future 객체가 내부적인 배열에 등록

2) Future와 관련해서 실행되어야 하는 코드들이 eventQueue에 등록

3) 불완전한 Future 객체 반환

4) 동기적으로 실행되어야 하는 코드들이 먼저 실행됨

5) eventLoop에 의해 먼저 처리된 후, data를 Future 객체에 담에 전달

즉, async와 await 역시 UI Thread에서 계산이 되기 때문에 무거운 작업을 UI Thread에서 처리하게 된다면 Jank가 발생할 수 있습니다.

그럼 어떻게 UI Thread에 부하를 주지 않고 무거운 작업을 처리할 수 있을까요? 병렬적으로 작업을 처리하기 위해서는 Isolate를 생성해야 합니다. 생성된 Isolate는 별개의 메모리와 EventLoop에 따라 동작하기 때문에 UI Processing에 영향을 주지 않으면서 동작할 수 있습니다. 한편 Isolate는 이름 그대로 다른 Isolate로부터 완전히 ‘격리’되어 있습니다. 따라서 새로 만든 Isolate는 메인 Isolate와 port를 통해 메시지를 주고 받는 방식으로 동작하게 됩니다.

{% include image.html url="https://cdn-api.elice.io/api-attachment/attachment/173866f256ef48f8964450820b757d9b/image.png" description="Main Isolate와 통신을 주고 받는 Timer Isolate ⁷" %}

Isolate는 별개의 메모리를 할당 받고, 메인 Isolate와 메시지를 주고 받는 오버헤드가 있기 때문에 무조건 Isolate를 생성해서 처리하는 건 좋은 방식이 아닙니다. 보통 UI가 업데이트 되는 주기인 16ms를 기준으로 잡고, 이보다 오래 걸리는 작업은 Isolate를 통해 처리하는 게 좋습니다. 16ms보다 길어질 수 있는 sync 작업의 대표적인 예시는 Json 직렬화입니다.

Flutter 2.5가 릴리즈되면서 이 부분에 꽤 중요한 변경사항이 생겼습니다. 변경사항은 아래의 Flutter 2.5 섹션에서 설명하도록 하겠습니다.

꼭 필요할 때만 effect를 사용
-------------------

effect들은 Raster 스레드와 GPU에 부하를 주게 됩니다.

*   `saveLayer()`는 구형 GPU를 가진 디바이스에서 속도 저하를 유발하는 원인이 됩니다. `SaveLayer()`를 명시적으로 호출하지 않더라도 `Clip.antiAliasWithSaveLayer`, `ShaderMask`, `ColorFilter`, `Chip`, `Text(overflowShader)`에서 `saveLayer()`가 트리거 될 수 있습니다.
*   Opacity위젯을 사용하는 것보다, 가능하다면 하위 위젯에서 옵션을 통해 투명도를 부여하는 편이 낫습니다.
*   Clip을 통해 borderRadius를 부여하는 것 보다는 모든 하위 위젯에 borderRadius 속성을 부여하는 게 더 낮은 cost를 소모합니다.

Flutter 2.5
===========

이 글을 쓰고 있는 오늘 아침에 Flutter 2.5가 릴리즈되었습니다. 성능 관련된 개선 사항이 굉장히 많고, Flutter DevTools도 업데이트 되었습니다.

먼저, iOS의 Metal Shader가 개선되어 Raster화 시간을 2/3으로 줄였다고 합니다. 또한 스케쥴링 정책이 변경되었습니다. 이전에는 위에서 설명드린대로 async 작업에 의해 UI 업데이트가 중단되는 경우가 있었는데, 이제는 frame processing을 microtask보다 우선권을 높여서, UI 스레드가 frame을 처리하고 있는 동안에는 microtask를 잠시 멈추도록 변경되었습니다. Jank 발생이 매우 줄어들 것으로 기대됩니다.

Garbage Collector가 메모리를 회수할 때 UI 스레드를 멈추는데, 이 역시 Jank의 원인이 됩니다. 이전에는 이미지에 대한 메모리가 느리게 회수되어 메모리가 부족한 기기에서 Jank가 매우 빈번하게 나타났는데, 이제는 Garbage Collector가 사용하지 않는 이미지에 대한 메모리를 매우 적극적으로 회수해 GC 실행 횟수를 상당히 줄이도록 변경되었습니다. 약 20초 짜리 GIF를 재생할 때 400번 이상의 GC 실행이 4번의 실행으로 줄었다고 합니다.

Performance Overlay에는 표시되지 않았지만 Platfrom Thread와의 통신도 대기 시간이 존재해서 Jank를 유발하는 원인이 될 수 있었습니다. 메시지 코덱의 불필요한 복사본을 제거하면서 디바이스에 따라 대기시간이 최대 50% 감소했다고 합니다.

개발자가 최적화 할 수 없었던 Jank들이 이번 Flutter 2.5 릴리즈로 상당수 해결되었습니다. 성능 관련 업데이트 말고도 Apple Silicon M1 관련 소소한 업데이트, Dart 2.14, Android 전체 화면, Material You, MateiralState.scrolledUnder, Material Banner, TextEditingShortcuts 등 굵직굵직한 변화가 많습니다.

Flutter 2.2.3에서 Flutter 2.5.0으로 건너 뛸 만큼 중요한 업데이트이니, 변경사항을 [What’s new in flutter 2.5](https://medium.com/flutter/whats-new-in-flutter-2-5-6f080c3f3dc)에서 확인해 보시고 앱에 적용해 보시기 바랍니다. 오늘도 즐거운 Flutter 하세요!

참고 문헌 및 출처
==========

\[1\] [https://www.youtube.com/watch?v=PKGguGUwSYE](https://www.youtube.com/watch?v=PKGguGUwSYE)

\[2\] [https://flutter.dev/docs/development/tools/devtools/performance](https://flutter.dev/docs/development/tools/devtools/performance)

\[3\] [https://flutter.dev/docs/perf/rendering/shader](https://flutter.dev/docs/perf/rendering/shader)

\[4\] [https://flutter.dev/docs/resources/architectural-overview](https://flutter.dev/docs/resources/architectural-overview)

\[5\] [https://www.didierboelens.com/2019/01/futures-isolates-event-loop/](https://www.didierboelens.com/2019/01/futures-isolates-event-loop/)

\[6\] [https://developpaper.com/a-brief-introduction-to-the-concept-of-dart-asynchronous-programming/](https://developpaper.com/a-brief-introduction-to-the-concept-of-dart-asynchronous-programming/)

\[7\] [https://medium.com/@valiodas/dart-isolates-and-computation-e6bbbb076d74](https://medium.com/@valiodas/dart-isolates-and-computation-e6bbbb076d74)

\[8\] [https://flutter.dev/docs/perf](https://flutter.dev/docs/perf)

\[9\] [https://github.com/flutter/flutter/issues/35162](https://github.com/flutter/flutter/issues/35162)

\[10\] [https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html#performance-considerations](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html#performance-considerations)

\[11\] [https://blog.codemagic.io/how-to-improve-the-performance-of-your-flutter-app./](https://blog.codemagic.io/how-to-improve-the-performance-of-your-flutter-app./)

