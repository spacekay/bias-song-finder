[🇺🇸 English](../backend.md)

# 백엔드 기술 스택

## 핵심 프레임워크

| 기술 | 비고 |
|-----------|------|
| Kotlin | 주 언어 |
| Java 21 | 툴체인 |
| Spring Boot 3.4 | Web, Data JPA, Security, Data Redis, Mustache (어드민) |
| Gradle (Kotlin DSL) | 빌드 시스템 |

---

## 데이터베이스

| 기술 | 역할 |
|-----------|------|
| PostgreSQL | 주 데이터 저장소 (schema-per-tenant) |
| Hibernate (JPA) | ORM — validate 모드 (스키마는 Flyway로 관리) |
| Flyway | 버전 관리 스키마 마이그레이션, 멀티스키마 지원 |
| hypersistence-utils | 복합 데이터 타입을 위한 JSONB 컬럼 타입 매핑 |

### JSONB 활용

리스트형 및 구조화된 필드를 PostgreSQL JSONB 컬럼으로 저장합니다. 단순한 중첩 데이터에 대해 조인 테이블 오버헤드를 회피하면서도 쿼리 가능성을 유지합니다.

### Flyway 전략

공유 스키마와 테넌트별 스키마의 마이그레이션을 분리 관리합니다. 테넌트 스키마 마이그레이션은 애플리케이션 시작 시 각 테넌트에 적용됩니다.

---

## 세션 & 캐시

| 기술 | 역할 |
|-----------|------|
| Spring Data Redis | TTL 기반 세션 데이터 저장 |
| Redisson | 분산 rate limiting |

세션 상태는 전적으로 Redis에 저장됩니다 — 백엔드에 인메모리 상태가 없으므로 수평 확장이 가능합니다. API 엣지에서는 Redisson 기반 분산 rate limiting을 적용하여 인스턴스 전반에 걸쳐 세션 단위 남용을 제어합니다.

---

## 보안

| 기술 | 역할 |
|-----------|------|
| Spring Security | 무상태 세션, 경로 기반 인증 규칙 |
| JJWT | JWT 토큰 생성 및 검증 |

- 프론트엔드가 공유 시크릿으로 인증하여 단기 JWT를 발급받음
- 퀴즈 엔드포인트는 세션 키를 식별자로 사용하며 JWT 인증에서 제외
- 분산 rate limiting으로 세션 단위 남용 방지

---

## 곡 유사도 엔진

여러 속성 카테고리를 가중치 기반으로 종합해 가까운 곡을 랭킹으로 산출하는 유사도 엔진을 운영합니다. 후보 풀이 비거나, 모든 가능한 질문이 소진되거나, 사용자가 no-match 경로에 도달하는 등 한 곡으로 좁혀지지 않는 모든 시나리오에서 호출되어 — 응답이 에러 대신 의미 있는 추천 결과를 항상 담도록 합니다.

---

## 테스팅

| 기술 | 역할 |
|-----------|------|
| MockK | 서비스 레이어 단위 테스트 |
| Testcontainers | 실제 PostgreSQL + Redis 컨테이너 기반 통합 테스트 |
| JaCoCo | 코드 커버리지 강제 |

---

## 어드민 UI

`:admin` 모듈은 서버 렌더링 관리 인터페이스(Mustache 템플릿, 로컬 전용)를 제공하며, 모든 도메인 데이터 관리 및 CSV 일괄 가져오기를 지원합니다. 콘텐츠 큐레이션을 돕기 위해 Spotify, YouTube 연동도 어드민 플로우에 통합되어 있습니다.

---

## API 문서

SpringDoc OpenAPI — API 소비자를 위한 JSON 스펙 제공
