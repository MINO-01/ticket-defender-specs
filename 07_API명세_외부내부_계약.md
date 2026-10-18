# 07. REST API/DTO/오류 규약 — v1 제안

> Spring의 FastAPI 호출부가 **소비자로서 기대하는 내부 경로**는 `POST /api/v1/clusters/analyze`다. Python이 현재 제공하는 `/api/v1/clusters/macro` 및 WCC 결과와는 일치하지 않는다. 목표 내부 계약은 [Track 1 proposal](contracts/track1-internal-v1-proposal.yaml), Spring이 React에 제공할 목표 관리자 API는 [관리자 API proposal](contracts/admin-api-v1-proposal.yaml)을 각각 참조한다. 두 계약 모두 아직 제안 상태이며 현행 동작이나 승인 완료를 뜻하지 않는다.

> Spring의 기존 외부 진입점 `/api/v1/audits/analyze`와 Spring → FastAPI 내부 호출 URL, 아래 목표 관리자 API URL은 서로 다른 인터페이스다. 계약 승인과 실제 구현·검증 상태는 [계약 레지스트리](contracts/registry.yaml)에서 별도로 추적한다.

## 관리자 API — Spring (인증·공연 범위 선행, 모두 PROPOSED)

아래 중 Case·Approval·ReportVersion 경로의 필드·상태·오류는 [Specs 관리자 OpenAPI](contracts/admin-api-v1-proposal.yaml)를 공유 검토용 단일 초안으로 사용한다. 이 부분은 [Backend PR #21의 지정 커밋](https://github.com/MINO-01/ticket-defender-backend/blob/c30f8282e8754b78961faaa13c987b520fd7bfd5/docs/api/case-approval-v1-proposal.yaml)과 겹치는 경로를 맞춘 것이다. 문서 동기화는 API 구현·승인이 아니며 기존 `/api/v1/audits/**`는 별도 현행 인터페이스다. `{id}` 축약은 아래 표에 쓰지 않는다.

| Method | Path | 핵심 요청/응답·권한 | 가드 |
|---|---|---|---|
| POST | /api/v1/analysis-jobs | `batchId,algorithm` → 202 AnalysisJob | 기존 관리자 초안; Idempotency-Key |
| GET/POST | /api/v1/analysis-jobs/{jobId}, /api/v1/analysis-jobs/{jobId}/retry | 기술 작업 조회/재시도 | 실패 ≠ 완료 0건 |
| POST | /api/v1/fan-reports | multipart `eventId,image` → 201 `reportId,evidenceId` | 제안 응답 필드명 유지. `reportId`는 제보 응답 ID, `evidenceId`는 소속 파일 ID. 원본 비공개 |
| POST | /api/v1/cases | `eventId,type,reason`; GRAPH_CLUSTER는 `findingIds`, FAN_REPORT는 `fanReportIds`, COMBINED는 둘 다; 선택한 `evidenceIds`는 제보 소속. 후속이면 `relatedCaseId,relationReason` 쌍 | ANALYST/REVIEWER; 서버 tenant/event·소유 관계, 후속 Case는 독립 OPEN |
| GET | /api/v1/cases | `eventId,status,page,pageSize` → `items,page,pageSize,hasNext` | 서버 범위와 필터 검사 |
| GET | /api/v1/cases/{caseId} | `caseVersion,evidenceRevision,allowedActions,createdAt` 포함 Case | 현재 ReportVersion/종료 근거 조회 |
| GET | /api/v1/cases/{caseId}/sources | 출처·증거 항목 및 현재 snapshot이 있으면 ID/digest | 조회는 검토 완료 활동이 아님 |
| GET | /api/v1/cases/{caseId}/activities | append-only CaseActivity 목록 | 업무 활동과 승인 감사 구분 |
| GET | /api/v1/cases/{caseId}/evidence-snapshots/{evidenceSnapshotId} | 불변 manifest 메타데이터 | 원본 bytes/object key 미노출 |
| POST | /api/v1/cases/{caseId}/evidence-review-completions | REVIEWER `expectedCaseVersion,evidenceRevision,evidenceSnapshotId,manifestSha256,reviewOutcome=NO_NEW_REPORT,reason` → 201 `EVIDENCE_REVIEW_COMPLETED` 활동 | Idempotency-Key 필수; 종료 전에 서버 확정, Case 버전 증가 |
| POST | /api/v1/cases/{caseId}/transitions | REVIEWER `toStatus,expectedCaseVersion,reason`; CLOSED는 조건부 `closureDisposition`·분기별 필드 → Case | 직접 APPROVED_FOR_REPORT·REOPEN 금지; STALE 후 신규 미발급 종료에는 사전 `reviewActivityId,reviewedEvidenceRevision,noNewReportReason` 필수 |
| PUT | /api/v1/cases/{caseId}/assignee | 담당자·사유·`expectedCaseVersion` | 배정 변경은 승인 근거 revision 증가와 별개 |
| POST | /api/v1/cases/{caseId}/report-versions | REVIEWER가 현재 snapshot의 봉인 DRAFT 생성 → ReportVersion | 내용 `revision`과 상태 lock `version` 구분 |
| GET/POST | /api/v1/cases/{caseId}/approval-requests | 현재 요청 조회 / REVIEWER가 봉인 대상 제출 | 새 Request에 ReportVersion revision·내용 digest·snapshot ID/hash 고정 |
| GET | /api/v1/approval-requests/{approvalRequestId} | Request·Decision 상세 | 범위 검사 |
| GET/POST | /api/v1/approval-requests/{approvalRequestId}/review-package | APPROVER GET은 조회만, POST는 서버 ReviewReceipt와 일회용 토큰 발급 | GET은 영수증 생성·폐기 없음; POST 응답 `no-store` |
| POST | /api/v1/approval-requests/{approvalRequestId}/decision | APPROVER 단일 결정: `decision,reason`, 확인한 snapshot/hash·보고서 digest, 영수증 토큰·`reviewConfirmed=true`, 세 자원 기대 버전 → DecisionResult | 요청자와 다른 사용자, 영수증 1회 소비·Decision·감사 원자 처리 |
| POST | /api/v1/approval-requests/{approvalRequestId}/withdraw | REVIEWER의 PENDING 철회와 세 자원 기대 버전 → WithdrawalResult | 철회/무효화·반려 후 새 ReportVersion revision 필요 |
| GET | /api/v1/report-versions/{reportVersionId} | ReportVersion 상세 | 봉인 내용과 상태 별개 |
| POST | /api/v1/report-versions/{reportVersionId}/render | 현재 승인·근거·내용 digest와 Case/Report 기대 버전 → 202 RENDERING | 발급 역할·차단 해제 미결정, `BLOCKED_POLICY_AND_SECURITY` |
| GET | /api/v1/report-versions/{reportVersionId}/download | 이미 공식 공개된 불변 PDF 동일 파일 | CLOSED에서도 인증·인가·접근 감사 후 재다운로드 |

팬 제보 접수 API 초안의 `reportId`는 응답 필드명이고, Case의 `fanReportIds`는 영속 팬 제보 UUID 참조다. 향후 두 필드가 같은 제보 UUID를 가리킬 수는 있지만 Backend 기준 커밋에는 독립적인 팬 제보 UUID 영속 모델이 없어 현재 ID 동일성이나 매핑을 보장하지 않는다. 해당 모델 구현 전에는 클라이언트가 응답 `reportId`를 Case의 `fanReportIds`에 넣어 유효한 연결이라고 가정하지 않는다. 실제 Case 연결은 서버가 인증된 tenant/event 범위와 영속 제보·증거의 소속을 검증한 뒤 수행한다. 현행 `TicketAudit.id`는 `Long`이며 UUID로 직접 변환하지 않는다. 기존 감사 데이터와 신규 제보 UUID를 연결해야 한다면 명시적인 대응 관계와 이행 절차를 별도 구현한다. 이 설명은 신규 ID 발급 방식이나 DB 스키마를 확정하지 않는다.

기존 `/api/v1/report-versions/{reportVersionId}/approve` 제안은 단일 Decision 경로로 대체하여 관리자 OpenAPI에서 제거한다. 승인 쓰기 경로 두 개를 함께 운영하지 않는다. 이전 경로는 미구현 제안이므로 코드 호환성을 검증했다고 주장하지 않으며, 소비자 fixture와 연결 PR을 기준으로 폐기 또는 명시적 오류·전환 순서를 승인한다. `GENERATED_REPORT`, `ISSUANCE_ABANDONED`, `NOT_APPLICABLE`, `NO_NEW_REPORT_AFTER_STALE`의 종료 조건은 04와 12를 따른다. 후자는 서버에 **미리 저장된** 검토 활동, 당시 REVIEWER 권한·현재 snapshot ID/hash/revision·증거 변경에 따른 STALE Decision을 검사하고 과거 PDF를 서버 이력으로 연결한다.

반입·finding·추출·교정 및 제보 상세의 기존 07 목표 경로는 관리자 OpenAPI에 아직 수록되지 않은 **후속 설계 목록**이다. 해당 경로의 DTO·권한은 이 표나 현재 구현으로 확정하지 않는다.

장기 운영 권장안은 공유 승인 후 Specs의 관리자 OpenAPI와 artifact SHA-256/Specs commit을 API 단일 기준으로 고정하고, Backend의 별도 설명용 YAML 및 Frontend 타입·fixture가 그 revision을 참조하게 하는 것이다. 연결된 각 저장소 PR에서 겹치는 경로의 필수 필드·enum·상태/오류·예제 차이를 자동 비교하고, 제공자 → 소비자 적용 순서·구버전 공존/명시적 오류·복귀 방식을 기록한다. 현재는 Backend PR #21과 Specs 모두 미승인 제안이므로 이 운영 방식 자체도 공유 승인 전 권장안이다.

## 내부 분석 API — Python v1 계약 (target)
`POST /api/v1/clusters/analyze` : `{requestId: UUID, tickets: [{accountId,reservationNo,eventId,paymentHash?,addressHash?,deviceIdHash?,ipHash?}]}`. 최소 1 티켓, 각 티켓 최소 1 토큰, 64자리 lowercase HMAC hex. tenant context는 외부 사용자 입력이 아닌 인증된 내부 서비스 신뢰 범위. 최초 v1에서는 **한 요청에 단일 tenant만**, 그래프 DB도 단일 tenant 격리 조건 검증 전엔 demo-only.

응답(성공): `{requestId,status:"COMPLETED",algorithm:"LOUVAIN",algorithmVersion,analyzedAt,clusters:[{clusterId,members:[{accountId,reservationNo,eventId}],paymentHashes,addressHashes,deviceIdHashes,ipHashes,riskScore}]}`. 모든 배열 non-null, 구성원 >=2, 점수 `[0,1]`, 입력에 없는 예매 반환 금지.

**주의:** 응답의 각 토큰은 해당 cluster 구성원의 입력에 존재하는 토큰임을 서버에서 추가 검증하는 방안을 권장한다. 기존 Spring은 형식과 멤버십은 검증하지만 토큰의 소속 여부까지 모두 확인하는지는 추가 테스트 필요.

## 내부 Track2 API (신규 target)
`POST /api/v1/parse/ticket-image`: 현행 base64 API를 임시 보존하되 내부 서비스 인증/이미지 검증, 결과 `fieldObservations`(zone/row/seat 각각 state/value/source) 및 model/version으로 v2 설계; 요청 업로드 파일이 반드시 Spring의 검증을 거치도록 한다. 5MB 문자열 제한과 실제 디코딩 바이트 제한 별도로 검증.

## 상태 코드 / 오류
| Code | 의미 | 클라이언트 대응 |
|---|---|---|
| 400 | 잘못된 입력 구조 | 수정 유도 |
| 401 | 인증 없음 | 로그인 유도 |
| 403 | 기능 권한 없음 | 액션 차단 |
| 404 | 객체 없음/타 tenant | 식별자 존재 노출 금지 |
| 409 | 상태/버전/idempotency 충돌 | 최신 상태 새로고침 |
| 413 | 크기 제한 초과 | 파일 크기 안내 |
| 415 | 지원하지 않는 MIME | 형식 안내 |
| 422 | 필드 제약 위반 | `violations` 표시 |
| 429 | 요청 제한 | retry-after 존중 |
| 502 | 하위 분석 응답 계약 위반 | 오류 기록/운영 알림 |
| 503 | 분석 서비스 일시적 사용 불가 | job 재시도 정책 |
| 504 | 내부 분석 타임아웃 | job FAILED/RETRY_WAIT |

오류 공통 `{type,title,status,code,detail,requestId,violations?}` (`application/problem+json`). 민감 정보, 원본 토큰, 원본 이미지, stacktrace 응답 금지.

Case·Approval의 409 세부 코드는 관리자 OpenAPI `x-error-catalog`를 따른다. 특히 `CASE_VERSION_CONFLICT`는 `expectedCaseVersion`, `EVIDENCE_VERSION_MISMATCH`는 현재 증거 revision, `EVIDENCE_REVIEW_NOT_COMPLETED`는 같은 Case의 다른 활동 유형, `EVIDENCE_REVIEW_MISMATCH`는 사전 검토 활동과 현재 snapshot/revision 불일치에 대응한다. 미존재·타 tenant/event/Case 활동 ID는 모두 404 `RESOURCE_NOT_FOUND`다. `ISSUANCE_HOLD_ACTIVE`, `REPORT_RENDER_IN_PROGRESS`, `REVIEW_RECEIPT_UNUSABLE`도 409이며 장애는 500/503으로 분리한다.

## API 설계 일관성 규칙
목록은 각 경로의 page/pageSize 상한, 시각은 UTC ISO8601, key는 camelCase다. 단순 ID로 tenant 보안이 보장되지 않는다. Case 명령은 `expectedCaseVersion`, Request 명령은 `expectedRequestVersion`, ReportVersion 상태 잠금은 `expectedReportVersion`을 사용하며 내용 `revision`·`evidenceRevision`과 구분한다. 멱등성 키가 필요한 경로는 OpenAPI에 명시한다. Webhook은 MVP 제외. OpenAPI 계약 테스트에는 Java DTO serialization과 Backend/Frontend generated TS 타입의 동일 fixture를 사용한다. 새 Case API의 인증·tenant membership·공연 범위·비공개 파일 권한이 구현되기 전에는 활성화하지 않는다.
