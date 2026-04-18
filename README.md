<h1 align="center">Claude Skills</h1>
<p align="center">
  Claude Code를 위한 커스텀 스킬 & 에이전트 컬렉션
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href="#스킬-목록"><img src="https://img.shields.io/badge/skills-6-brightgreen.svg" alt="Skills"></a>
  <a href="#에이전트-목록"><img src="https://img.shields.io/badge/agents-2-brightgreen.svg" alt="Agents"></a>
  <a href="https://github.com/Tianea2160/claude-skills/pulls"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome"></a>
</p>

---

## 소개

**Claude Skills**는 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)의 반복 작업을 자동화하는 스킬과 에이전트 모음이다. 각 스킬은 검증된 워크플로우를 캡슐화하여, 슬래시 명령어 하나로 복잡한 작업을 체계적으로 수행한다. 에이전트는 특정 도메인에 특화된 서브에이전트를 정의하여, Claude가 적절한 시점에 자동으로 위임한다.

> 전문가의 워크플로우를 스킬로 패키징하면, 누구나 전문가 수준의 결과를 얻을 수 있다.

**왜 이렇게 설계했는지** 궁금하다면 [설계 철학](docs/design-philosophy.md) 문서를 참고한다.

## 스킬 목록

| 스킬 | 설명 | 사용법 |
|------|------|--------|
| [claude-md-writer](skills/claude-md-writer/SKILL.md) | CLAUDE.md 작성/리팩토링 및 `.claude/rules/` 관리 | `/claude-md-writer [create\|refactor\|rules]` |
| [new-feature](skills/new-feature/SKILL.md) | 4개 feature-* 스킬을 순차 호출하는 **래퍼**. 전체 워크플로우 + 최종 개조식 보고 | `/new-feature <기능 설명>` |
| [feature-research](skills/feature-research/SKILL.md) | 리서치 단계. 코드베이스 병렬 탐색 + 외부 조사 → 승인된 Plan 파일 생성 | `/feature-research <기능 설명> [--plan <경로>]` |
| [feature-work](skills/feature-work/SKILL.md) | 구현 단계. Plan을 TaskCreate로 분해 → 병렬 에이전트 실행 → 테스트/simplify | `/feature-work <plan-path>` |
| [feature-test](skills/feature-test/SKILL.md) | 테스트 작성 단계. 시나리오 도출 → 파일별 에이전트로 테스트 작성/실행 + 자가 치유 | `/feature-test <plan-path>` |
| [feature-review](skills/feature-review/SKILL.md) | 검증 단계. 목표 정합성 + 코드 품질 병렬 검증 → 루프 → CLAUDE.md 반영 | `/feature-review <plan-path>` |

## 에이전트 목록

| 에이전트 | 모델 | 역할 |
|----------|------|------|
| [external-research](agents/external-research.md) | sonnet | 외부(웹) 조사 전용. WebSearch/WebFetch로 업계 사례·공식 문서·알려진 함정을 **출처 포함** 수집 |
| [code-implementer](agents/code-implementer.md) | sonnet | 코드 작업 전담 실행자. Plan 기반 task 단위 편집 + 스코프 테스트 + 2회 자가 치유, 경로·심볼 수준 요약만 반환 |

두 에이전트 모두 `feature-*` 스킬이 위임 대상으로 호출한다. `code-implementer`는 `feature-work`의 구현, `feature-test`의 테스트 작성, `feature-review`의 자가 치유 단계에서 사용된다.

## 빠른 시작

### 1. 저장소 클론

```bash
git clone https://github.com/Tianea2160/claude-skills.git
```

### 2. 설치

이 저장소는 `.claude/` 디렉토리 구조를 그대로 따르므로, 원하는 항목을 프로젝트의 `.claude/` 경로에 복사하면 된다.

```bash
# 스킬 전체 설치
cp -r claude-skills/skills/* /your-project/.claude/skills/

# 에이전트 설치
cp -r claude-skills/agents/*.md /your-project/.claude/agents/
```

개인 전역 설치 (모든 프로젝트에서 사용):

```bash
# 스킬 전역 설치
cp -r claude-skills/skills/* ~/.claude/skills/

# 에이전트 전역 설치
cp -r claude-skills/agents/*.md ~/.claude/agents/
```

### 3. 사용

Claude Code에서 슬래시 명령어로 실행한다.

```bash
# CLAUDE.md 새로 생성
/claude-md-writer create

# 기존 CLAUDE.md 리팩토링
/claude-md-writer refactor

# 새 기능 개발
/new-feature 로그인 API에 OAuth2 소셜 로그인 추가
```

## 스킬 상세

<details>
<summary><b>claude-md-writer</b> — CLAUDE.md 작성 및 관리</summary>

### 핵심 기능

- **생성 모드**: 코드베이스를 분석하여 best practice 기반의 CLAUDE.md를 자동 생성
- **리팩토링 모드**: 기존 CLAUDE.md를 평가하고 개선점을 제안
- **Rules 관리**: `.claude/rules/` path-scoped 규칙 파일 생성 및 관리

### 워크플로우

```
Discovery → Analysis → Interview → Generate/Refactor → Validate
```

1. 프로젝트 유형과 기존 파일을 탐색한다
2. Explore 에이전트로 빌드 시스템, 아키텍처, 환경을 분석한다
3. 코드에서 알 수 없는 정보를 사용자에게 질문한다
4. 200줄 이하의 효율적인 CLAUDE.md를 생성/리팩토링한다
5. 최종 검증 체크리스트로 품질을 확인한다

### 핵심 원칙

- 파일당 **200줄 이하** 유지
- **코드에서 알 수 없는 것만** 포함
- 모든 명령어는 **copy-paste 실행 가능**
- **프로젝트 고유 정보만** 작성

자세한 내용은 [SKILL.md](skills/claude-md-writer/SKILL.md)를 참고한다.

</details>

<details>
<summary><b>new-feature</b> + <b>feature-*</b> — 4단계 분리형 기능 개발 워크플로우</summary>

### 구성

- **`/new-feature` (래퍼)** — 아래 4개 스킬을 순차 호출하고 최종 **개조식 종합 보고**를 출력
- **`/feature-research`** — 코드베이스 병렬 탐색 + external-research → 승인된 Plan 파일 작성
- **`/feature-work`** — Plan의 `## Task Order`를 TaskCreate로 분해 → 파일 겹침 없는 태스크를 **병렬 에이전트**로 실행 → 테스트 + `/simplify`
- **`/feature-test`** — Plan 기반 테스트 시나리오 도출 → 파일별 에이전트 위임으로 테스트 작성/실행 → 2회 한도 자가 치유
- **`/feature-review`** — 목표 정합성 + 코드 품질 **2개 에이전트 병렬** 검증 → delta-only 재검증 → CLAUDE.md 반영

### 워크플로우

```
feature-research → [승인 게이트] → feature-work → feature-test → feature-review
                                                                     ↓
                                                 new-feature 래퍼가 최종 보고
```

### 설계 원칙

- **Plan 파일 계약**: `.claude/plans/<slug>.md`를 4개 스킬이 공유. 본문은 경로로만 전달
- **Agent 격리**: 모든 위임 프롬프트에 고정 반환 스키마를 강제해 main context에 요약만 수렴
- **개조식 보고**: 모든 mini-보고와 래퍼 종합 보고가 outline 형식(명사구/동사구, 2단 bullet, `✓ ⚠ ✗ ▸` 심볼) 통일
- **작성 언어**: 스킬 본문은 영어, 런타임 출력은 프로젝트 언어 설정 준수

자세한 내용은 각 SKILL.md를 참고한다.

</details>

## 프로젝트 구조

이 저장소는 `.claude/` 디렉토리 구조를 그대로 따른다.

```
claude-skills/
├── skills/                             # 스킬 (.claude/skills/ 대응)
│   ├── claude-md-writer/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── best-practices.md
│   │       └── section-guide.md
│   ├── new-feature/                    # 래퍼 스킬
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── report-template.md      # 개조식 최종 보고 템플릿
│   ├── feature-research/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   ├── plan-schema.md          # Plan 파일 공유 계약
│   │   │   └── plan-template.md
│   │   └── examples/
│   │       └── good-plan.md
│   ├── feature-work/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── task-breakdown.md
│   ├── feature-test/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── test-patterns.md
│   └── feature-review/
│       ├── SKILL.md
│       └── references/
│           └── review-checklist.md
├── agents/                             # 에이전트 (.claude/agents/ 대응)
│   ├── external-research.md
│   └── code-implementer.md
├── docs/
│   └── design-philosophy.md
├── .claude/rules/                      # path-scoped 규칙
│   └── skill-authoring.md
├── README.md
└── LICENSE
```

## 나만의 확장 만들기

### 스킬 만들기

```
your-skill/
├── SKILL.md    # 스킬 정의 (필수)
└── references/ # 참조 문서 (선택)
```

**SKILL.md** 필수 frontmatter:

```yaml
---
name: your-skill
description: "스킬에 대한 한 줄 설명"
argument-hint: "<인자 설명>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---
```

자세한 작성법은 [Claude Code Skills 공식 문서](https://code.claude.com/docs/en/slash-commands)를 참고한다.

### 에이전트 만들기

에이전트는 마크다운 파일 하나로 정의한다:

```yaml
---
name: my-agent
description: "에이전트 설명"
model: sonnet
tools: Read, Grep, Glob
---

에이전트의 시스템 프롬프트를 여기에 작성한다.
```

자세한 작성법은 [Claude Code Subagents 공식 문서](https://code.claude.com/docs/en/sub-agents)를 참고한다.

## 기여하기

기여를 환영합니다! 다음 절차를 따라주세요:

1. 이 저장소를 **Fork** 한다
2. 기능 브랜치를 생성한다 (`git checkout -b feature/amazing-skill`)
3. 변경사항을 커밋한다 (`git commit -m 'Add amazing-skill'`)
4. 브랜치에 푸시한다 (`git push origin feature/amazing-skill`)
5. **Pull Request**를 생성한다

### 기여 가이드라인

- 스킬은 한 가지 목적에 집중한다
- SKILL.md에 워크플로우를 명확히 문서화한다
- 기존 스킬의 패턴과 일관성을 유지한다
- `skills/` 또는 `agents/` 하위에 올바른 위치에 추가한다

## 라이선스

이 프로젝트는 [MIT License](LICENSE)를 따른다.
