---
title: "Claude 스킬 생성 연습"
date: 2026-07-05
tags: ["claude", "skill", "cowork", "automation"]
---

## 개요

Cowork에서 학습 노트 발행을 자동화하는 커스텀 스킬(daily-study)을 직접 만들어 보며,
Claude 스킬의 구조와 제작·테스트·배포 과정을 공부했다.

## 스킬의 구조

스킬은 `SKILL.md` 하나가 핵심이다. 상단 YAML frontmatter에 `name`과 `description`을 쓰고,
본문에 Claude가 따라야 할 절차를 마크다운으로 작성한다. 반복 작업은 `scripts/` 폴더에
셸 스크립트로 번들해서 매번 명령을 새로 조립하지 않게 한다.

- `description`이 스킬 트리거의 핵심 메커니즘 — 무엇을 하는지뿐 아니라 **언제 써야 하는지**를
  구체적 표현("오늘 공부한 내용 정리해줘" 등)까지 포함해 적어야 잘 발동된다.
- 스킬명은 kebab-case(소문자·숫자·하이픈)만 허용된다. `daily_study`처럼 언더스코어는 등록 불가.

## Cowork 환경의 제약

- 로컬 파일 저장은 Cowork(데스크톱)에서 폴더를 연결해야 가능하다. 웹 Claude.ai에서는 불가.
- 셸 명령은 샌드박스 리눅스에서 실행되므로 Windows에 저장된 GitHub 자격증명을 쓸 수 없다.
  → push 자동화는 PAT(Personal Access Token)를 저장소의 `.env.local`에 두고 스크립트가 읽는 방식으로 해결.
  `.env.local`은 반드시 `.gitignore`에 등록해 토큰이 커밋되지 않게 한다.

## 스킬 테스트 방법

시뮬레이션 git 저장소(로컬 bare remote)를 만들어 스킬 사용/미사용을 서브에이전트로 병렬 실행하고
assertion 기반으로 채점했다. 결과: 스킬 사용 100% vs 미사용 80% — 미사용 쪽은 PAT 없는 환경에서
push를 그대로 시도해 실패했다. 스킬의 가치는 인증 처리 흐름과 규칙(파일명·커밋 메시지)의 일관성에 있었다.

## Q&A 하이라이트

- **Q. 스킬로 로컬 저장 + commit/push가 가능한가?** → md 생성·저장·commit은 가능. push는 샌드박스라
  자격증명 문제가 있어 PAT 방식이 필요하다는 것이 핵심 제약이었다.
- **Q. 스킬명을 daily_study로 할 수 있나?** → 불가. 명명 규칙상 `daily-study`로 등록하고,
  description에 "daily_study" 표현을 넣어 자연어 트리거는 유지했다.
- **Q. frontmatter는 어떻게 맞추나?** → 추측하지 말고 실제 `content.config.ts`의 zod 스키마를
  확인하는 것이 안전하다. 실제 스키마(title/date/tags)가 처음 알려준 것과 달랐다.

## 더 알아볼 것

- 스킬 description 최적화(트리거 정확도 개선) 자동화 루프
- 스킬을 플러그인으로 묶어 배포하는 방법
- Astro 콘텐츠 컬렉션 zod 스키마 확장(category, description 필드 추가)
