---
title: "GitHub Actions로 GitHub Pages 자동 배포하기"
description: "main 브랜치에 푸시하면 자동으로 빌드·배포되는 워크플로를 구성합니다."
pubDate: 2026-07-01
category: "DevOps"
tags: ["github-actions", "deploy"]
---

## 배포 흐름

이 블로그는 `main` 브랜치에 푸시하면 GitHub Actions가 자동으로 Astro 빌드를 실행하고, 결과물을 GitHub Pages에 배포합니다.

```yaml
on:
  push:
    branches: [main]
```

## 저장소 설정

배포가 동작하려면 저장소 설정이 한 가지 필요합니다.

1. GitHub 저장소 → **Settings** → **Pages**
2. **Source**를 **GitHub Actions**로 변경

이후에는 글을 쓰고 커밋·푸시만 하면 1~2분 안에 사이트가 갱신됩니다.

## 로컬 미리보기

배포 전 로컬에서 확인하려면:

```bash
npm run dev      # 개발 서버 (localhost:4321)
npm run build    # 프로덕션 빌드
npm run preview  # 빌드 결과 미리보기
```
