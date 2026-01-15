# Claude Code 개발 지침 (환자 의료기록 상세 저장 플랫폼)

## 1. Overview (개요)

- **Project:** `pedicle-note-api`
- **Purpose:** 환자 진료 기록을 “자유 텍스트”가 아니라 **구조화된 임상 데이터**(증상/부위/기간/치료/판독 등)로 저장하여, 향후 **검색·분석·추천·코드(상병) 제안**까지 가능한 기반을 만든다.
- **Core Concept:** 모든 기록은 “진료 이벤트(Encounter)”를 중심으로 축적되며 진단 결과만 저장하는 것이 아니라, 진단에 이르는 **'근거 데이터'를 분해하여 저장**함으로써 유사 환자 추천 및 진료 패턴 분석이 가능한 자산으로 만드는 것이 핵심이다. 그리고 멀티테넌트 환경에서 **테넌트 격리 + 의료정보 보안 + 변경 추적(Audit)**을 최우선으로 한다.

---

## 2. Tech Stack (기술 스택)

- **Framework:** Spring Boot 3.x (Spring MVC)
- **Auth:** Spring Security + JWT (자체 인증, OAuth/Keycloak 미사용)
- **Database:** MariaDB (멀티테넌트 `tenant_id` 기반 논리 격리)
- **Persistence:** JPA (Hibernate) + Querydsl
- **Multi-Tenancy:** 요청 스코프 `TenantContext` + `tenant_id` 강제 조건(전 테이블/전 쿼리)
- **Docs:** `/docs` 기반 PRD/ADR/ERD 관리

---

## 3. Command (실행 및 테스트)

* **Server Run:** `./gradlew bootRun`
* **Build:** `./gradlew build`
* **Test All:** `./gradlew test` (Junit5 기반, 서비스 로직 및 데이터 정합성 검증 필수)
* **Querydsl Compile:** `./gradlew compileJava` (QClass 생성 확인)
* **Config:** `application.yml`, `application-dev.yml`

---

## 4. Product Invariants (제품 불변 원칙)

기획, 설계, 구현 전 단계에서 타협할 수 없는 `pedicle-note-api`의 가드레일입니다.

### 4.1 도메인 불변 원칙 (Product Truths)

- **S-T-D 데이터 구조 유지:** 모든 차팅 데이터는 상황(S), 치료(T), 기간(D)이라는 3가지 축을 기준으로 구조화되어 저장되어야 하며, 이 연결 고리가 끊어져서는 안 된다.

  - S-T-D는 Core Axis이며, 확장 Axis가 결합될 수 있다.(예시 anatomy, severity, diagnostics etc.)
- **기록의 증거성:** 의료기록은 “나중에 증명 가능한 로그”여야 한다.

  - 누가/언제/무엇을 입력·수정했는지 추적(Audit)이 불가능한 설계는 금지한다.
- **임상 데이터의 구조화 및 자산화:** 진료 정보는 가능한 한 구조화(축/코드/선택지)로 저장한다.

  - 자유 텍스트는 보조 수단이며, 구조화 데이터가 “주인”이다.
  - 모든 진료 기록은 향후 유사 환자 추천 및 AI 학습을 위해 분석 가능한 형태(Structured)로 저장되어야 한다.
- **시간축(Timeline) 보존:** 증상 onset, 치료 시작, 경과/재발 우려 기간 등 “시간 정보”는 추론/분석의 핵심이므로 누락 없이 모델링한다.
- **식별자/참조의 일관성:** 환자/진료/관찰/치료/판독 등의 관계는 항상 일관된 외래키/참조 규칙을 갖는다(느슨한 문자열 참조 금지).
- **근거 기반 상병 제안:** 상병 코드(ICD)는 단순 입력값이 아니라, 입력된 증상과 부위(Anatomy) 데이터를 기반으로 **추론 및 제안**되는 구조를 유지한다.
- **지식 그래프 기반 추론:** ICD 코드는 단순 검색 대상이 아니라, 증상과 부위의 조합으로 유도되는 지식 노드여야 한다.

### 4.2 시스템 동작 제약 (System Constraints)

- **보안 우선:** PHI(개인건강정보) 성격의 데이터는 기본적으로 민감정보로 취급한다.

  - 최소권한(Principle of Least Privilege) + 접근통제 + 감사로그는 필수다.
- **UX 우선주의:** 구조화가 입력의 방해가 되어서는 안 된다. 최소한의 노력으로 최대의 구조화 데이터를 얻는 흐름을 지향한다.
- **Strict Multi-tenancy:** 어떤 상황에서도 타 병원(Tenant)의 데이터가 노출되거나 섞여서는 안 된다.

---

## 5. Universal Thinking Process (공통 사고 프로세스)

클로드는 모든 요청을 수행하기 전, 아래의 **'헌법적 사고 단계'**를 반드시 거쳐야 한다.

### 1. [Clinical Context] 의료적 의도 파악

- 이 기능이 의사의 **어떤 진료 행위**를 돕는가?
- 이러한 의사의 진료 행위는 환자에게 어떤 도움이 되는가?
- 혹은 **어떤 임상 데이터(S/T/D, 부위, 판독 등)**를 생성·변경·해석하는 작업인가?

### 2. [S-T-D Alignment] 데이터 축 정합성

- 해당 작업이 **상황(S) / 치료(T) / 경과(D)** 중 **어느 축에 속하는지 명시**한다.
- 축 간 책임을 혼합하지 않는가?
- 이 데이터가 향후 **유사 환자 추천 또는 임상 분석의 Feature**로 활용 가능하도록 설계되었는가?

### 3. [Knowledge Mapping] 지식 연결성

- 이 데이터는 **ICD-10/11, 해부학적 구조(Anatomy)** 등과
  - (a) 지금 직접 연결되어야 하는가?
  - (b) 추후 연결을 위한 확장 포인트로 남겨야 하는가?
- 이 연결 방식이 향후 **추론/추천 엔진**에 기여하는 구조인가?

### 4. [Physician UX] 의사 관점 검증

- 이 설계가 진료 현장에서 **의사에게 추가적인 번거로움**을 주지 않는가?
- 강제 입력이 아니라, **자연스러운 구조화(Natural Structuring)**가 가능한가?

### 5. [Integrity Check] 헌법 위배 여부

- `tenant_id` 격리가 모든 경로에서 보장되는가?
- 의료 데이터 보안 및 접근 통제 원칙을 위반하지 않는가?
- 데이터 변경 시 **추적 가능성(Audit/History)**이 보장되는가?
- 이 작업이 프로젝트의 **'데이터 자산화' 목적**에 부합하는가?

---

## 6. MCP (Model Context Protocol)

- **`sequential-thinking`:** 복잡한 임상 데이터 모델간의 관계 및 ICD 추론 로직 설계 또는 설계 및 트랜잭션 흐름 정리
- **`shrimp-task-manager`:** 기능 단위(Fxxx) 작업 분해/로드맵 추적
- **`context7`:** PRD ↔ ERD ↔ API 스펙 간 정합성 교차 검증

---

## 7. Sub Agents (역할 분담) - 추후 수정

- **prd-writer:** 의료기록 입력 플로우/필드 정의, In/Out Scope, 엣지케이스 정리
- **data-modeler:** ERD/엔티티/제약조건/인덱스/Soft Delete/Audit 설계
- **api-designer:** REST API 스펙(검색/필터/정렬/페이지네이션), DTO/응답 규격 설계
- **security-reviewer:** JWT/권한모델/테넌트 격리 누락/정보노출 위험 점검
- **querydsl-optimizer:** Querydsl 조회 성능(조인/페치 전략/인덱스) 리뷰

---

## 8. Docs (Knowledge Map)

- **`/docs/10-product`** : PRD, 용어집(증상/부위/기간/치료/판독 정의), 사용자 시나리오
- **`/docs/20-architecture`** : 레이어 구조, 멀티테넌트 전략, 보안/감사 정책
- **`/docs/30-engineering`** : 코딩 컨벤션, 테스트 규칙, Querydsl 패턴, 마이그레이션 규칙
- **`/docs/40-planning`** : ADR(의사결정 기록), 로드맵/릴리즈 노트

---

## 9. Language & Communication

- **Primary Language:** 대화/PRD/설계 문서는 **한국어**
- **Technical Terms:** 필요 시 영문 병기(JWT, Claim, Tenant Isolation, Soft Delete, Audit)
- **Code Style:** 코드/식별자/테이블/컬럼명은 **영어**, 주석/설명은 **한국어**
- **Commit/PR 규칙(권장):** 변경 범위(tenant/security/model/query)를 제목에 명시

---

## 10. 핵심 철학 (Claude Code 행동 지침)

이 프로젝트는 단순 CRUD가 아니라 **“증거로 남는 의료기록 데이터 자산”**을 만든다.따라서 모든 구현에서 다음을 최우선으로 한다.

1) **테넌트 격리 누락 방지** (실수 방지 장치 포함)
2) **감사 가능성(Auditability)** (누가/언제/무엇을)
3) **구조화 데이터 우선** (분석/추천/코드 제안이 가능한 형태)
4) **보안 기본값(Secure by Default)** (최소권한·정보노출 방지)
