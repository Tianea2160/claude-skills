# 좋은 Plan 예시

아래는 "기존 REST API에 WebSocket 실시간 알림 기능 추가" 요청에 대한 구현 계획 예시다.
Plan 모드에서 이 수준의 구체성으로 작성해야 한다.

---

## 목표

주문 상태 변경 시 클라이언트에 실시간 알림을 전달하기 위해 WebSocket 엔드포인트를 추가한다. 기존 polling 방식(5초 간격)을 대체하여 사용자 경험과 서버 부하를 개선한다.

## 구현 전략

**Server-Sent Events(SSE) 대신 WebSocket을 선택한 이유:**
- 향후 클라이언트 → 서버 메시지(읽음 확인 등)가 필요하므로 양방향 통신이 유리하다
- 기존 프로젝트에서 `socket.io`를 devDependency로 이미 포함하고 있어 추가 의존성 없이 활용 가능하다

**인증 방식:**
- WebSocket handshake 시 기존 JWT 토큰을 query parameter로 전달한다
- 기존 `authMiddleware`의 검증 로직을 재사용하되, Express middleware가 아닌 순수 함수로 추출하여 WebSocket에서도 호출 가능하게 한다

## 변경 계획

| 파일 | 변경 유형 | 구체적 내용 |
|------|---------|------------|
| `src/middleware/auth.ts` | 수정 | `verifyToken()` 로직을 `validateJWT(token): Promise<User>` 순수 함수로 추출. 기존 `authMiddleware`는 이 함수를 호출하는 래퍼로 변경하여 하위 호환 유지 |
| `src/websocket/handler.ts` | 생성 | WebSocket 연결 관리 모듈. `handleConnection(socket)`, `handleDisconnect(socket)`, `broadcastToUser(userId, event)` 3개 함수 |
| `src/websocket/index.ts` | 생성 | socket.io 서버 초기화 및 `validateJWT`를 사용한 인증 미들웨어 등록 |
| `src/services/order.service.ts` | 수정 | `updateOrderStatus()` 메서드 끝에 `broadcastToUser(order.userId, { type: 'ORDER_STATUS', payload: order })` 호출 추가 (1줄) |
| `src/index.ts` | 수정 | HTTP 서버 생성 후 `initWebSocket(server)` 호출 추가 (2줄) |
| `src/types/websocket.ts` | 생성 | `WebSocketEvent`, `OrderStatusEvent` 타입 정의 |
| `test/websocket/handler.test.ts` | 생성 | 연결/인증/브로드캐스트 단위 테스트 |

## 재사용할 기존 코드

- `src/middleware/auth.ts:15-28` — JWT 검증 로직. 순수 함수로 추출하여 WebSocket 인증에 재사용
- `src/types/order.ts:OrderStatus` — 주문 상태 enum. WebSocket 이벤트 타입에서 import
- `src/services/order.service.ts:45` — `updateOrderStatus()` 메서드. 마지막에 broadcast 호출만 추가
- `test/helpers/auth.helper.ts:createTestToken()` — 테스트용 JWT 생성. WebSocket 테스트에서 그대로 사용

## 작업 순서

### 1. validateJWT 순수 함수 추출
- **내용**: `auth.ts:15-28`의 JWT 검증 로직을 `validateJWT(token): Promise<User>` 순수 함수로 추출. 기존 `authMiddleware`는 이 함수를 호출하는 래퍼로 변경하여 하위 호환 유지
- **의존**: 없음

### 2. WebSocket 이벤트 타입 정의
- **내용**: `src/types/websocket.ts` 생성. `WebSocketEvent`, `OrderStatusEvent` 타입 정의. 기존 `OrderStatus` enum을 import하여 재사용
- **의존**: 없음 (1번과 병렬 가능)

### 3. WebSocket 핸들러 구현
- **내용**: `src/websocket/handler.ts` 생성. `handleConnection(socket)`, `handleDisconnect(socket)`, `broadcastToUser(userId, event)` 3개 함수 구현
- **의존**: 1, 2번 완료 필요

### 4. WebSocket 초기화 및 서버 연결
- **내용**: `src/websocket/index.ts` 생성하여 socket.io 서버 초기화 및 인증 미들웨어 등록. `src/index.ts`에 `initWebSocket(server)` 호출 추가 (2줄)
- **의존**: 3번 완료 필요

### 5. 주문 서비스에 broadcast 연동
- **내용**: `order.service.ts`의 `updateOrderStatus()` 메서드 끝에 `broadcastToUser(order.userId, { type: 'ORDER_STATUS', payload: order })` 호출 추가 (1줄)
- **의존**: 3번 완료 필요 (4번과 병렬 가능)

### 6. 테스트 작성 및 실행
- **내용**: `test/websocket/handler.test.ts` 생성. 연결/인증 실패/브로드캐스트 단위 테스트 작성. `createTestToken()` 헬퍼 재사용
- **의존**: 모든 구현 완료 후

## 외부 참고사항

- socket.io v4 공식 문서에서 Express 서버와의 통합 시 `createServer`로 HTTP 서버를 먼저 생성해야 한다 (기존 `app.listen()` 패턴 변경 필요)
- JWT를 WebSocket query parameter로 전달하면 서버 로그에 토큰이 노출될 수 있으므로, 프로덕션에서는 연결 후 첫 메시지로 토큰을 전송하는 방식이 더 안전하다 (1차 구현은 query parameter, 후속 개선으로 전환)

---

## 이 예시가 좋은 이유

1. **구체적이다** — "WebSocket 추가"가 아니라 어떤 파일에 어떤 함수를 어떻게 변경하는지 명시
2. **근거가 있다** — SSE 대신 WebSocket을 택한 이유, 인증 방식의 이유를 설명
3. **재사용을 보여준다** — 기존 코드를 file:line으로 참조하여 중복 구현 방지
4. **TaskCreate와 직접 매핑된다** — 각 항목의 제목이 `subject`, 내용이 `description`으로 그대로 사용 가능. 의존성도 명시
5. **위험을 인지한다** — 외부 참고사항에서 보안 고려사항과 단계적 접근을 제시
