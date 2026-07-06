---
title: "Next.js와 Vite + React SPA, 프론트엔드 도구 선택 전략"
date: 2026-07-06
tags: ["nextjs", "vite", "react", "rendering", "architecture"]
---

## 개요

어제 렌더링 전략(CSR/SSR/SSG/ISR)을 공부한 데 이어, 오늘은 Next.js라는 프레임워크를 깊게 파고들었다. "React만 쓰는 것과 뭐가 다른가?"에서 시작해 Vite의 정체, 별도 백엔드가 있을 때의 Next.js 역할, 그리고 최종적으로 "프론트엔드 도구를 고르는 기준"까지 도달했다.

## Next.js — 개념과 히스토리

Next.js는 Vercel이 만든 React 기반 풀스택 프레임워크다. React는 UI 라이브러리일 뿐이라 라우팅·서버 렌더링·번들링·데이터 페칭을 직접 구성해야 하는데, Next.js는 이를 통합 제공한다. 핵심 가치는 **렌더링 방식의 선택권** — 페이지별로 SSR/SSG/ISR/CSR을 섞어 쓸 수 있고, App Router에서는 RSC(React Server Components)가 기본이다.

히스토리 요약: 2016 v1(설정 없는 SSR) → 2019 v9(파일 기반 동적 라우팅, API Routes) → 2021 v12(SWC, Middleware) → 2022 v13(**App Router, RSC** — 최대 전환점) → 2023 v14(Server Actions) → 2024 v15(캐싱 기본값 완화, React 19) → 현재 v16.2 LTS.

장점: SEO·초기 로딩, 렌더링 전략 유연성, RSC로 번들 감소, 풀스택 통합, 자동 최적화. 단점: 잦은 패러다임 변화로 인한 러닝 커브, 다층 캐싱의 복잡성, 서버 인프라 필요, Vercel 종속 우려, hydration 등 SSR 특유의 디버깅 난이도.

## Vite의 정체 — 빌드 도구이지 프레임워크가 아니다

Vite는 Evan You가 만든 **빌드 도구**로, 개발 중엔 개발 서버, 배포 시엔 번들러 역할을 한다. 핵심 아이디어는 "개발 중에는 번들링하지 않는다":

- **의존성**은 esbuild로 한 번만 사전 번들링 후 캐시
- **소스 코드**는 네이티브 ESM으로 요청 시에만 즉석 변환 → 앱 크기와 무관하게 서버가 수백 ms에 뜨고 HMR이 항상 빠름
- **프로덕션 빌드**는 Rollup(→ Rolldown 전환 중)으로 전통적 번들링

Next.js는 프레임워크(자체 빌드 체인 사용), Vite는 빌드 도구 — 층위가 다르다. 오히려 SvelteKit, Nuxt, Astro, Remix가 모두 Vite 기반이다.

## React SPA vs Next.js — 사용 맥락이 가른다

Vite + React SPA의 강점은 단순한 멘탈 모델(모든 코드가 브라우저에서 실행), 정적 파일만으로 배포 가능한 가벼운 인프라, 백엔드 분리 아키텍처와의 궁합. 약점은 SEO 취약과 느린 초기 로딩.

그런데 이 약점은 **"불특정 다수가 한 번 방문하는 서비스"에서만 발동한다**. 사내 어드민·대시보드는 정반대 — 같은 사용자가 매일 접속(번들은 브라우저 캐시에), 한 세션을 오래 사용(전환 속도가 총 체감 성능을 지배), SEO 가치 0, 게다가 대시보드 로딩은 JS가 아니라 데이터 쿼리가 지배해서 SSR을 해도 별로 안 빨라진다. CSR이 어드민의 표준인 이유. 단, 번들 비대화 관리(라우트 단위 lazy loading)는 필수 과제다.

## 별도 백엔드가 있어도 Next.js가 유효한가

유효하다 — 단, 역할이 "풀스택 프레임워크"에서 **"렌더링 + BFF 레이어"**로 좁혀진다. 렌더링 문제(SEO, OG, 초기 로딩)는 백엔드가 해결해주지 않고, BFF로서 토큰 은닉(httpOnly 쿠키), API 조합(서버 간 저지연 병렬 호출), 캐싱(ISR로 백엔드 부하 감소)의 가치가 있다. 대신 운영 레이어 추가, 로직 이중화 위험, 프록시 홉 레이턴시라는 비용이 생긴다. SEO가 필요 없다면 이득이 급감해 SPA가 낫다.

## 렌더링 전략의 관할 범위

중요한 깨달음: SSR/SSG/CSR 구분은 **"최초 HTML을 누가 언제 만드느냐"**까지만 관할한다. hydration 이후의 폴링, 수동 refetch, 탭 포커스 갱신은 전부 클라이언트 사이드 데이터 페칭의 영역이고, 어떤 렌더링 전략을 쓰든 같은 도구(TanStack Query의 `refetchInterval`, `invalidateQueries`, WebSocket/SSE)로 해결한다. 오히려 "데이터가 금방 낡는다"는 대시보드 특성은 SSR로 채운 HTML의 유효 기간을 수십 초로 만들어 CSR 선택을 더 정당화한다.

## 도구 선택 전략 — 3단계 분기

1. **제품 특성** (렌더링/SEO — 1차 필터): 검색 유입 → Next.js 계열 / 로그인 뒤 도구 → Vite SPA / 정적 콘텐츠 → Astro
2. **팀·채용**: 팀이 아는 도구 > 기술적 최적. 기술 선택 = 채용 파이프라인 선택
3. **운영 제약**: SSR = Node 서버 운영. 인프라 역량·비용에 따라 최종 결정

추가 원칙: 되돌리기 어려운 결정(프레임워크)일수록 보수적으로, 쉬운 결정(상태 라이브러리)은 과감하게. 세부 스택은 커뮤니티 표준(TanStack Query, Zustand, Tailwind)을 따라 고민 비용을 0으로.

## Node 서버가 필요한 진짜 이유 — 시점 구분

"node_modules를 쓰려면 Node 서버가 필수 아닌가?"라는 의문이 있었는데, **Node가 필요한 시점**을 나누면 풀린다.

- **개발 시점**: Vite 개발 서버 = Node 프로그램. 개발자 PC에서만 실행
- **빌드 시점**: `npm run build`도 Node로 실행. node_modules의 코드는 이때 번들 JS 안으로 **복사되어 녹아든다** — 운영 환경에 node_modules는 존재할 필요 없음
- **운영 시점**: 여기서 갈린다. CSR은 번들 JS를 실행하는 주체가 **사용자 브라우저**이므로 서버는 파일 전송만 하면 됨(Node 불필요). SSR은 매 요청마다 서버에서 `renderToString()` 등 JS를 실행해야 하므로 **JS 런타임이 서버에 상주**해야 함(Node 필요)

즉 Node가 필요한 기준은 node_modules가 아니라 "운영 중 서버에서 JS를 실행할 일이 있느냐"다.

## 인프라 실체 — systemd vs Nginx 파일 서빙

**Next.js 셀프 호스팅**: `next build` → `next start`로 Node 프로세스가 3000 포트에 상주. 크래시 대비를 위해 systemd(`Restart=always`)나 PM2, Docker에 등록하고, 앞에 Nginx 리버스 프록시(TLS, gzip)를 둔다. 운영 부담 = 프로세스 감시·재시작, 메모리 누수, 무중단 배포, 스케일링. Vercel은 이걸 대신해주는 서비스다.

**Vite CSR**: `dist/`를 Nginx 문서 루트에 놓기만 하면 끝. Nginx도 프로세스지만 **내 애플리케이션 코드를 실행하는 프로세스가 아니라 범용 파일 서버** — 내 코드 버그로 죽을 일도, 메모리 누수도 없고, 배포는 파일 교체로 끝난다. S3/GitHub Pages면 VM 자체가 불필요. 유일한 설정은 SPA 라우팅용 `try_files $uri /index.html` fallback.

## 첫 렌더링 이후의 페칭 경로 — 렌더링 방식과 무관, 아키텍처가 결정

hydration이 끝나면 앱은 브라우저 안의 JS다. 이후 컴포넌트별 페칭은 `브라우저 → 백엔드 API` 직거래이고, 프론트를 서빙한 인프라(Node 프로세스든 Nginx든)는 이 경로에 등장하지 않는다. 그래서 SSR/CSR 무관하게 동일하게 동작한다.

**단 하나의 예외가 BFF 경유**: `브라우저 → Next.js 서버(Route Handler) → 백엔드`로 구성하면 Node 프로세스가 런타임 내내 모든 페칭 경로에 상주한다. 즉 "이후 페칭이 프론트 서버와 무관한가"는 렌더링 방식이 아니라 **직행 vs BFF 경유라는 아키텍처 선택**이 결정한다. CSR + Nginx는 경유할 프로세스가 없으니 항상 직행.

## BFF(Backend for Frontend) 패턴

BFF는 Next.js 전용이 아니라 **일반 아키텍처 패턴**(2015년경 SoundCloud/Sam Newman)이다. 범용 백엔드 API와 각 클라이언트(웹/iOS/Android)의 요구 사이 간극을 메우기 위해, 클라이언트마다 전용 중간 서버를 두는 것.

하는 일: API 조합(서버 간 저지연 병렬 호출로 여러 API를 한 응답으로), 응답 가공(도메인 모델 → 화면 모델), 토큰 은닉(httpOnly 쿠키로 XSS 노출 차단 — 최근 채택의 최대 동기), 캐싱, 프로토콜 변환.

구현 도구: Express/NestJS 독립 서버, Spring Cloud Gateway, GraphQL 서버, 그리고 Next.js. Next.js가 자주 언급되는 이유는 Route Handlers/Server Actions로 **별도 서버 없이 프론트 프로젝트에 BFF를 내장**할 수 있어서일 뿐, 패턴상 Express BFF와 동일하다.

원칙: **비즈니스 로직·권한 검증은 백엔드, BFF는 조합·가공·전달만**. 클라이언트가 웹 하나뿐이고 API가 화면에 잘 맞으면 BFF는 불필요한 홉이다.

## Q&A 하이라이트

- **Q. SSR 하려면 프레임워크가 필수인가?** A. 아니다. `renderToString()` + `hydrateRoot()`는 React API고 Vite도 SSR을 공식 지원한다. 하지만 라우팅 매칭, 데이터 직렬화/복원, hydration 정합성, 코드 스플리팅 연동을 전부 손으로 해야 해서 사실상 프레임워크를 만드는 일이 된다. 프레임워크의 상품은 SSR "가능"이 아니라 이 배관의 패키징이다.
- **Q. 실무 최다 사용 조합은?** A. ① Next.js + TS + Tailwind ② Vite + React + TS ③ Vue 3 + Nuxt ④ Angular(엔터프라이즈) ⑤ Svelte/Astro(콘텐츠·성능 특화). 1~2위가 시장의 60% 이상.
- **결론**: Next.js / Vite + React / Astro 셋은 제품 유형 3가지(검색 유입 서비스 / 로그인 도구 / 정적 콘텐츠)에 각각 대응하므로 이 셋을 다루면 웹 프론트 프로젝트의 ~90%를 커버한다. 진짜 자산은 그 아래 공통 기반(React, TS, 렌더링 전략 이해)이다.

## 더 알아볼 것

- Next.js 다층 캐싱 구조(fetch cache, Full Route Cache, Router Cache)의 정확한 동작
- RSC의 직렬화 방식과 `'use client'` 경계 규칙
- React Router v7 framework 모드 — Vite 생태계의 SSR 대안 직접 체험
- BFF에서 httpOnly 쿠키 기반 토큰 관리 실습 (세션 vs 토큰 릴레이)
- Next.js `output: 'standalone'`과 Docker 컨테이너 배포 구성
- 무중단 배포(blue-green, rolling) 실제 구성 방법
