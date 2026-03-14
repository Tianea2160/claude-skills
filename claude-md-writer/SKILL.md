---
name: claude-md-writer
description: "CLAUDE.md 작성/리팩토링 및 .claude/rules/ 관리. 코드베이스를 분석하여 best practice 기반의 효율적인 CLAUDE.md와 path-scoped rules를 생성. 사용법: /claude-md-writer [create|refactor|rules]"
argument-hint: "[create|refactor|rules]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion
---

# CLAUDE.md Writer

코드베이스를 분석하여 best practice 기반의 효율적인 CLAUDE.md를 작성하거나 리팩토링하고, `.claude/rules/` path-scoped 규칙 파일을 관리한다.

## 모드 판별

`$ARGUMENTS`에 따라 모드를 결정한다:
- `create` 또는 인자 없음 + CLAUDE.md 없음 → **생성 모드**
- `refactor` 또는 인자 없음 + CLAUDE.md 있음 → **리팩토링 모드**
- `rules` → **Rules 관리 모드**

## 핵심 원칙

**이 원칙을 모든 작업에 반드시 적용한다:**

1. **200줄 이하** - 파일당 200줄을 넘기지 않는다. 길수록 Claude가 지시를 무시할 확률이 높아진다.
2. **코드에서 알 수 없는 것만** - 매 줄마다 "이걸 빼면 Claude가 실수하나?"를 자문한다.
3. **실행 가능한 명령어** - 모든 명령어는 copy-paste로 바로 실행 가능해야 한다.
4. **프로젝트 고유 정보만** - 일반적인 best practice나 언어 기본 규칙은 쓰지 않는다.
5. **계층적 분리** - root CLAUDE.md는 전체 맥락, 하위 CLAUDE.md는 도메인 맥락, rules는 조건부 규칙.

### 포함해야 하는 것

- 빌드/테스트/배포 명령어 (Claude가 추측할 수 없음)
- 기본값과 다른 코드 스타일 규칙
- 아키텍처 결정 사항과 그 이유 (코드는 WHAT, CLAUDE.md는 WHY)
- 환경 변수, 셋업 요구사항
- Gotchas (비직관적 동작, 흔한 실수)
- 워크플로우 (브랜치 네이밍, PR 컨벤션)

### 절대 포함하지 않는 것

- 파일별 설명 (코드를 읽으면 알 수 있음)
- 표준 언어 컨벤션 (Claude가 이미 앎)
- 장황한 설명이나 튜토리얼
- API 문서 (링크로 대체)
- 자주 변하는 정보
- "clean code를 작성하라" 같은 자명한 지시

## Phase 1: Discovery

### 1.1 기존 파일 탐색

```bash
find . -name "CLAUDE.md" -o -name ".claude.local.md" 2>/dev/null | head -50
```

```bash
ls -la .claude/rules/ 2>/dev/null
```

### 1.2 프로젝트 유형 감지

다음을 탐색하여 프로젝트 유형을 파악한다:

| 파일/디렉토리 | 프로젝트 유형 |
|-------------|-------------|
| `package.json` | Node.js/Frontend |
| `build.gradle.kts`, `pom.xml` | JVM (Kotlin/Java) |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `pyproject.toml`, `requirements.txt` | Python |
| `Dockerfile`, `docker-compose.yml` | Container |
| `kustomization.yaml`, `Chart.yaml` | Kubernetes/Helm |
| `terraform/`, `*.tf` | IaC |

### 1.3 언어 결정

- 기존 CLAUDE.md가 있으면 → 그 언어를 따른다
- 없으면 → AskUserQuestion으로 사용자에게 묻는다

## Phase 2: Analysis

Explore 에이전트를 사용하여 코드베이스를 병렬로 분석한다.

### 분석 대상

1. **빌드 시스템** - 빌드/테스트/린트 명령어, 스크립트
2. **아키텍처** - 디렉토리 구조, 핵심 패턴, 모듈 관계
3. **환경** - 필수 환경 변수, 설정 파일, 의존성
4. **워크플로우** - CI/CD, 브랜치 전략, 배포 방식
5. **기존 문서** - README, 기존 CLAUDE.md, 주석

## Phase 3: Interview

코드에서 알 수 없는 정보를 AskUserQuestion으로 수집한다.

**반드시 물어볼 것:**
- 팀/개인 워크플로우 규칙 (브랜치 네이밍, PR 규칙 등)
- 코드에서 드러나지 않는 gotchas
- 아키텍처 결정의 이유 (WHY)
- 특별한 코딩 컨벤션

**물어보지 않아도 되는 것:**
- 코드에서 읽을 수 있는 구조
- 표준적인 설정

## Phase 4: Generate / Refactor

### 생성 모드

[references/section-guide.md](references/section-guide.md)를 참고하여 CLAUDE.md를 작성한다.

**구조 템플릿:**

```markdown
# 프로젝트명

한 줄 설명.

## 명령어

| 명령어 | 설명 |
|-------|------|
| `command` | 설명 |

## 아키텍처

\```
dir/    # 용도
\```

## 코드 스타일

- 기본값과 다른 규칙만

## 주의사항

- gotcha 1
- gotcha 2
```

### 리팩토링 모드

1. 기존 CLAUDE.md를 읽는다
2. [references/best-practices.md](references/best-practices.md) 기준으로 평가한다
3. 문제점을 사용자에게 보여준다:
   - 불필요한 내용 (코드에서 알 수 있는 것)
   - 누락된 내용 (코드에서 알 수 없는 것)
   - 200줄 초과 여부
4. 사용자 승인 후 수정한다

### 하위 CLAUDE.md

모노레포나 대규모 프로젝트에서는 하위 디렉토리에 CLAUDE.md를 분리한다:
- root: 전체 프로젝트 맥락, 공통 명령어
- 하위: 도메인별 세부 규칙, 해당 모듈 고유 정보

## Phase 5: Rules 관리

`.claude/rules/*.md` 파일은 특정 경로의 파일을 작업할 때만 로드되는 조건부 규칙이다.

### Rules가 적합한 경우

- 특정 파일 유형에만 적용되는 규칙 (예: API 엔드포인트 작성 규칙)
- 특정 디렉토리에만 적용되는 패턴 (예: 테스트 작성 규칙)
- CLAUDE.md에 넣기엔 범위가 좁은 규칙

### Rules 파일 형식

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
---

# API 작성 규칙

- 모든 엔드포인트에 입력 검증 포함
- 표준 에러 응답 형식 사용
```

### Rules 생성 워크플로우

1. 프로젝트의 주요 도메인/디렉토리를 파악한다
2. 각 도메인에 반복적으로 적용되는 패턴을 식별한다
3. CLAUDE.md에서 path-scoped로 분리할 수 있는 규칙을 추출한다
4. 사용자에게 제안 후 승인받아 생성한다

## 최종 검증

작성/수정 후 반드시 확인:

- [ ] 각 CLAUDE.md가 200줄 이하인가?
- [ ] 모든 명령어가 실행 가능한가?
- [ ] 코드에서 알 수 있는 내용을 포함하지 않았는가?
- [ ] 프로젝트 고유 정보만 담고 있는가?
- [ ] rules 파일의 paths가 실제 존재하는 경로인가?
