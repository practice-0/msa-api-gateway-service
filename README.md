# Gateway Service

> 외부 요청의 단일 진입점으로서 인증을 처리하고, 내부 서비스로 안전하게 라우팅하는 역할을 담당합니다.

## 설계 의도
- 모든 외부 요청을 Gateway 단일 진입점으로 통제
- 내부 서비스 직접 노출 방지
- 인증/인가 로직을 서비스에서 분리

## 인증 처리 방식

- `Authentication` 헤더에서 JWT 추출
- Gateway에서 서명 검증
- subject(userId) 파싱
- 내부 서비스로 `X-User-Id` 헤더 전달

→ 내부 서비스는 JWT 파싱 로직을 가지지 않음
→ 인증 책임을 Gateway에 집중

### 동작 세부 흐름

1. `Authentication` 헤더에서 JWT 추출
2. HMAC 기반 서명 검증 (`jwt.secret` 사용)
3. 토큰의 subject(userId) 파싱
4. 내부 요청 헤더에 `X-User-Id` 추가
5. 필터 체인을 통해 실제 서비스로 전달

## 예외 처리 전략

- 토큰이 없을 경우 401 Unauthorized 반환
- JWT 서명 검증 실패 시 요청 차단

## 요청 흐름

```
Client
  ↓
API Gateway (JWT 검증 + Header 변환)
  ↓
Internal Service (Board / User / Point)
```

## 기술 스택
- Spring Boot 3.x
- Spring Cloud Gateway
- jjwt (JWT 파싱 및 검증)

## 실행 방법
docker-compose up -d

## 환경 변수

| 변수명 | 설명 |
|--------|------|
| jwt.secret | JWT 서명 검증에 사용되는 HMAC Secret Key |

application.yml 또는 환경 변수로 주입하여 사용합니다.

## 설계 포인트

- 인증 로직을 Gateway에 집중시켜 내부 서비스의 책임을 최소화
- 내부 서비스는 사용자 식별을 `X-User-Id` 헤더 기반으로 처리
- 인증 실패 시 즉시 401 반환하여 불필요한 내부 호출 차단
