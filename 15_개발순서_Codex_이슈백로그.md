# 실행계획: 신규 기능보다 계약·데이터·보안 경계를 먼저

## 규칙
- 한 PR에서 하나의 계약 변화만 수행; PR당 review 가능한 변경 크기 목표(예: 200~400 LOC 중심, migration 등 예외 허용).
- 현행 확인(파일/실제 테스트) → 변경안/ADR 승인 → test-first/contract fixture → 구현 → 통합 테스트 → 문서 업데이트.
- OpenAPI/schema/state diagrams가 변경되면 영향받는 저장소별로 서로 연결된 PR에서 Java DTO/Pydantic/TS 타입/SQL migration/테스트를 조율하고, 호환성 및 배포 순서를 명시한다.
- 저장소 전체를 자동 재작성하지 않는다. 데이터 손실 위험 SQL은 사람 승인.

## Stepwise backlog
| ID | 우선 | 선행 | 소유 | Scope(예상) | 인수 기준 |
|---|---|---|---|---|---|
| TD-00 | P0 | - | Backend+Data | 전체 인벤토리, `./gradlew test`, `pytest`, migration 확인 | 기준 커밋 SHA/테스트 결과/목록 기록 |
| TD-01 | P0 | TD-00 | Backend+Data | Track1 OpenAPI v1 계약 고정 | Spring/Python 성공/422/500 fixture 통과 |
| TD-02 | P0 | TD-01 | Data | `/clusters/analyze` v1 + DTO 검증 | nullable/64hex/requestId/members 통과 |
| TD-03 | P0 | TD-00 | Data | Neo4j schema Reservation/Event/Token; null | 2회 적재 멱등/입력 ticket ID 보존 |
| TD-04 | P0 | TD-03 | Data | tenant 범위/배치 projection; GDS 설치 | 전체 DB 분석 금지, DB/서비스 격리 시험 |
| TD-05 | P0 | TD-02,03,04 | Data | WCC baseline/Louvain 실행/score explanation | LOUVAIN 문자열·알고리즘 일치, version |
| TD-06 | P0 | TD-01,02,05 | Backend | FastApiAdapter 응답 검증/실패 mapping | 0 clusters vs unavailable 분리, 3s/5s timeout 테스트 |
| TD-07 | P0 | TD-00 | Backend | MySQL 현재 모델 diff, Flyway baseline | 기존 데이터 보존+ddl-auto validate |
| TD-08 | P0 | TD-07 | Backend | job/run 상태+lease+idempotency | worker kill/retry no duplicate, 202 API |
| TD-09 | P0 | TD-07 | Backend | tenant auth/membership RBAC | BOLA/BFLA suite 통과, 단일 demo 제한 |
| TD-10 | P1 | TD-07,09 | Backend | evidence/image private storage+digest | URL 접근권한/MIME/size 테스트 |
| TD-11 | P1 | TD-10 | Data | VLM response revision+UNREADABLE, 오류분리 | masked seat no guesses, 로그 redaction |
| TD-12 | P1 | TD-11,09 | Backend | 매치 후보 조회 및 권한 | UNVERIFIABLE data absent/forbidden 분리 |
| TD-13 | P1 | TD-07,10,12 | Backend | Case evidence & decision state | cross track link without auto verdict |
| TD-14 | P1 | TD-13 | Backend | PDF immutable version 승인+발행 | SHA/actor/snapshot/race/권한 검증 |
| TD-15 | P1 | TD-08,09 | Front | React IA/공통 에러/잡목록 | generated API 타입, 202/FAILED screen |
| TD-16 | P1 | TD-13,15 | Front | Graph review + image split + case UI | 권한/상태가드/E2E |
| TD-17 | P1 | TD-14,16 | Front | PDF center | approve/failed/retry/download audit flow |
| TD-18 | P1 | TD-05,11,17 | QA/PM | 그래프·OCR/VLM·업무시간 실험 | 재현 seed, 통계표, 실험 조건 |
| TD-19 | P2 | TD-18 | Infra | 분리 worker/큐/GDS scale | 실제 병목 입증 후 ADR 승인 |


## 권장 병렬 개발
- 계약/Graph 준비 전 프론트는 **Figma 및 MSW mock 기반 공통 컴포넌트**까지만 병행한다.
- Backend: TD-00→TD-01→TD-06, TD-07→TD-08→TD-09.
- Data: TD-00→TD-02→TD-03→TD-04→TD-05.
- Frontend: IA+design tokens/mock handlers→TD-15→TD-16→TD-17.
- QA: 계약 fixture와 권한/장애 케이스를 먼저 만들고 각 PR에서 실행한다.

## Codex 공통 작업 템플릿
```
목표: TD-XX (1개 이슈만)
현행 근거: 확인한 파일 및 메서드, 테스트, OpenAPI 버전
수정 범위: 허용된 경로 목록
변경 금지: 관련 없는 클래스/DDL/기존 public contract
기능 규칙: [input → validation → state → output → exception]
보안: tenant/object-level auth, No raw PII, No raw image logging
테스트: happy path + invalid + timeout + concurrency + idempotency
보고: 실제 변경 파일, diff, test commands/results, 남은 위험, ADR 필요한 사항
코드 변경 전에 계획 및 영향 범위를 먼저 출력하고 승인 대기.
```

## PR 체크리스트
- [ ] 변경의 근거/범위/기능 ID 포함
- [ ] 신규/변경 OpenAPI와 샘플 JSON, 양 언어 test fixture
- [ ] tenant ID trust boundary 및 객체 접근 검증
- [ ] nullable vs unreadable vs missing 구분
- [ ] 기술 작업 상태와 사건 상태 분리
- [ ] migration upgrade/rollback·백업 영향
- [ ] 성공/실패/중복/타임아웃/동시성 테스트
- [ ] API error code, frontend 빈 상태/에러 상태
- [ ] 분석/모델 버전과 증거 출처/로그 redaction
- [ ] PR reviewer가 배포/실험 재현할 명령 기록

## 추가 작업 제안 (기존 TD-01~TD-19 유지)

아래 항목은 모두 **PROPOSED**이며 GitHub Issue 생성이나 범위 승인 완료를 뜻하지 않는다. 세부 선행 조건과 계약 결정은 [결정 대기 목록](governance/decision-log.md)에서 추적한다.

| ID | 상태 | 목적 | 기존 TD와의 관계 |
|---|---|---|---|
| TD-00D | PROPOSED | Specs Git·문서·계약 관리 체계 | TD-01 계약 고정 전 |
| TD-BUG-01 | PROPOSED | 기존 매크로 감사 PDF의 `mappingScore` null 경로 수정 | TD-14 이전 결함 수정 |
| TD-SEC-01 | PROPOSED | 인증·인가와 tenant 경계의 선행 안전 조치 | TD-09 및 TD-04 이전 검토 |
| TD-SEC-02 | PROPOSED | 민감 로그·PDF 외부 리소스·모델 전송 정책 점검 | TD-10·TD-11·TD-14 이전 검토 |
| TD-DATA-01 | PROPOSED | 데이터 처리 스크립트의 삭제·중복 적재 위험 분리 | TD-03·TD-04 이전 검토 |
| TD-QA-01 | PROPOSED | FastAPI 단위·계약 테스트 기준선 마련 | TD-01·TD-02 검증 지원 |
| TD-INGEST-01 | PROPOSED | 입력 식별자·검증·불변 배치 수집 설계 | TD-03·TD-07·TD-08 연계 |
| TD-INT-01 | PROPOSED | Track 1·Track 2 결과 통합 및 실패 상태 계약 | TD-06·TD-11·TD-13 연계 |
| TD-AUD-01 | PROPOSED | 검토·승인·PDF 발급의 감사 이력 | TD-13·TD-14 연계 |
