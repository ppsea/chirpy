---
title: 웹사이트 성능 최적화 (3) Resource load
date: 2024-01-28 20:16:00 +0900
categories: [Web, Optimization, Resource]
tags: [cs, web, optimization] # TAG names should always be lowercase
---

앞선 [포스팅](https://ppsea.github.io/posts/rendering/)에서 렌더링과정 최적화에 대해 알아보았고 렌더링을 차단하는 리소스의 일부(CSS - 렌더링 차단, Javasript - 파서 차단)를 알아보았습니다. 이번에는 Critical Rendering을 최적화하는 몇가지 기법을 더 알아보겠습니다.

## Preload Scanner 사용

Preload Scanner는 HTML 파서가 리소스를 발견하기 전에 원시 HTML 응답을 스캔하여 리소스를 찾아 추측하는 방식으로 가져오는 보조 HTML 파서 형태의 브라우저 최적화 기술입니다. 예를 들어 HTML 파싱이 차단된 상태여도 img 태그에 지정된 리소스 다운로드를 할 수 있습니다.

아래 Case들에는 preload scanner 사용이 되지 않으므로 특정한 이유가 없다면 최적화를 위해 피해야 할 패턴일 것입니다.

- `background-image` 속성을 사용하여 CSS에서 로드한 이미지 : 이러한 이미지 참조는 CSS에 있으며 Preload Scanner로 검색할 수 없습니다.
- JavaScript 또는 dynamic import를 사용하여 로드된 모듈을 사용하여 DOM에 삽입된 `<script>` 요소
- Javascript를 사용하여 클라이언트에서 렌더링된 HTML입니다. 이러한 마크업은 Javascript 리소스의 문자열 내에 포함되며 Preload Scanner에서 검색할 수 없습니다.

## CSS 로드 최적화 기법들

1. 축소하기
   - CSS를 축소하여 파일 크기를 줄여 더 빠르게 다운로드할 수 있습니다.
2. 사용하지 않는 CSS를 삭제하기
   - Chrome Dev Tools의 Souces Tab의 Coverage를 통해 사용하지 않는 CSS를 확인 가능
3. CSS `@import` 선언
   - @import선언을 찾기 위해 이 선언이 포함된 CSS파일을 먼저 다운로드해야 하기 때문에 Request chain이 발생하여 처음 렌더링 시간을 지연 시킵니다. `<link rel="stylesheet">` 방법으로 대체할 수 있습니다.
   - CSS 전처리기(SASS, LESS등)를 사용한다면 @import구문을 사용할 때 별개의 더 모듈화된 소스파일을 허용하므로 Request Chain 패널티를 방지합니다.
4. 인라인 CSS
   - 초기 표시 영역내에 표시되는 콘텐츠를 렌더링하는데 필요한 스타일을 `<head/>` 에 삽입하면 CSS리소스에 대한 네트워크 요청이 사라지며, 올바르게 처리된 경우 브라우저 캐시가 준비되지 않았을 때 초기 로드 시간이 개선될 수 있습니다. 나머지 CSS는 비동기 로드하거나 `<body>`의 끝에 추가할 수 있습니다.

## Javascript 로드 최적화 기법들

1. async와 defer 속성을 사용하기
   - `defer` 또는 `async` 속성 없이 `<script>` 요소를 로드하면 스크립트가 다운로드, 파싱, 실행될 때까지 브라우저에서 파싱과 렌더링이 차단됩니다.
     `async`로 로드된 스크립트는 다운로드 즉시 파싱 및 실행되는 반면 `defer`로 로드된 스크립트는 HTML 문서 파싱이 완료될 때 실행됩니다. 이 스크립트는 브라우저의 `DOMContentLoaded` 이벤트와 동시에 발생합니다. 또한 `async` 스크립트는 비순차적으로 실행되는 반면 `defer` 스크립트는 마크업에 표시된 순서대로 실행됩니다.
2. 클라이언트 측 렌더링은 LCP(Largest Conentful Paint)에 영향을 준다.
   - React와 같은 SPA에서 클라이언트 사이드 렌더링은 일반적으로 사용되는 기법입니다. 하지만 자바스크립트에서 렌더링된 마크업은 Preload Scanner를 회피하기에 이미지와 같은 중요한 리소스의 다운로드가 지연될 수 있습니다. 브라우저는 스크립트가 실행되고 요소를 DOM에 추가한 후에만 이미지 다운로드를 시작합니다. 결과적으로 스크립트는 검색, 다운로드, 파싱된 후에만 실행될 수 있습니다.
3. 축소하기
   - CSS의 경우와 마찬가지로 Javascript코드도 축소하여 다운로드 속도를 빠르게 할 수 있습니다.
