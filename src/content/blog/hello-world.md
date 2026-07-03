---
title: "블로그를 시작합니다"
description: "Astro + React + Tailwind + shadcn/ui 조합으로 만든 블로그의 첫 글입니다."
pubDate: 2026-07-03
category: "회고"
tags: ["블로그", "시작"]
---

## 블로그를 시작하며

개발하며 배운 것들을 기록하기 위해 블로그를 만들었습니다. 이 블로그는 다음 스택으로 구성되어 있습니다.

- **Astro** — 정적 사이트 생성, 콘텐츠 컬렉션
- **React** — 인터랙티브 컴포넌트 (Islands)
- **Tailwind CSS v4 + shadcn/ui** — 스타일링과 UI 컴포넌트
- **GitHub Pages** — 호스팅 및 자동 배포

## 글 쓰는 방법

`src/content/blog/` 폴더에 마크다운 파일을 추가하면 글이 발행됩니다.

```md
---
title: "글 제목"
description: "글 요약"
pubDate: 2026-07-03
category: "Frontend"
tags: ["astro", "react"]
---

본문 내용...
```

`draft: true`를 추가하면 빌드에서 제외됩니다.
