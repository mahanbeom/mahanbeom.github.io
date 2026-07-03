# mahanbeom.github.io

Astro + React + Tailwind CSS v4 + shadcn/ui로 만든 개인 블로그.

## 기술 스택

| 영역 | 도구 |
| --- | --- |
| 프레임워크 | [Astro 5](https://astro.build) (정적 사이트 생성) |
| UI | React 19 (Islands), shadcn/ui 스타일 컴포넌트 |
| 스타일링 | Tailwind CSS v4 |
| 콘텐츠 | 마크다운/MDX (콘텐츠 컬렉션) |
| 배포 | GitHub Actions → GitHub Pages |

## 시작하기

```bash
pnpm install
pnpm dev       # 개발 서버 (http://localhost:4321)
pnpm build     # 프로덕션 빌드 (dist/)
pnpm preview   # 빌드 결과 미리보기
```

pnpm이 없다면: `npm i -g pnpm` 또는 `corepack enable`

## 글 쓰기

`src/content/blog/`에 마크다운 파일을 추가합니다.

```md
---
title: "글 제목"
description: "글 요약 (목록 카드와 SEO에 사용)"
pubDate: 2026-07-03
category: "Frontend"
tags: ["astro", "react"]
draft: false
---

본문 내용...
```

- `category`: 사이드바 카테고리에 자동 집계됩니다.
- `tags`: 태그 페이지(`/tags/이름/`)가 자동 생성됩니다.
- `draft: true`: 빌드에서 제외됩니다.

## 배포

`main` 브랜치에 푸시하면 GitHub Actions가 자동으로 빌드·배포합니다.

**최초 1회 설정**: 저장소 → Settings → Pages → Source를 **GitHub Actions**로 변경.

## 커스터마이징

- 사이트 제목·프로필·연락처: `src/consts.ts`
- 소개 페이지: `src/pages/about.astro`
- 색상 테마: `src/styles/global.css` (shadcn CSS 변수)
- shadcn 컴포넌트 추가: `pnpm dlx shadcn@latest add <component>`

## 구조

```
src/
├── components/
│   ├── ui/              # shadcn/ui 컴포넌트
│   ├── Sidebar.astro    # 프로필·카테고리·태그 사이드바
│   ├── PostCard.astro   # 글 목록 카드
│   └── ThemeToggle.tsx  # 다크모드 토글 (React Island)
├── content/blog/        # 블로그 글 (마크다운)
├── layouts/BaseLayout.astro
├── pages/               # 라우트 (홈, 글, 카테고리, 태그, 소개, RSS)
└── styles/global.css    # Tailwind + shadcn 테마
```
