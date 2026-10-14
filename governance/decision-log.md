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
