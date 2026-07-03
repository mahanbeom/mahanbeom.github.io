---
title: "Astro Islands 아키텍처 이해하기"
description: "정적 HTML 위에 필요한 곳만 JavaScript를 활성화하는 Islands 아키텍처를 알아봅니다."
pubDate: 2026-07-02
category: "Frontend"
tags: ["astro", "react", "architecture"]
---

## Islands 아키텍처란

Astro는 기본적으로 모든 페이지를 빌드 시점에 순수 HTML로 렌더링합니다. 인터랙션이 필요한 컴포넌트만 "섬(Island)"으로 지정해 클라이언트에서 JavaScript를 실행합니다.

## client 디렉티브

React 컴포넌트를 Astro 페이지에서 사용할 때 하이드레이션 시점을 지정할 수 있습니다.

```astro
---
import Counter from "@/components/Counter";
---

<!-- JS 없이 서버 렌더링만 -->
<Counter />

<!-- 페이지 로드 시 즉시 하이드레이션 -->
<Counter client:load />

<!-- 뷰포트에 보일 때 하이드레이션 -->
<Counter client:visible />
```

이 블로그의 다크모드 토글도 `client:load`로 동작하는 작은 섬입니다.

## 왜 좋은가

블로그처럼 읽기 중심인 사이트는 대부분의 페이지에 JavaScript가 필요 없습니다. Islands 방식은 필요한 곳에만 JS를 보내 로딩 속도와 Lighthouse 점수를 크게 개선합니다.
