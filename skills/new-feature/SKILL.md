---
name: new-feature
description: "기능 개발 워크플로우 자동화. 리서치 → 작업 → 검토 → 보고 4단계로 체계적으로 기능을 구현한다. 사용법: /new-feature <기능 설명>"
argument-hint: "<기능 설명>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion, WebSearch, WebFetch, TaskCreate, TaskUpdate, TaskGet, TaskList
---

# New Feature Workflow

기능 개발을 4단계(리서치 → 작업 → 검토 → 보고)로 체계적으로 수행한다.

`$ARGUMENTS`가 기능 설명이다. 인자가 없으면 AskUserQuestion으로 목표를 먼저 수집한다.

> **필수 규칙:** 사용자에게 질문, 확인, 승인, 선택을 요청할 때는 **반드시 AskUserQuestion 도구를 사용**한다. 일반 텍스트로 질문하지 않는다.

## 참조 문서

- [references/templates.md](references/templates.md) — 리서치 결과 제안 및 작업 완료 보고 형식
- [references/examples.md](references/examples.md) — 작업 분해 및 병렬 실행 예시
- [references/review-checklist.md](references/review-checklist.md) — 코드 품질 검증 체크리스트

---

## Phase 1: 리서치

**목표:** 무엇을 어떻게 만들지 명확히 정의한다.

### 1.1 요구사항 수집

`$ARGUMENTS`에 사용자의 목표가 포함되어 있으면 요구사항 수집은 완료된 것으로 본다.

이후 코드베이스 분석(1.2) 및 외부 조사(1.3) 과정에서 부족한 정보가 발견되면
AskUserQuestion으로 단계적으로 질문한다:
- **제약조건**: 기술적 제약, 호환성 요구사항
- **우선순위**: 핵심 기능 vs nice-to-have 구분
- **기대 결과**: 완성 시 어떤 상태여야 하는가

불명확한 부분이 있으면 가정하지 말고 반드시 AskUserQuestion으로 질문한다.

### 1.2 코드베이스 분석

다음 3가지를 분석한다:

1. **재사용 가능한 코드** — 기존 유틸리티, 패턴, 컴포넌트 중 활용할 수 있는 것
2. **영향 범위** — 변경이 미치는 파일/모듈 범위
3. **기존 패턴** — 프로젝트에서 유사 기능이 어떻게 구현되어 있는지

새 코드를 작성하기 전에 기존 구현을 반드시 먼저 찾는다.

### 1.3 외부 조사

WebSearch/WebFetch로 best practice를 조사한다:
- 동일/유사 기능의 업계 구현 사례
- 공식 문서의 권장 패턴
- 알려진 함정이나 주의사항

### 1.4 제안 및 승인

[references/templates.md](references/templates.md)의 "리서치 결과 제안 형식"에 따라 조사 결과를 제안한다.

반드시 AskUserQuestion으로 승인/수정/거부를 받는다.

**승인 시** Phase 2로 진행한다.
**그 외** 사용자의 피드백을 분석하여 진정으로 원하는 것이 무엇인지 파악하고,
필요하다면 추가 조사(1.2~1.3)를 수행한 뒤 제안을 수정하여 다시 승인을 요청한다.
이 피드백 루프는 사용자가 승인할 때까지 반복한다.

승인된 내용을 기반으로 Plan을 생성하여 요구사항과 접근 방식을 기록한다.

---

## Phase 2: 작업

**목표:** 승인된 접근 방식으로 체계적으로 구현한다.

### 2.1 작업 분해

TaskCreate로 작업을 세분화한다. 각 태스크는:
- **독립적으로 완료 가능한 단위**여야 한다
- **의존성이 명확**해야 한다 (선행 작업 명시)
- **검증 가능**해야 한다 (완료 조건 명시)

구체적인 분해 예시는 [references/examples.md](references/examples.md)를 참고한다.

### 2.2 실행

1. 의존성이 없는 태스크부터 시작한다
2. 각 태스크 시작 시 TaskUpdate로 `in_progress`로 변경한다
3. 완료 시 TaskUpdate로 `completed`로 변경한다
4. 다음 태스크의 선행 조건이 충족되었는지 확인 후 진행한다

**병렬 실행 원칙:**
- 수정 대상 파일이 겹치지 않는 태스크들은 Agent를 활용하여 **병렬로 실행**한다
- 병렬 실행 시 각 Agent에게 태스크의 목적, 수정할 파일, 준수할 패턴을 명확히 전달한다
- 병렬 Agent가 모두 완료된 후, 의존성이 있는 다음 단계로 진행한다
- 상세 예시는 [references/examples.md](references/examples.md)의 "병렬 실행 예시" 참고

### 2.3 테스트

- 각 태스크 완료 시, 해당 태스크에서 수정한 범위에 대한 **단위/관련 테스트를 실행**한다
- 모든 태스크가 완료된 후, **프로젝트 전체 테스트 명령어를 실행**하여 회귀를 확인한다
- 테스트 실패 시 즉시 수정하고, 수정 후 다시 테스트를 실행한다

테스트 명령어는 CLAUDE.md, package.json, Makefile, build.gradle 등
프로젝트 설정 파일에서 확인한다.

### 2.4 작업 원칙

- 기존 코드/패턴을 최대한 재사용한다
- 과도한 추상화를 피한다 — 현재 필요한 만큼만
- 보안 취약점을 만들지 않는다 (OWASP Top 10)
- 변경 범위를 최소화한다 — 요청된 것만

---

## Phase 3: 검토

**목표:** 결과물이 요구사항에 부합하고 결함이 없는지 확인한다.

### 3.1 목표 정합성 검증

Plan에 기록된 요구사항을 기준으로 하나씩 대조한다:
- [ ] 핵심 기능이 모두 구현되었는가?
- [ ] 제약조건을 준수했는가?
- [ ] 기대 결과와 일치하는가?
- [ ] 원래 방향에서 벗어나지 않았는가?

### 3.2 코드 품질 검증

[references/review-checklist.md](references/review-checklist.md)에 따라 변경된 코드를 전수 검토한다.

### 3.3 문제 수정

발견된 문제는 즉시 수정한다. 수정 후 다시 3.1-3.2를 반복한다.
미발견 문제가 없을 때 검토를 종료하고 Phase 4로 진행한다.

Phase 3은 에이전트의 자체 검토 단계이며, 사용자에게 별도 승인을 받지 않는다.

---

## Phase 4: 보고

**목표:** 작업 결과를 사용자에게 명확하게 전달한다.

[references/templates.md](references/templates.md)의 "작업 완료 보고 형식"에 따라 보고한다.
