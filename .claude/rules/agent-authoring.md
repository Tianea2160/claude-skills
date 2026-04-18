---
paths:
  - "agents/*.md"
---

# 에이전트 정의 규칙

> 참조: [Agent tool — 공식 문서](https://code.claude.com/docs/en/agents)  
> 참조: [Equipping agents for the real world with Agent Skills — Anthropic Engineering](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

## Frontmatter 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | 권장 | 에이전트 식별자 (subagent_type 값으로 사용됨) |
| `description` | **필수** | Claude가 위임 대상 선택에 사용. 구체적일수록 정확한 자동 위임 가능 |
| `tools` | **권장** | 사용할 도구를 명시 — 미지정 시 환경에 따라 deferred 상태로 시작되어 0 tool use 종료 위험 |
| `model` | 선택 | 기본값은 호출자 모델 상속. 비용·속도 조정 시 명시 |

## 필수 섹션

에이전트 파일은 아래 섹션을 모두 포함해야 한다:

1. **전략** — 작업 수행 순서와 방법 (쿼리 구성, 교차 검증 방법 등)
2. **출력 형식** — 호출자가 기대하는 마크다운 구조를 코드 블록으로 예시
3. **실패 처리** — 도구 접근 불가·결과 빈약 시 동작 명시 (**0 tool uses로 종료 금지**)
4. **금지 사항** — 해서는 안 되는 행동 (예: 출처 없는 주장, 코드 수정 등)

## 작성 언어

- 에이전트 파일의 **frontmatter `description` 및 본문 전체는 영어로 작성**한다 (예외 없음)
- 런타임에 사용자에게 노출되는 출력(에이전트가 반환하는 요약, 보고 등)은 **호출자가 지정한 언어 또는 프로젝트 CLAUDE.md의 language 설정**을 따른다
- 근거:
  - Claude의 `description` 기반 자동 위임 판단 성능은 영어에서 일관되게 작동한다
  - 글로벌 subagent ecosystem과의 호환성을 확보한다
- 기존 한국어로 작성된 에이전트는 **재작성 시 영어로 전환**한다 (`external-research`, `code-implementer` 등)

## 주의

Path-scoped rules는 **Read 연산에서만 로드**되며 Write(파일 생성) 시에는 적용되지 않음.
([GitHub Issue #23478](https://github.com/anthropics/claude-code/issues/23478) 참조)
