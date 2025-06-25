# DBCP (Database Connection Pool) 

## 1. DBCP
데이터베이스와의 연결을 미리 생성하여 풀(Pool)에 저장해두고, 필요할 때마다 이를 재사용하는 방식의 기술. (Thread Pool과 유사한듯)
이는 매번 DB 연결을 생성하고 종료하는 비용을 줄여서성능을 향상시키는 것이 목적.

---

## 2. DBCP 동작 구조

```text
[Application] ⇄ [Connection Pool] ⇄ [Database]
                     ↑           ↓
         (빌림/반납, 유휴/최대 커넥션 관리)
```

1. 초기 커넥션 풀 생성 (minIdle, initialSize)
2. 클라이언트가 요청 시 커넥션 대여
3. 처리 완료 후 커넥션을 풀에 반납
4. 커넥션이 유휴 상태로 대기
5. 필요 시 maxTotal 제한 내에서 커넥션 추가 생성

---

## 4. 주요 설정 항목

| 설정 항목         | 설명 |
|------------------|------|
| `initialSize`     | 초기 생성할 커넥션 수 |
| `maxTotal`        | 최대 커넥션 수 (동시 접속 제한) |
| `minIdle`         | 최소 유휴 커넥션 수 유지 |
| `maxIdle`         | 최대 유휴 커넥션 수 제한 |
| `maxWaitMillis`   | 커넥션 대기 최대 시간 |
| `validationQuery` | 커넥션 유효성 검사 쿼리 (예: `SELECT 1`) |

---

## 5. 대표적인 DBCP 구현체

| 구현체         | 설명 |
|----------------|------|
| **Apache DBCP** | Tomcat에서 기본 제공, 안정적인 성능 |
| **HikariCP**    | 경량, 고성능, 빠른 속도로 유명함 (Spring Boot 기본) |
| **C3P0**        | 설정이 풍부하고 안정적이지만 상대적으로 느림 |
| **BoneCP**      | 과거 많이 사용되었으나 현재는 HikariCP에 밀림 |

---

## 6. Spring Boot에서의 HikariCP 설정 예시
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: user
    password: pass
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      idle-timeout: 30000
      max-lifetime: 1800000
      connection-timeout: 30000
      validation-timeout: 5000
```

---

## 7. 장점
- **성능 향상**: 커넥션 생성/소멸 비용 절감
- **리소스 효율**: 커넥션 수 제한으로 DB 과부하 방지
- **안정성 향상**: 커넥션 유효성 체크 가능

## 8. 주의사항
- 풀 크기를 너무 작게/크게 설정 시 병목 또는 리소스 낭비
- 유휴 커넥션 유지 시 timeout 설정 필요 (유령 커넥션 방지)
- validation query 설정으로 유효하지 않은 커넥션 검출 필요
