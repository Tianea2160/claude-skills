# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

Claude Code용 커스텀 스킬·에이전트 컬렉션. 빌드 시스템 없이 마크다운 파일로만 구성된다.

## 디렉토리 구조

```
skills/<name>/
  SKILL.md            — 스킬 정의 (frontmatter + 프롬프트 본문)
  references/         — 스킬이 런타임에 참조하는 문서
  examples/           — 좋은 출력 예시
agents/<name>.md      — 에이전트 정의
docs/                 — 설계 철학 등 프로젝트 문서
```

## 스킬 파일 규약

`SKILL.md` 필수 frontmatter:

```yaml
---
name: skill-name
description: "한 줄 설명. 사용법: /skill-name <args>"
argument-hint: "<args>"
allowed-tools: Read, Write, Edit, ...
---
```

에이전트 파일 필수 frontmatter:

```yaml
---
name: agent-name
description: 에이전트 설명
tools: Tool1, Tool2, ...
model: sonnet
---
```

## 새 스킬 추가 시

1. `skills/<name>/SKILL.md` 생성
2. `README.md`의 스킬 목록 테이블에 항목 추가

## 주요 설계 결정

**외부 조사는 반드시 `external-research` 에이전트 사용**  
`general-purpose` 에이전트는 환경에 따라 `WebSearch`/`WebFetch`가 deferred 상태로 시작되어 0 tool use로 조기 종료하는 사례가 반복 관측되었다. `external-research`는 웹 조사 도구를 frontmatter에서 고정 로드한다.

**Phase 4 보고는 모델 텍스트로 직접 출력**  
`Bash printf`로 출력하면 Claude Code UI에서 요약·축소되어 사용자에게 보이지 않는다. `**볼드**`와 유니코드 심볼(✓ ⚠ ✗)을 사용한 마크다운으로 직접 출력한다.

**CLAUDE.md는 200줄 이하 유지**  
200줄 초과 시 Claude의 후반부 지시 준수율이 하락한다 (Anthropic 권장사항).
