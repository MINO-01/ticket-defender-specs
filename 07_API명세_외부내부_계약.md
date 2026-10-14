# 07. REST API/DTO/오류 규약 — v1 제안

> Spring의 FastAPI 호출부가 **소비자로서 기대하는 내부 경로**는 `POST /api/v1/clusters/analyze`다. Python이 현재 제공하는 `/api/v1/clusters/macro` 및 WCC 결과와는 일치하지 않는다. 목표 내부 계약은 [Track 1 proposal](contracts/track1-internal-v1-proposal.yaml), Spring이 React에 제공할 목표 관리자 API는 [관리자 API proposal](contracts/admin-api-v1-proposal.yaml)을 각각 참조한다. 두 계약 모두 아직 제안 상태이며 현행 동작이나 승인 완료를 뜻하지 않는다.

> Spring의 기존 외부 진입점 `/api/v1/audits/analyze`와 Spring → FastAPI 내부 호출 URL, 아래 목표 관리자 API URL은 서로 다른 인터페이스다. 계약 승인과 실제 구현·검증 상태는 [계약 레지스트리](contracts/registry.yaml)에서 별도로 추적한다.

## 관리자 API — Spring (모두 인증 필요)
| Method | Path | Request | 2xx | 역할 | 주의 |
|---|---|---|---|---|---|
| POST | /api/v1/import-batches | eventId, sourceId, tickets[], idempotency | 202 batchId/status | Admin/Analyst | 422 형식, 409 중복키 충돌 |
| GET | /api/v1/import-batches/{id} | - | 200 batch/status/errors | Admin/Analyst | tenant scope |
| POST | /api/v1/analysis-jobs | batchId, algorithm=LOUVAIN | 202 jobId,QUEUED | Admin/Analyst | 409 상태충돌 |
| GET | /api/v1/analysis-jobs/{id} | - | 200 status/attempt/errors | team read | 실패 ≠ 완료 0건 |
| POST | /api/v1/analysis-jobs/{id}/retry | reason | 202 jobId/status | Admin/Analyst | FAILED/RETRY_WAIT만 |
| GET | /api/v1/findings | eventId, pagination | 200 page/items | analysts+ | risk는 확률 아님 |
| GET | /api/v1/findings/{id} | - | 200 members/signals | analysts+ | evidence source |
| POST | /api/v1/fan-reports | multipart image/eventId/description | 201 reportId,evidenceId | Reporter/Reviewer | 413/415/422 |
| GET | /api/v1/fan-reports/{id} | - | 200 report/extraction | Reviewer/본인 제한 | raw image URL 비공개 |
| POST | /api/v1/fan-reports/{id}/extractions | mode | 202 extractionRunId | Reviewer | provider permission |
| POST | /api/v1/extractions/{id}/corrections | fields + reason, If-Match | 201 revision | Reviewer | 원본 overwrite 금지 |
| POST | /api/v1/cases | eventId,type,findingIds,evidenceIds | 201 caseId,OPEN | Analyst/Reviewer | 자동 병합 금지 |
| GET | /api/v1/cases | query/page | 200 items | member authorized | tenant scope |
| GET | /api/v1/cases/{id} | - | 200 status/evidence/permissions | member authorized | 허용 actions 제공 |
| POST | /api/v1/cases/{id}/transitions | toStatus,reason,expectedVersion | 200 status/version | Reviewer/Approver | 상태가드+409 |
| POST | /api/v1/cases/{id}/report-versions | evidence snapshot | 201 reportVersionId | Reviewer | 승인 전 DRAFT |
| POST | /api/v1/report-versions/{id}/approve | decision,reason,expectedVersion | 200 APPROVED | Approver | 자기승인 정책 결정 |
| POST | /api/v1/report-versions/{id}/render | - | 202 RENDERING | Approver | 승인 필요 |
| GET | /api/v1/report-versions/{id}/download | - | 200 signed URL/stream | allowed | audit 생성 |

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

## API 설계 일관성 규칙
모든 list는 cursor/pageSize 상한, createdAt UTC ISO8601, key는 camelCase, 단순 ID로 tenant 보안 미보장, `Idempotency-Key`와 expectedVersion/If-Match, webhook은 MVP 제외, 수정은 승인 및 증거 snapshot audit 기록. OpenAPI 계약 테스트는 Java DTO serialization, Pydantic model, backend frontend generated TS types에 동일 샘플 사용.
