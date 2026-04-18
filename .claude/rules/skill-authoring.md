---
paths:
  - "skills/*/SKILL.md"
---

# 스킬 정의 규칙

> 참조: [Extend Claude with skills — 공식 문서](https://code.claude.com/docs/en/skills)

## Frontmatter 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | 선택 | 생략 시 디렉토리명 사용. 디렉토리명과 다를 때만 명시 |
| `description` | **권장** | Claude 자동 호출 판단에 사용. `when_to_use`와 합산 **1,536자 제한** |
| `when_to_use` | 선택 | 추가 트리거 조건. `description`과 합산 1,536자 |
| `argument-hint` | 선택 | `/skill <args>` 자동완성 힌트. 예: `"<기능 설명>"` |
| `allowed-tools` | 선택 | 승인 없이 사용 가능한 도구 (**허용 목록**, 나머지 도구를 차단하지 않음) |
| `disable-model-invocation` | 선택 | 배포·커밋 등 부작용 있는 스킬은 `true` 권장 |
| `user-invocable` | 선택 | `false`로 설정 시 `/` 메뉴에서 숨김, Claude만 호출 가능 |
| `context` | 선택 | `fork` 설정 시 격리된 서브에이전트에서 실행 |

## 구조 규칙

- 사용자 질문·선택 요청은 반드시 **AskUserQuestion 도구** 사용 (일반 텍스트 질문 금지)
- 구현 승인은 **EnterPlanMode → 계획 작성 → ExitPlanMode** 순서
- Phase 4 보고는 **모델 텍스트로 직접 출력** — Bash printf 사용 금지 (Claude Code UI에서 축소됨)
- 외부 웹 조사는 `general-purpose`가 아닌 **`external-research` 에이전트** 사용
  (general-purpose는 WebSearch/WebFetch가 deferred 상태로 시작되어 0 tool use 조기 종료 사례 있음)

## 작성 언어

- 신규 SKILL.md 본문, frontmatter `description`, `references/` 하위 문서는 **영어로 작성**한다
- 런타임에 사용자에게 노출되는 출력(AskUserQuestion 텍스트, 보고 출력 등)은 프로젝트 CLAUDE.md의 language 설정을 따른다
- 근거: Claude의 description 기반 자동 호출 성능이 영어에서 안정적이며, 글로벌 스킬 ecosystem 호환성을 확보하기 위함
- 기존 한국어로 작성된 스킬은 이 규칙을 소급 적용하지 않는다 — 재작성 시점에 영어로 전환

## 보고 양식

- 모든 스킬의 최종/mini-보고는 **개조식(outline)** 으로 렌더링한다
- 완결 문장 금지, 명사구·동사구 중심 bullet
- 계층은 2단까지 (`-`, `  -`)
- 심볼 팔레트 고정: `✓` 성공/완료, `⚠` 경고/부분, `✗` 실패/블로커, `▸` 섹션 헤더
- 수치·상태어·고유 식별자는 `**볼드**` 또는 인라인 코드로 강조
- Bash printf/echo 출력 금지 — 항상 모델 텍스트로 출력

## 참조 문서 패턴

- `references/` 하위 문서는 SKILL.md에서 상대 경로로 링크
- Claude가 런타임에 필요할 때만 읽도록 설계 (SKILL.md 본문에 내용 복사 금지)

## 주의

Path-scoped rules는 **Read 연산에서만 로드**되며 Write(파일 생성) 시에는 적용되지 않음.
새 SKILL.md 생성 규범은 루트 `CLAUDE.md`의 "스킬 파일 규약" 섹션을 기준으로 함.
([GitHub Issue #23478](https://github.com/anthropics/claude-code/issues/23478) 참조)
