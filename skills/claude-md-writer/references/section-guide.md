# 섹션별 작성 가이드

CLAUDE.md의 각 섹션에 무엇을 포함하고 제외할지 안내한다.
모든 섹션이 필수는 아니다 — 프로젝트에 관련된 섹션만 사용한다.

## 프로젝트 설명

**한 줄로.** 프로젝트가 무엇인지 Claude에게 맥락을 준다.

```markdown
# my-project

Spring Boot 기반 소셜 인증 API 서버. PostgreSQL + Redis 사용.
```

- 기술 스택을 간략히 포함하면 Claude가 적절한 패턴을 적용
- 비즈니스 로직 설명 불필요

## 명령어

**가장 중요한 섹션.** Claude가 빌드/테스트를 못 하면 생산성이 급락한다.

```markdown
## 명령어

| 명령어 | 설명 |
|-------|------|
| `./gradlew build` | 전체 빌드 |
| `./gradlew test` | 테스트 실행 |
| `./gradlew :module:test --tests "*.ClassName"` | 단일 테스트 |
| `npm run dev` | 개발 서버 (포트 3000) |
```

- 표준 명령어라도 프로젝트별 플래그가 있으면 포함
- 단일 테스트 실행 방법은 반드시 포함

## 아키텍처

**핵심 디렉토리와 용도만.** 파일 단위까지 내려가지 않는다.

```markdown
## 아키텍처

\```
src/
├── api/          # REST 컨트롤러
├── domain/       # 비즈니스 로직 (DDD)
├── infra/        # 외부 시스템 연동
└── config/       # 설정 클래스
\```
```

- 3-5 depth까지만
- 용도가 이름에서 명확하면 주석 생략

## 코드 스타일

**기본값과 다른 규칙만.** "camelCase 사용" 같은 언어 기본은 쓰지 않는다.

```markdown
## 코드 스타일

- 에러 응답은 항상 `ErrorResponse` sealed class 사용
- Repository 메서드명: `findByXxx` (Spring Data 컨벤션 따름)
- 환경별 설정은 `application-{profile}.yml`에만
```

## 환경

**셋업에 필요한 것만.**

```markdown
## 환경

필수:
- Java 21+
- Docker (PostgreSQL, Redis 로컬 실행용)
- `KAKAO_REST_API_KEY` - 카카오 OAuth 인증용

셋업:
- `cp .env.example .env` 후 값 채우기
```

## 주의사항 (Gotchas)

**가장 가치 높은 섹션.** 코드에서 알 수 없고, 실수를 방지하는 정보.

```markdown
## 주의사항

- PVC 삭제 금지 — 데이터 손실 위험
- `staging` 네임스페이스는 여러 앱이 공유함
- Helm 외부 차트는 반드시 버전 고정
- `base/` 변경 시 모든 환경에 영향
```

## 워크플로우

**팀 규칙이 있을 때만.**

```markdown
## 워크플로우

- 브랜치: `feature/ISSUE-123-description`
- PR 머지: squash merge only
- main 직접 push 금지
```

## 시크릿 관리

**시크릿 위치와 관리 방법만. 실제 값은 포함하지 않는다 (private repo 예외).**

```markdown
## 시크릿

- `postgresql-secret` (namespace: prod) — DB 자격증명
- `.env` — 로컬 개발용 (gitignored)
```

## Rules 파일 (.claude/rules/)

CLAUDE.md에서 분리할 수 있는 path-scoped 규칙.
특정 파일 유형/디렉토리에만 적용되는 규칙에 적합.

```yaml
# .claude/rules/api-design.md
---
paths:
  - "src/api/**/*.kt"
---

# API 작성 규칙

- 모든 엔드포인트에 입력 검증
- 에러 응답은 ErrorResponse sealed class
- Controller에 비즈니스 로직 금지
```

### Rules vs CLAUDE.md 판단 기준

| 기준 | CLAUDE.md | Rules |
|------|-----------|-------|
| 적용 범위 | 프로젝트 전체 | 특정 경로만 |
| 로드 시점 | 항상 | 해당 파일 작업 시 |
| 토큰 비용 | 항상 소비 | 필요할 때만 |
| 예시 | 빌드 명령어, 전체 아키텍처 | API 작성 규칙, 테스트 패턴 |
