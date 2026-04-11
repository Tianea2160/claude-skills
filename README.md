<h1 align="center">Claude Skills</h1>
<p align="center">
  Claude Code를 위한 커스텀 스킬 & 에이전트 컬렉션
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href="#스킬-목록"><img src="https://img.shields.io/badge/skills-2-brightgreen.svg" alt="Skills"></a>
  <a href="#에이전트-목록"><img src="https://img.shields.io/badge/agents-coming_soon-yellow.svg" alt="Agents"></a>
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
| [new-feature](skills/new-feature/SKILL.md) | 리서치 → 작업 → 검토 → 보고 4단계 기능 개발 워크플로우 | `/new-feature <기능 설명>` |

## 에이전트 목록

> 에이전트는 준비 중입니다. 곧 추가될 예정입니다.

## 빠른 시작

### 1. 저장소 클론

```bash
git clone https://github.com/Tianea2160/claude-skills.git
```

### 2. 설치

이 저장소는 `.claude/` 디렉토리 구조를 그대로 따르므로, 원하는 항목을 프로젝트의 `.claude/` 경로에 복사하면 된다.

```bash
# 스킬 설치
cp -r claude-skills/skills/claude-md-writer /your-project/.claude/skills/
cp -r claude-skills/skills/new-feature /your-project/.claude/skills/

# 에이전트 설치 (추후 추가 시)
cp -r claude-skills/agents/<agent-name>.md /your-project/.claude/agents/
```

개인 전역 설치 (모든 프로젝트에서 사용):

```bash
# 스킬 전역 설치
cp -r claude-skills/skills/claude-md-writer ~/.claude/skills/
cp -r claude-skills/skills/new-feature ~/.claude/skills/

# 에이전트 전역 설치 (추후 추가 시)
cp -r claude-skills/agents/<agent-name>.md ~/.claude/agents/
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
<summary><b>new-feature</b> — 체계적 기능 개발 워크플로우</summary>

### 핵심 기능

- **리서치**: 요구사항 수집, 코드베이스 분석, 외부 best practice 조사
- **작업**: Task 기반으로 분해하여 순차 실행
- **검토**: 요구사항 정합성, 엣지 케이스, 보안 취약점 전수 검토
- **보고**: 구현 내용, 변경 파일, 검증 결과를 구조화된 형식으로 보고

### 워크플로우

```
리서치 → [승인 게이트] → 작업 → 검토 → 보고
```

1. 요구사항을 명확히 하고 코드베이스를 분석한다
2. 접근 방식을 제안하고 **사용자 승인을 받은 후** 진행한다
3. TaskCreate로 작업을 분해하고 순차적으로 구현한다
4. 결과물을 전수 검토하고 문제를 수정한다
5. 구조화된 보고서로 결과를 전달한다

자세한 내용은 [SKILL.md](skills/new-feature/SKILL.md)를 참고한다.

</details>

## 프로젝트 구조

이 저장소는 `.claude/` 디렉토리 구조를 그대로 따른다.

```
claude-skills/
├── skills/                         # 스킬 (.claude/skills/ 대응)
│   ├── claude-md-writer/
│   │   ├── SKILL.md                # 스킬 정의
│   │   └── references/
│   │       ├── best-practices.md   # CLAUDE.md 작성 best practice
│   │       └── section-guide.md    # 섹션별 작성 가이드
│   └── new-feature/
│       ├── SKILL.md                # 스킬 정의
│       └── references/
│           ├── examples.md         # 작업 분해 및 병렬 실행 예시
│           ├── review-checklist.md # 코드 품질 검증 체크리스트
│           └── templates.md        # 리서치/보고 형식 템플릿
├── agents/                         # 에이전트 (.claude/agents/ 대응)
│   └── .gitkeep
├── docs/
│   └── design-philosophy.md        # 설계 철학 및 근거
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
