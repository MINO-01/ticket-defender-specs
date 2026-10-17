# 12. 증거 관리·검토 이력·PDF 생성 명세

## 목적 / 법적 표현
보고서는 **관찰된 사실**, **모델 추출 결과**, **담당자 판단**, **미확인 사실**, **고객사 후속 요청**을 분리한 조사 지원 문서. '법적 방어막', '반박불가능', '자동 취소 근거 확정'이라는 표현은 사용하지 않는다.

## 증거 타입
| 유형 | 예시 | 필수 메타데이터 |
|---|---|---|
| ORIGINAL_IMAGE | 제보 캡처 | assetId/objectKey/SHA256/MIME/byteSize/sourceAt/uploader/retention |
| IMAGE_EXTRACTION | OCR/VLM 필드 | extractionId/modelName/version/runAt/state/value/location/manualReview |
| GRAPH_FINDING | 공유 신호 군집 | runId/algorithmVersion/graphSnapshot/clusterId/memberRefs/riskScore/featureList |
| DATABASE_MATCH | 예매 후보 검증 | queryScope/matchType/checkedAt/authorizedSource, 조회 사유 |
| REVIEW_DECISION | 담당자 의견 | actor/time/reason/from/to/action/relatedEvidence |
| EVIDENCE_REVIEW_COMPLETED | V2 검토 후 신규 미발급 판단 | 서버 저장 CaseActivity ID, REVIEWER/시각, 현재 evidenceRevision·snapshot ID/digest, NO_NEW_REPORT·사유 |
| APPROVAL_DECISION | 독립 승인/반려 | Request·ReportVersion 내용·snapshot ID/hash, 결정자, 서버 ReviewReceipt ID, 사유·시각·감사 ID |

## 증거 생성·변경 불변 규칙
1. 원본은 private object storage에 immutable version/retention control로 보관.
2. 증거 파일 업로드 시 streaming SHA256 계산 후 byteSize/MIME/제출 시간 기록.
3. 모델 결과를 원본처럼 취급하지 않음. 교정 시 새 revision과 actor/reason 기록.
4. 파일이 같은 증거 중복은 flag만 설정; 다른 제출/사건의 출처를 소거하지 않음.
5. 증거를 사건에 연결할 때 tenant/event/reservation 범위 검증.
6. 삭제·법적 보전(legal hold) 여부는 적법한 권한·기한·감사 규약으로 처리.

`EvidenceSnapshot`은 선택된 원본 object version·bytes digest, 추출/교정·대조·분석 실행/알고리즘·출처/판단 revision을 안정된 순서의 불변 manifest로 묶는다. `manifestSha256`은 그 digest 필드를 제외한 canonical JSON bytes로 계산한다. 팬 제보는 `fanReportId`, 그 제보 소속 파일은 `evidenceId`와 `fanReportId`를 함께 보존한다. 서버가 저장된 소속과 tenant/event를 확인하며 다른 제보 파일 ID를 연결하지 않는다. SHA-256은 무결성 검사이지 익명화·진정성·법적 증거력 보증이 아니다.

Case의 `evidenceRevision`은 채택한 원본·추출/교정·대조·분석 실행·출처 연결·조사 판단 등 승인 근거의 실질 변경 때 증가한다. 단순 배정·운영 메모는 `caseVersion`만 바꾼다. 보고서 본문·템플릿은 Evidence Revision과 별도인 ReportVersion 내용 `revision`, `contentSha256`, `templateVersion`으로 봉인하고, 상태 잠금 `version`도 구분한다. 과거 snapshot·ReportVersion·Decision은 덮어쓰지 않는다. READY_FOR_APPROVAL 전이와 snapshot 고정은 원자 처리하며 `caseVersionAtCapture`는 전이 후 값이다.

## 발급 보류와 승인 적격성

중요·반대 정보가 아직 근거 revision으로 정형화되지 않아도 서버는 `IssuanceHold.ACTIVE`를 사유·주체·시각·당시 revision·감사 ID와 함께 지속 저장한다. ACTIVE 보류가 하나라도 있으면 새 승인 결정·렌더 예약·공식 공개를 차단하고 미소비 검토 영수증을 폐기한다. 관련성이 확인되면 새 revision/snapshot을 고정하고 PENDING 요청을 `INVALIDATED` 또는 과거 승인 적격성을 `STALE`로 처리한 후 근거·감사와 함께 해제한다. 무관성이 확인되면 무관성 판단 근거·해제 사유·시각을 남기고 해제한다. 보류 자동 만료나 UI 플래그 해제는 허용하지 않는다. 등록·해제 공개 API 및 운영 역할·SLA는 별도 결정 대상이다.

승인은 특정 EvidenceSnapshot ID/manifest digest와 ReportVersion ID/내용 revision/digest/템플릿에만 유효하다. `ApprovalRequest`는 REVIEWER가 제출하며 Case의 현재 발급 대상 PENDING은 최대 1건, Request당 Decision은 최대 1건이다. APPROVER는 요청자와 다른 인증 사용자여야 한다. GET 검토 패키지는 조회만 한다. 명시적 POST 검토 패키지는 같은 대상·승인자·tenant/event·Case·Request에 묶인 일회용 `ReviewReceipt`를 서버에 기록하고 비밀 토큰을 한 번 반환한다. 결정은 `reviewConfirmed=true`와 확인한 ID/hash·내용 digest·토큰을 다시 검증해 영수증 소비·Decision·Request/ReportVersion/Case·감사를 원자 기록한다. 토큰 원문은 서버 감사에 넣지 않으며, 영수증은 사람의 이해를 증명하지 않는다. TTL 15분은 Backend 초기 권장값이며 확정 운영 설정이 아니다. 반려·철회·시스템 무효화 뒤 새 제출에는 새 ReportVersion 내용 revision이 필요하다.

## PDF 내용 규격
- Header: company, report ID/version, case ID, generatedAt UTC + 한국 표기, classification, approval.
- Section 1: 사건 개요/접수 경로·데이터 처리 범위.
- Section 2: 분석 결과 (Track1 graph summary; Track2 원본과 visible 필드).
- Section 3: 신호와 출처표 (가명 참조, 관찰 조건, 시간, algorithmVersion).
- Section 4: 데이터 대조 결과 (MATCH_CANDIDATE/NO_MATCH/UNVERIFIABLE + 한계).
- Section 5: 수동 검토·변경 이력·판단 이유.
- Section 6: 한계·추가 확인 필요 사항, 자동 확정 아님 고지.
- Section 7: 후속 요청/승인 담당자, 보고서 검증 정보 (reportSha256은 PDF 외부 메타/조회페이지에 보존 가능; PDF 안에 자기 자신의 hash를 넣는 자기참조 설계는 피함).

## 버전·렌더 규칙
1. REVIEWER가 현재 EvidenceSnapshot에 묶인 불변 DRAFT ReportVersion을 만들고 승인 요청을 제출한다. APPROVER 결정은 `POST /api/v1/approval-requests/{approvalRequestId}/decision` 한 경로로만 기록한다. 과거 `/report-versions/{id}/approve` 제안은 독립 승인 명령으로 사용하지 않는다.
2. 공식 PDF 발급은 Case `APPROVED_FOR_REPORT`, 현재 승인 바인딩·적격성, ACTIVE 보류 없음, 미종료 Case, 단일 렌더 예약을 확인해야 한다. 실제 발급 권한과 기존 `AuditReportService` 차단 해제는 별도 승인·구현 전제다.
3. Spring 렌더러는 봉인된 입력만 사용한다. 후보 PDF를 비공개 저장하고 checksum을 계산한 뒤 짧은 최종화 트랜잭션에서 Case·승인·revision·보류·예약을 다시 검사해 `GENERATED` 참조와 hash·감사를 확정한다. 실패 후보는 다운로드 불가이며 정리 대상이다. 서명·직인은 고객사의 명시적 사용 권한이 확인된 경우에만 사용한다.
4. 새 공식 PDF의 공개 성공과 같은 최종화에서만 이전 `GENERATED`를 `SUPERSEDED`로 바꾼다. 새 DRAFT, 승인, 현재 근거 `STALE`, 종료만으로 과거 PDF를 SUPERSEDED로 만들지 않는다. 이미 GENERATED된 파일을 덮어쓰거나 제자리 재렌더하지 않는다.
5. 기존 PDF 다운로드는 권한 있는 tenant/event·역할·파일 접근을 재검증하고 감사한 뒤 **동일 저장 파일**을 stream/짧은 URL로 제공한다. CLOSED에서도 가능하지만 새 렌더·`GENERATION_FAILED` 재시도는 불가하다. V1 파일은 V2 신규 발급 근거가 아니다.
6. 기술적 생성 실패는 `GENERATION_FAILED`로 구분한다. 동일 봉인 입력·현재 적격성으로만 제한 재시도하며, 새 증거·보고서 내용에는 새 snapshot/ReportVersion revision/Request/Decision이 필요하다. 렌더 중 보류·근거 변경·종료가 먼저 확정되면 공식 공개를 거절하고 `REPORT_PUBLICATION_BLOCKED`를 감사한다.

## 종료와 과거 보고서

`GENERATED_REPORT`는 현재 승인 보고서의 공식 PDF 공개, `ISSUANCE_ABANDONED`는 REVIEWER의 별도 포기 사유가 있는 미발급 종료, `NOT_APPLICABLE`은 근거 있는 기각이다. V1 승인 뒤 증거 V2로 과거 승인 적격성이 `STALE`이 되었고 REVIEWER가 신규 발급을 원하지 않는 경우에는 `NO_NEW_REPORT_AFTER_STALE`를 제안한다. 먼저 별도 `POST /api/v1/cases/{caseId}/evidence-review-completions`에서 현재 V2 revision·불변 snapshot ID/digest에 묶인 `EVIDENCE_REVIEW_COMPLETED` CaseActivity를 확정해야 한다. 종료 명령의 `reviewActivityId`, `reviewedEvidenceRevision`, 공통 종료 사유 및 별도 신규 미발급 사유를 서버 저장 기록과 대조하고 당시 REVIEWER 권한·tenant/event/Case·증거 변경에 따른 STALE Decision·ACTIVE 보류/PENDING 요청/렌더 예약 부재를 재확인한다. 과거 공식 PDF ID 목록은 서버가 이력에서 조회해 종료 활동·승인 감사에 연결한다. V1 결정/PDF는 불변으로 보존하고 V2 승인·발급 완료로 표시하지 않는다. 이 내용은 Backend PR #21과 동기화한 **PROPOSED·미구현** 정책이며 Specs DEC-09는 여전히 `PENDING`이다.

## PDF 생성 합격 테스트
- 필수 근거 누락 상태 승인 거부, 독립 사용자 승인·영수증 재사용 차단, 한글 폰트·페이지 넘침·표 크기 검사.
- 원본 이미지의 개인정보/바코드 안전 마스킹, 동일 입력의 보고서 snapshot 동일성, generated PDF checksum 저장.
- 취소 또는 벌금 산정이 자동 진행되지 않음. 작성 담당자와 승인 담당자 audit 확인.
- 출력한 그래프는 전체 원본 데이터가 아닌 case에 속한 가명 요약 및 설명으로 제한.
