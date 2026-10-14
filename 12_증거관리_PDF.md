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

## 증거 생성·변경 불변 규칙
1. 원본은 private object storage에 immutable version/retention control로 보관.
2. 증거 파일 업로드 시 streaming SHA256 계산 후 byteSize/MIME/제출 시간 기록.
3. 모델 결과를 원본처럼 취급하지 않음. 교정 시 새 revision과 actor/reason 기록.
4. 파일이 같은 증거 중복은 flag만 설정; 다른 제출/사건의 출처를 소거하지 않음.
5. 증거를 사건에 연결할 때 tenant/event/reservation 범위 검증.
6. 삭제·법적 보전(legal hold) 여부는 적법한 권한·기한·감사 규약으로 처리.

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
1. Reviewer DRAFT → PENDING_APPROVAL 요청.
2. Approver 승인 시 immutable evidence snapshot(JSON)과 정확한 참조버전 확정.
3. Spring PDF 렌더러(기존 openhtmltopdf 사용 가능)로 템플릿 생성. 서명·직인은 고객사의 명시적 사용 권한이 확인된 경우에만.
4. PDF 저장 및 SHA256 기록, GENERATED 후 변경 불가. 재발행은 revision +1.
5. 다운로드는 role/tenant/approval 검증 → 짧은 signed URL(또는 stream) → AUDIT 기록.
6. 실패 시 GENERATION_FAILED, 재시도는 기존 immutable snapshot 기준. 새 증거가 나타나면 신규 버전 승인.

## PDF 생성 합격 테스트
- 필수 근거 누락 상태 승인 거부, 독립 사용자 승인(정책), 한글 폰트·페이지 넘침·표 크기 검사.
- 원본 이미지의 개인정보/바코드 안전 마스킹, 동일 입력의 보고서 snapshot 동일성, generated PDF checksum 저장.
- 취소 또는 벌금 산정이 자동 진행되지 않음. 작성 담당자와 승인 담당자 audit 확인.
- 출력한 그래프는 전체 원본 데이터가 아닌 case에 속한 가명 요약 및 설명으로 제한.
