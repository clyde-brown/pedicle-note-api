# Pedicle Note API MVP PRD

## 🎯 핵심 정보

**목적**: S-T-D (Situation/Treatment/Duration) 3축 구조화를 통한 임상 데이터 자산화 플랫폼 구축
**사용자**: 전문의(Primary) 및 의료기관 관리자(Secondary)

## 🚶 사용자 여정 (Clinical & Admin Flow)

### 의사 여정 (Physician Journey)
1. **로그인** → 테넌트(병원) 인증 후 진료 콘솔 진입
2. **환자 검색/선택** → 기존 환자 조회 또는 신규 환자 등록
3. **진료 시작(Encounter)** → 새로운 진료 세션 생성
4. **S-T-D 기반 차팅**
   - Situation: 증상, 발생 부위, onset 시점, 중증도 입력
     - 신체검사: ROM, 근력, 특수검사, 압통점 등 이학적 소견 기록
     - 영상검사: X-ray, MRI, CT 등 검사 오더 및 판독 소견 입력
   - Treatment: 치료 계획, 처방, 시술 내용 기록
   - Duration: 예상 경과, 재진 일정, 회복 기간 설정
5. **검사 결과 종합** → 신체검사 + 영상검사 통합 뷰에서 진단 근거 확인
6. **ICD 코드 매핑** → 입력된 S-T-D 데이터 및 검사 소견 기반 자동 제안 → 확정
7. **유사 사례 확인** → 동일 증상-부위-검사소견 조합의 과거 케이스 조회
8. **진료 종료** → 데이터 저장 및 Audit 로그 생성

### 관리자 여정 (Admin Journey)
1. **관리자 로그인** → 테넌트 관리 콘솔 진입
2. **사용자 관리** → 의료진 계정 생성/권한 설정
3. **마스터 데이터 관리**
   - ICD-10/11 코드 테이블 유지보수
   - 해부학적 구조(Anatomy) 분류체계 관리
   - S-T-D 각 축별 선택지 커스터마이징
4. **진료 통계** → 주요 상병별 통계, 의사별 진료 패턴 분석
5. **데이터 품질 모니터링** → 미완성 차트, 코드 미매핑 건 추적

### 예외 흐름 (Exception Flows)
- **필수 데이터 누락**: S-T-D 중 최소 2개 축 이상 입력 필요 → 경고 표시
- **ICD 매핑 실패**: 자동 제안 불가 시 → 수동 검색/입력 모드 전환
- **동시 편집 충돌**: 동일 환자를 여러 의사가 동시 접근 → 읽기 전용 모드 또는 잠금 처리
- **테넌트 전환 시도**: 타 병원 데이터 접근 시도 → 401/403 차단

## 📜 제품 핵심 규약 (정책 확정분만)

- **규약 1**: 모든 임상 데이터는 `tenant_id` 기반으로 완전 격리되며, 어떤 경우에도 교차 접근은 불가능하다.
- **규약 2**: S-T-D 3축 중 최소 2개 이상의 축에 데이터가 입력되어야 유효한 진료 기록으로 간주한다.
- **규약 3**: 모든 데이터 변경(생성/수정/삭제)은 Audit Trail에 기록되며, 이는 변경 불가능한 로그로 보존된다.
- **규약 4**: 자유 텍스트는 구조화 데이터의 보충 설명으로만 사용되며, 핵심 임상 정보는 반드시 구조화 필드에 입력한다.
- **규약 5**: ICD 코드는 입력된 S-T-D 데이터를 기반으로 추론/제안되는 것이 원칙이며, 직접 입력은 보조 수단이다.

## ⚡ 기능 명세 (Fxxx)

| ID | 기능명 | 설명(이벤트-상태-명령/tenant_id/S·T·D 영향) | 관련 페이지 |
|----|--------|---------------------------------------------|-------------|
| F001 | 테넌트 인증 | JWT 기반 로그인 → 테넌트 컨텍스트 설정 / tenant_id 결정 / - | 로그인 |
| F002 | 환자 검색 | 이름/차트번호 검색 → 환자 목록 조회 / tenant_id 필터 / - | 환자 목록 |
| F003 | 환자 등록 | 신규 환자 정보 입력 → 환자 레코드 생성 / tenant_id 할당 / - | 환자 등록 |
| F004 | Encounter 생성 | 진료 시작 → 새 진료 세션 오픈 / tenant_id 상속 / S-T-D 초기화 | 진료 차트 |
| F005 | Situation 입력 | 증상/부위/onset 입력 → S축 데이터 저장 / tenant_id 격리 / S축 영향 | 진료 차트 |
| F006 | Treatment 입력 | 치료 계획/처방 입력 → T축 데이터 저장 / tenant_id 격리 / T축 영향 | 진료 차트 |
| F007 | Duration 입력 | 경과/재진 일정 입력 → D축 데이터 저장 / tenant_id 격리 / D축 영향 | 진료 차트 |
| F008 | ICD 자동 제안 | S-T 데이터 분석 → ICD 코드 후보 추출 / tenant_id 격리 / - | 진료 차트 |
| F009 | ICD 수동 검색 | 키워드 검색 → ICD 코드 목록 조회 / 전역 데이터 / - | ICD 검색 |
| F010 | ICD 코드 확정 | 제안/검색 결과 선택 → Encounter에 매핑 / tenant_id 격리 / - | 진료 차트 |
| F011 | 유사 환자 조회 | S-T-D 패턴 매칭 → 과거 케이스 조회 / tenant_id 필터 / - | 유사 사례 |
| F012 | 진료 기록 저장 | 차트 확정 → 데이터 영속화 + Audit 생성 / tenant_id 격리 / S-T-D 확정 | 진료 차트 |
| F013 | 진료 기록 수정 | 기존 차트 편집 → 변경사항 저장 + History 기록 / tenant_id 격리 / S-T-D 변경 | 진료 차트 |
| F014 | 사용자 관리 | 의료진 계정 CRUD → 권한 할당 / tenant_id 격리 / - | 사용자 관리 |
| F015 | 마스터 데이터 관리 | ICD/Anatomy/선택지 설정 → 테넌트별 커스터마이징 / tenant_id 격리 / - | 마스터 설정 |
| F016 | 진료 통계 조회 | 기간/의사/상병별 집계 → 대시보드 표시 / tenant_id 필터 / - | 통계 대시보드 |
| F017 | Audit Trail 조회 | 변경 이력 검색 → 로그 목록 표시 / tenant_id 필터 / - | 감사 로그 |
| F018 | 데이터 품질 검토 | 미완성/미매핑 차트 탐지 → 알림 생성 / tenant_id 필터 / - | 품질 모니터링 |
| F019 | 신체검사 기록 | 이학적 검사 소견 입력 → S축 객관적 데이터 저장 / tenant_id 격리 / S축 영향 | 신체검사 |
| F020 | ROM 측정 | 관절 가동범위 측정값 입력 → 정량 데이터 저장 / tenant_id 격리 / S축 영향 | 신체검사 |
| F021 | 근력 평가 | 근력 등급(0-5) 입력 → 평가 데이터 저장 / tenant_id 격리 / S축 영향 | 신체검사 |
| F022 | 특수검사 수행 | 특수검사 결과(양성/음성) 기록 → 검사 데이터 저장 / tenant_id 격리 / S축 영향 | 신체검사 |
| F023 | 압통점 마킹 | 해부학적 위치에 압통점 표시 → 위치 데이터 저장 / tenant_id 격리 / S축 영향 | 신체검사 |
| F024 | 영상검사 오더 | 검사 처방 생성 → 오더 데이터 저장 / tenant_id 격리 / - | 영상검사 |
| F025 | 영상 판독 입력 | 판독 소견 구조화 입력 → S축 진단 근거 저장 / tenant_id 격리 / S축 영향 | 영상검사 |
| F026 | 영상 시계열 비교 | 이전/현재 영상 비교 → 변화 추이 기록 / tenant_id 격리 / S축 영향 | 영상검사 |
| F027 | DICOM 메타데이터 연동 | 영상 메타정보 자동 수집 → 검사 정보 보강 / tenant_id 격리 / - | 영상검사 |
| F028 | 검사-ICD 매핑 | 검사 소견 기반 ICD 추천 → 진단 코드 제안 / tenant_id 격리 / - | 영상검사 |
| F029 | 검사 결과 통합 뷰 | 모든 검사 결과 종합 → 통합 진단 근거 표시 / tenant_id 격리 / S축 영향 | 검사 통합 |
| F030 | 검사 이상소견 하이라이트 | 비정상 소견 자동 강조 → 주의 항목 표시 / tenant_id 격리 / - | 검사 통합 |

## 🧭 메뉴 구조

### [Physician Console]
- **환자 관리**
  - 환자 검색 (F002)
  - 환자 등록 (F003)
- **진료**
  - 진료 시작 (F004)
  - S-T-D 차팅 (F005, F006, F007)
  - 신체검사 (F019, F020, F021, F022, F023)
  - 영상검사 (F024, F025, F026, F027)
  - 검사 결과 통합 (F029, F030)
  - ICD 코드 매핑 (F008, F009, F010, F028)
  - 진료 기록 저장 (F012)
  - 진료 기록 수정 (F013)
- **임상 도구**
  - 유사 환자 조회 (F011)
  - ICD 검색 (F009)

### [Admin Console]
- **사용자 관리**
  - 의료진 계정 관리 (F014)
  - 권한 설정 (F014)
- **시스템 설정**
  - 마스터 데이터 관리 (F015)
  - 테넌트 설정 (F001)
- **모니터링**
  - 진료 통계 (F016)
  - 데이터 품질 검토 (F018)
  - 감사 로그 (F017)

## 📄 페이지별 상세 기능

### [로그인 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 사용자 인증 및 테넌트 컨텍스트 설정 |
| 진입 경로 | 최초 접속 또는 세션 만료 시 자동 리다이렉트 |
| 사용자 행동 | ID/PW 입력 → 로그인 버튼 클릭 → JWT 토큰 발급 → 메인 화면 이동 |
| 주요 기능 | 테넌트별 로그인, JWT 토큰 발급, 자동 로그인 옵션 |
| 구현 기능 ID | F001 |

### [환자 목록 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 환자 검색 및 선택을 위한 인터페이스 |
| 진입 경로 | Physician Console > 환자 관리 > 환자 검색 |
| 사용자 행동 | 검색 조건 입력 → 검색 실행 → 환자 선택 → 진료 시작 |
| 주요 기능 | 이름/차트번호 검색, 최근 진료 환자 표시, 환자 상세 정보 미리보기 |
| 구현 기능 ID | F002 |

### [환자 등록 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 신규 환자 정보 입력 및 등록 |
| 진입 경로 | 환자 목록 > 신규 환자 등록 버튼 |
| 사용자 행동 | 기본 정보 입력 → 의료 정보 입력 → 유효성 검사 → 저장 |
| 주요 기능 | 환자 기본정보 입력, 중복 검사, 차트번호 자동 생성 |
| 구현 기능 ID | F003 |

### [진료 차트 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | S-T-D 기반 구조화된 진료 기록 작성 |
| 진입 경로 | 환자 선택 > 진료 시작 또는 기존 진료 기록 편집 |
| 사용자 행동 | Encounter 생성 → S/T/D 탭 전환하며 입력 → ICD 매핑 → 저장 |
| 주요 기능 | S-T-D 3축 입력 폼, ICD 자동 제안, 실시간 저장, 입력 완성도 표시 |
| 구현 기능 ID | F004, F005, F006, F007, F008, F010, F012, F013 |

### [ICD 검색 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | ICD-10/11 코드 수동 검색 및 선택 |
| 진입 경로 | 진료 차트 > ICD 수동 검색 버튼 |
| 사용자 행동 | 키워드 입력 → 검색 실행 → 결과 목록 확인 → 코드 선택 |
| 주요 기능 | 한글/영문 검색, ICD-10/11 전환, 즐겨찾기 기능 |
| 구현 기능 ID | F009 |

### [유사 사례 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 현재 환자와 유사한 과거 케이스 조회 |
| 진입 경로 | 진료 차트 > 유사 사례 보기 |
| 사용자 행동 | S-T-D 패턴 자동 분석 → 유사도 기반 정렬 → 상세 내용 확인 |
| 주요 기능 | 유사도 점수 표시, 치료 결과 비교, 익명화된 케이스 표시 |
| 구현 기능 ID | F011 |

### [사용자 관리 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 의료진 계정 생성 및 권한 관리 |
| 진입 경로 | Admin Console > 사용자 관리 |
| 사용자 행동 | 사용자 추가 → 정보 입력 → 권한 설정 → 저장 |
| 주요 기능 | 계정 CRUD, 역할 기반 권한 설정, 비밀번호 초기화 |
| 구현 기능 ID | F014 |

### [마스터 설정 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 테넌트별 마스터 데이터 커스터마이징 |
| 진입 경로 | Admin Console > 시스템 설정 > 마스터 데이터 관리 |
| 사용자 행동 | 카테고리 선택 → 항목 추가/수정 → 적용 |
| 주요 기능 | S-T-D 각 축별 선택지 관리, 자주 사용하는 ICD 코드 설정 |
| 구현 기능 ID | F015 |

### [통계 대시보드 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 진료 현황 및 통계 시각화 |
| 진입 경로 | Admin Console > 모니터링 > 진료 통계 |
| 사용자 행동 | 기간 설정 → 필터 적용 → 차트/그래프 확인 → 상세 데이터 드릴다운 |
| 주요 기능 | 상병별/의사별 통계, 추세 분석, 엑셀 내보내기 |
| 구현 기능 ID | F016 |

### [감사 로그 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 모든 데이터 변경 이력 추적 |
| 진입 경로 | Admin Console > 모니터링 > 감사 로그 |
| 사용자 행동 | 검색 조건 설정 → 조회 → 상세 내역 확인 |
| 주요 기능 | 사용자별/기간별 필터, 변경 전후 비교, 복원 불가 표시 |
| 구현 기능 ID | F017 |

### [품질 모니터링 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 데이터 완성도 및 품질 관리 |
| 진입 경로 | Admin Console > 모니터링 > 데이터 품질 검토 |
| 사용자 행동 | 품질 지표 확인 → 미완성 차트 목록 조회 → 담당 의사 알림 |
| 주요 기능 | 미완성 차트 탐지, ICD 미매핑 건 추적, 품질 점수 계산 |
| 구현 기능 ID | F018 |

### [신체검사 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 이학적 검사 소견의 구조화된 기록 |
| 진입 경로 | 진료 차트 > 신체검사 탭 |
| 사용자 행동 | 검사 항목 선택 → 측정값/소견 입력 → 정상/비정상 분류 → 저장 |
| 주요 기능 | ROM 측정값 입력, 근력 등급 선택, 특수검사 결과 기록, 압통점 해부학적 마킹, 검사 템플릿 |
| 구현 기능 ID | F019, F020, F021, F022, F023 |

### [영상검사 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 영상 검사 오더 및 판독 소견 관리 |
| 진입 경로 | 진료 차트 > 영상검사 탭 |
| 사용자 행동 | 검사 종류 선택 → 오더 생성 → 판독 소견 입력 → ICD 연계 |
| 주요 기능 | 검사 오더 생성, 판독 소견 구조화 입력, 시계열 비교 뷰, DICOM 메타데이터 표시, 영상 소견 기반 ICD 추천 |
| 구현 기능 ID | F024, F025, F026, F027, F028 |

### [검사 통합 뷰 페이지]
| 항목 | 내용 |
|---|---|
| 역할 | 모든 검사 결과의 종합적 조회 및 분석 |
| 진입 경로 | 진료 차트 > 검사 결과 통합 보기 |
| 사용자 행동 | 검사 종류별 필터 → 시계열 정렬 → 이상 소견 확인 → 진단 근거 도출 |
| 주요 기능 | 신체검사/영상검사/혈액검사 통합 표시, 이상 소견 자동 하이라이트, 검사 결과 기반 진단 추론 지원, 검사 히스토리 타임라인 |
| 구현 기능 ID | F029, F030 |

## 🗄️ 데이터 모델 (요구 수준)

### Core Entities

```
tenant
- id: bigint (PK)
- name: varchar(100)
- code: varchar(50) unique
- status: enum('active', 'inactive')
- created_at: timestamp
- updated_at: timestamp

user
- id: bigint (PK)
- tenant_id: bigint (FK)
- username: varchar(50)
- password_hash: varchar(255)
- role: enum('physician', 'admin', 'nurse')
- name: varchar(100)
- license_number: varchar(50)
- status: enum('active', 'inactive')
- created_at: timestamp
- updated_at: timestamp

patient
- id: bigint (PK)
- tenant_id: bigint (FK)
- chart_number: varchar(50)
- name: varchar(100)
- birth_date: date
- gender: enum('male', 'female', 'other')
- phone: varchar(20)
- created_at: timestamp
- updated_at: timestamp

encounter
- id: bigint (PK)
- tenant_id: bigint (FK)
- patient_id: bigint (FK)
- physician_id: bigint (FK → user)
- encounter_date: timestamp
- status: enum('in_progress', 'completed', 'cancelled')
- created_at: timestamp
- updated_at: timestamp
```

### S-T-D Axis Entities

```
situation_record
- id: bigint (PK)
- tenant_id: bigint (FK)
- encounter_id: bigint (FK)
- symptom_code: varchar(50)
- symptom_text: text
- anatomy_code: varchar(50)
- anatomy_text: varchar(255)
- onset_date: date
- severity: enum('mild', 'moderate', 'severe', 'critical')
- additional_notes: text
- created_at: timestamp
- updated_at: timestamp

treatment_record
- id: bigint (PK)
- tenant_id: bigint (FK)
- encounter_id: bigint (FK)
- treatment_type: enum('medication', 'procedure', 'therapy', 'other')
- treatment_code: varchar(50)
- treatment_name: varchar(255)
- dosage: varchar(100)
- frequency: varchar(100)
- start_date: date
- additional_notes: text
- created_at: timestamp
- updated_at: timestamp

duration_record
- id: bigint (PK)
- tenant_id: bigint (FK)
- encounter_id: bigint (FK)
- expected_duration_days: int
- follow_up_date: date
- prognosis: enum('excellent', 'good', 'fair', 'poor', 'guarded')
- recovery_milestones: text
- additional_notes: text
- created_at: timestamp
- updated_at: timestamp
```

### Examination Entities

```
physical_examination_record
- id: bigint (PK)
- tenant_id: bigint (FK)
- encounter_id: bigint (FK)
- examination_type: varchar(100)
- examination_date: timestamp
- examiner_id: bigint (FK → user)
- status: enum('normal', 'abnormal', 'borderline')
- created_at: timestamp
- updated_at: timestamp

rom_measurement
- id: bigint (PK)
- tenant_id: bigint (FK)
- physical_examination_id: bigint (FK)
- joint_name: varchar(100)
- movement_type: varchar(100)
- measurement_value: decimal(5,2)
- normal_range_min: decimal(5,2)
- normal_range_max: decimal(5,2)
- laterality: enum('left', 'right', 'bilateral')
- created_at: timestamp

muscle_strength_test
- id: bigint (PK)
- tenant_id: bigint (FK)
- physical_examination_id: bigint (FK)
- muscle_group: varchar(100)
- strength_grade: int (0-5)
- laterality: enum('left', 'right', 'bilateral')
- notes: text
- created_at: timestamp

special_test_result
- id: bigint (PK)
- tenant_id: bigint (FK)
- physical_examination_id: bigint (FK)
- test_name: varchar(200)
- test_result: enum('positive', 'negative', 'equivocal')
- laterality: enum('left', 'right', 'bilateral', 'na')
- clinical_significance: text
- created_at: timestamp

tender_point
- id: bigint (PK)
- tenant_id: bigint (FK)
- physical_examination_id: bigint (FK)
- anatomy_code: varchar(50)
- anatomy_name: varchar(200)
- tenderness_grade: enum('mild', 'moderate', 'severe')
- laterality: enum('left', 'right', 'bilateral')
- notes: text
- created_at: timestamp

imaging_order
- id: bigint (PK)
- tenant_id: bigint (FK)
- encounter_id: bigint (FK)
- order_date: timestamp
- modality: enum('xray', 'ct', 'mri', 'ultrasound', 'pet', 'bone_scan')
- body_part: varchar(100)
- urgency: enum('routine', 'urgent', 'stat')
- clinical_indication: text
- ordering_physician_id: bigint (FK → user)
- status: enum('ordered', 'scheduled', 'completed', 'cancelled')
- created_at: timestamp
- updated_at: timestamp

imaging_result
- id: bigint (PK)
- tenant_id: bigint (FK)
- imaging_order_id: bigint (FK)
- examination_date: timestamp
- radiologist_id: bigint (FK → user)
- findings: text
- impression: text
- comparison_study_id: bigint (FK → self)
- dicom_study_uid: varchar(128)
- created_at: timestamp
- updated_at: timestamp

imaging_finding_detail
- id: bigint (PK)
- tenant_id: bigint (FK)
- imaging_result_id: bigint (FK)
- finding_type: varchar(100)
- anatomy_code: varchar(50)
- severity: enum('normal', 'mild', 'moderate', 'severe', 'critical')
- measurement_value: varchar(100)
- measurement_unit: varchar(20)
- change_from_prior: enum('new', 'improved', 'stable', 'worsened', 'resolved')
- created_at: timestamp
```

### Knowledge & Mapping Entities

```
encounter_icd_mapping
- id: bigint (PK)
- tenant_id: bigint (FK)
- encounter_id: bigint (FK)
- icd_code: varchar(20)
- icd_version: enum('ICD10', 'ICD11')
- mapping_type: enum('auto_suggested', 'manual_selected', 'verified')
- confidence_score: decimal(3,2)
- created_by: bigint (FK → user)
- created_at: timestamp

icd_master
- id: bigint (PK)
- code: varchar(20)
- version: enum('ICD10', 'ICD11')
- name_ko: varchar(500)
- name_en: varchar(500)
- category: varchar(100)
- parent_code: varchar(20)
- is_active: boolean
- created_at: timestamp
- updated_at: timestamp
```

### Audit & History

```
audit_log
- id: bigint (PK)
- tenant_id: bigint (FK)
- user_id: bigint (FK)
- entity_type: varchar(50)
- entity_id: bigint
- action: enum('create', 'update', 'delete', 'view')
- old_value: json
- new_value: json
- ip_address: varchar(45)
- user_agent: text
- created_at: timestamp

data_history
- id: bigint (PK)
- tenant_id: bigint (FK)
- table_name: varchar(50)
- record_id: bigint
- version: int
- change_type: enum('insert', 'update', 'delete')
- changed_data: json
- changed_by: bigint (FK → user)
- changed_at: timestamp
```

## 🧩 Policy Decision Request (정책 결정 요청)

> 아래 항목은 prd-writer가 결론내리지 않으며, clinical-policy-designer의 결정을 기다린다.

| 임시ID | 관련 기능(Fxxx) | 쟁점 | 필요한 결정(허용/금지/조건부) |
|---|---|---|---|
| P-REQ-001 | F013 | 진료 기록 수정 허용 범위 | 완료된 진료의 수정 허용 조건 및 시간 제한 |
| P-REQ-002 | F013 | 진료 기록 삭제 정책 | 삭제 가능 여부 및 Soft Delete vs Hard Delete |
| P-REQ-003 | F008 | ICD 자동 제안 알고리즘 | S-T 데이터 기반 매핑 가중치 및 우선순위 규칙 |
| P-REQ-004 | F008 | ICD-10 ↔ ICD-11 전환 | 버전 간 매핑 충돌 시 우선순위 |
| P-REQ-005 | F011 | 유사 환자 매칭 기준 | 유사도 계산에 사용할 피처 및 가중치 |
| P-REQ-006 | F011 | 환자 정보 비식별화 수준 | 유사 사례 표시 시 개인정보 마스킹 범위 |
| P-REQ-007 | F005, F006, F007 | S-T-D 필수 입력 규칙 | 각 축별 최소 입력 요구사항 |
| P-REQ-008 | F012 | 진료 완료 조건 | 차트 확정을 위한 최소 데이터 완성도 |
| P-REQ-009 | F016 | 통계 데이터 접근 권한 | 의사별 통계 열람 권한 범위 |
| P-REQ-010 | F017 | Audit 로그 보존 기간 | 법적 요구사항에 따른 최소 보존 기간 |
| P-REQ-011 | F002, F003 | 환자 중복 판단 기준 | 동명이인 구분을 위한 식별 규칙 |
| P-REQ-012 | F014 | 사용자 권한 체계 | 역할별 상세 권한 매트릭스 |
| P-REQ-013 | F019, F020, F021 | 신체검사 정상 범위 기준 | 연령/성별별 정상치 설정 방식 |
| P-REQ-014 | F022 | 특수검사 양성 판정 기준 | 검사별 양성/음성 판독 가이드라인 |
| P-REQ-015 | F023 | 압통점 중증도 분류 | 경도/중등도/중증 판정 기준 |
| P-REQ-016 | F025 | 영상 판독 자동화 수준 | AI 판독 지원 범위 및 최종 확정 권한 |
| P-REQ-017 | F026 | 영상 비교 시점 기준 | 비교 대상 영상 선정 규칙 (최근/동일검사/임상적 의미) |
| P-REQ-018 | F027 | DICOM 데이터 보존 정책 | 메타데이터만 vs 영상 파일 포함 저장 |
| P-REQ-019 | F028 | 검사-ICD 매핑 신뢰도 | 검사 소견별 진단 코드 제안 가중치 |
| P-REQ-020 | F029, F030 | 검사 결과 공유 범위 | 타과 협진 시 검사 결과 열람 권한 |
| P-REQ-021 | F030 | 이상 소견 자동 판정 기준 | 정상/비정상 자동 분류 알고리즘 |

## 🔧 기술 스택 (Platform 고정)

- **Backend**: Spring Boot 3.x
- **Persistence**: MariaDB, JPA + Querydsl
- **Auth**: JWT (Custom)
- **Multi-tenancy**: Request-scoped TenantContext + tenant_id 기반 논리 격리
- **Knowledge Base**: ICD-10/11 Master Data + Anatomy Classification System

---

*이 PRD는 MVP 범위를 정의하며, 의학적 타당성 및 정책적 결정이 필요한 부분은 P-REQ로 분리하여 추후 확정한다.*