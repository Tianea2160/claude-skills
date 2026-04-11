---
name: new-feature
description: "기능 개발 워크플로우 자동화. 리서치 → 작업 → 검토 → 보고 4단계로 체계적으로 기능을 구현한다. 사용법: /new-feature <기능 설명>"
argument-hint: "<기능 설명>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion, EnterPlanMode, ExitPlanMode, Skill, WebSearch, WebFetch, TaskCreate, TaskUpdate, TaskGet, TaskList
---

# New Feature Workflow

기능 개발을 4단계(리서치 → 작업 → 검토 → 보고)로 체계적으로 수행한다.

`$ARGUMENTS`가 기능 설명이다. 인자가 없으면 AskUserQuestion으로 목표를 먼저 수집한다.

> **필수 규칙:**
> - 사용자에게 질문, 선택을 요청할 때는 **반드시 AskUserQuestion 도구를 사용**한다. 일반 텍스트로 질문하지 않는다.
> - 구현 계획의 승인은 **EnterPlanMode → 계획 작성 → ExitPlanMode**로 받는다.

## 참조 문서

- [references/templates.md](references/templates.md) — 구현 계획 작성 가이드 및 작업 완료 보고 형식
- [references/examples.md](references/examples.md) — 작업 분해 및 병렬 실행 예시
- [references/review-checklist.md](references/review-checklist.md) — 코드 품질 검증 체크리스트
- [examples/good-plan.md](examples/good-plan.md) — 좋은 구현 계획 예시 (Plan 작성 시 참고)

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

### 1.4 구현 계획 수립 및 승인

EnterPlanMode로 진입하여, [references/templates.md](references/templates.md)의
"구현 계획 작성 가이드"에 따라 **구체적인 구현 계획**을 작성한다.
구체성의 기준은 [examples/good-plan.md](examples/good-plan.md)를 참고한다.

계획 작성 중 불명확한 부분이 있으면 AskUserQuestion으로 먼저 해소한 뒤 계획에 반영한다.

계획이 완성되면 ExitPlanMode로 사용자 승인을 요청한다.

**승인 시** Phase 2로 진행한다.
**수정 요청 시** 사용자의 피드백을 반영하여 계획을 수정하고 다시 승인을 요청한다.
필요하다면 추가 조사(1.2~1.3)를 수행한다.

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

### 2.4 코드 정리

모든 태스크와 테스트가 완료된 후, `/simplify`를 호출하여 변경된 코드를 자동으로 정리한다.
simplify가 코드 재사용, 품질, 효율성을 검토하고 문제가 있으면 수정한다.
수정이 발생하면 테스트를 다시 실행하여 회귀가 없는지 확인한다.

### 2.5 작업 원칙

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
미발견 문제가 없을 때 3.4로 진행한다.

### 3.4 환경 정리

이번 작업에서 새로 발견되었거나 확립된 규칙, 컨벤션, 패턴 중
코드만으로는 알 수 없는 것이 있다면 `**/CLAUDE.md` 또는 `.claude/rules/*.md`에 반영한다.

**대상:**
- 새로운 아키텍처 결정 사항 (예: "알림은 WebSocket으로 처리한다")
- 구현 중 확립된 컨벤션 (예: "이벤트 타입은 `src/types/` 하위에 정의한다")
- 다음 작업자가 알아야 할 주의사항 (예: "socket.io 초기화는 HTTP 서버 생성 후에 해야 한다")

**원칙:**
- 코드에서 이미 자명한 내용은 추가하지 않는다
- 기존 CLAUDE.md / rules 파일이 있으면 수정, 없으면 필요 시 생성한다
- 추가할 내용이 없으면 이 단계를 건너뛴다

Phase 3은 에이전트의 자체 검토 단계이며, 사용자에게 별도 승인을 받지 않는다.

---

## Phase 4: 보고

**목표:** 작업 결과를 사용자에게 명확하게 전달한다.

[references/templates.md](references/templates.md)의 "작업 완료 보고 형식"에 따라 보고한다.
