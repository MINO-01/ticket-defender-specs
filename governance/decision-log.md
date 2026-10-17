# 미결정 사항

모든 항목은 `PENDING`이다. 아래 대안은 비교 대상이며 승인·구현 결정이 아니다. 결정 시 근거, 승인자, 일시, 관련 TD와 계약 revision을 기록한다.

| ID | 주제 | 검토할 대안 | 현재 상태 | 영향 TD |
|---|---|---|---|---|
| DEC-01 | Track 1 내부 서비스 인증 | mTLS / 서비스 토큰·서명 / 내부망+게이트웨이 인증 | PENDING | TD-01, TD-SEC-01, TD-02 |
| DEC-02 | tenant 및 event scope 전달 | 인증된 Spring 컨텍스트의 서명된 scope / 작업별 별도 식별자 / tenant별 엔드포인트 | PENDING | TD-01, TD-03, TD-04, TD-SEC-01 |
| DEC-03 | Python GDS·Neo4j 호환성 | 검증된 GDS 플러그인 버전 고정 / 호환 조합 변경 / 알고리즘 fallback | PENDING | TD-04, TD-05 |
| DEC-04 | MySQL 기존 데이터 마이그레이션 | 증분 확장·backfill / 별도 이행 테이블과 단계 전환 | PENDING | TD-07, TD-INGEST-01 |
| DEC-05 | 분석 위험점수 의미·버전 | 규칙 기반 점수 / 보정된 모델 점수 / 별도 신뢰도·근거 필드 | PENDING | TD-05, TD-06, TD-13 |
| DEC-06 | Track 2 UNKNOWN/AMBIGUOUS 상태 | 두 상태 구분 / 하나의 표준 상태로 통합 후 이유코드 분리 | PENDING | TD-11, TD-12, TD-INT-01 |
| DEC-07 | 외부 Gemini 전송 정책 | 고객 동의·계약 후 제한 전송 / 비식별화 후 전송 / 로컬 OCR·VLM 우선 | PENDING | TD-SEC-02, TD-10, TD-11 |
| DEC-08 | 관리자 OpenAPI 확장 범위 | MVP 필수 API만 선승인 / 전체 목표 API 동시 승인 | PENDING | TD-01, TD-08, TD-15, TD-16 |
| DEC-09 | ReportVersion 승인·PDF 발급 | 승인 후 불변 PDF 발급 / 초안 PDF와 최종본 별도 발급 | PENDING | TD-14, TD-AUD-01, TD-BUG-01 |
| DEC-10 | Neo4j 고객사 격리 구조 | 공용 DB의 강제 tenant 범위 / 고객사별 DB 분리 | PENDING | TD-03, TD-04, TD-SEC-01 |
| DEC-11 | Job FAILED_RESOURCE 의미 | 재시도 가능한 일시 자원 부족 / 별도 영구 실패 상태와 구분 | PENDING | TD-08, TD-06 |
| DEC-12 | CLOSED 사건 재개 정책 | 권한 있는 재개·이력 보존 / 재개 금지 후 후속 사건 생성 | PENDING | TD-13, TD-AUD-01 |
| DEC-13 | Case·Approval 기본 역할과 예외 관리자 개입 | 기존 Specs의 TENANT_ADMIN 상태/초안·APPROVER 초안 허용 / Backend MVP의 REVIEWER 상태·초안·요청, APPROVER 독립 결정 및 별도 관리자 예외 절차 | PENDING | TD-09, TD-13, TD-14, TD-AUD-01 |
| DEC-14 | 공식 PDF 발급 활성화·발급 보류 운영 | 별도 발급 역할·차단 해제·IssuanceHold 등록/해제 공개 API·SLA 승인 / 내부 서버 업무 명령 유지 | PENDING | TD-14, TD-SEC-01, TD-AUD-01 |
| DEC-15 | ReviewReceipt 운영 기한·토큰 보존 | Backend 권장 15분 / 운영 설정 및 저장·폐기·감사 기간 결정 | PENDING | TD-14, TD-SEC-01 |

## Backend PR #21 MVP 방향과 공유 승인 경계

[Backend PR #21의 지정 커밋](https://github.com/MINO-01/ticket-defender-backend/pull/21)은 사용자 검토 뒤 선택한 MVP 방향을 **Backend 제안 문서**에 반영했다. DEC-09에는 불변 EvidenceSnapshot·ReportVersion 내용에 묶인 단일 ApprovalDecision, 신규 공식 PDF 공개 성공 후에만 이전 GENERATED를 SUPERSEDED 처리, `ISSUANCE_ABANDONED` 및 증거 V2 재검토 뒤 `NO_NEW_REPORT_AFTER_STALE` 종료가 해당한다. 후자는 사전 확정 `EVIDENCE_REVIEW_COMPLETED` CaseActivity, 당시 REVIEWER 권한·현재 snapshot ID/hash/revision·증거 변경으로 STALE인 이전 Decision, 별도 미발급 사유와 과거 PDF 이력 확인을 전제로 한다. DEC-12에는 DISMISSED/CLOSED 직접 REOPEN 대신 원 Case를 참조하는 새 OPEN 후속 Case가 해당한다. 이 선택은 공유 Specs의 승인자·시각·근거 PR을 갖춘 `APPROVED` 기록이 아니므로 두 항목은 `PENDING`으로 유지한다.

DEC-08은 Backend의 17개 Case·Approval 경로를 공유 관리자 OpenAPI 초안에 포함하는 범위에 영향을 받는다. 이번 동기화는 제안 계약의 중복 경로를 맞춘 것이며 API 확장 승인이나 운영 활성화가 아니다. DEC-13은 기존 Specs 역할표와 Backend의 제한적 MVP 권한이 충돌해 별도 승인 필요하다. DEC-14/15 및 개인정보 보유·파기·legal hold도 후속 결정이다. 구 `/report-versions/{id}/approve` 제안 경로에서 단일 `/approval-requests/{id}/decision` 경로로 옮길 때는 실제 소비자 DTO·fixture, 구버전 공존 또는 명시적 오류, 제공자/소비자 적용 순서와 복귀 방식, 연결된 저장소별 PR을 승인 전 기록한다. 현재 두 경로 모두 Case·Approval 구현 근거가 없어 실제 서비스 호환성을 검증했다고 표시하지 않는다.
