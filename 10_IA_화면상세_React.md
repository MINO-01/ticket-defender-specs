# 10. IA 및 React + TypeScript 화면 상세 명세

## 메뉴/라우트
- `/login`: 인증/세션 상태
- `/app/overview`: 일별 탐지·검토·작업 실패 KPI, 서비스 상태
- `/app/imports`, `/app/imports/:id`: Track1 batch 반입·검증 결과
- `/app/analysis/jobs`, `/app/analysis/jobs/:id`: 실행·재시도·실패 원인
- `/app/analysis/findings`, `/app/analysis/findings/:id`: 군집 목록·그래프·근거
- `/app/reports`, `/app/reports/:id`: 팬 제보 목록·이미지/필드 검토
- `/app/cases`, `/app/cases/:id`: 사건 목록·증거·담당자·상태 전이
- `/app/cases/:id/documents`: PDF 버전·승인·다운로드
- `/app/settings/members`, `/app/settings/sources`: 고객사 관리자 설정
- `/app/audit`: 감사 이력 (Auditor/Admin)

## 화면 상태(모든 비동기 페이지)
`initial | loading | ready(data) | empty(no result) | error(network/server) | forbidden | stale(refresh)`. 분류 오류일 때 `ready(empty)` 표시 금지. 검색·정렬·pagination은 서버 측; URL query에 필터 저장해 새로고침 재현.

## 화면별 상세
| View | 표시·필드 | 사용자 액션 | 권한·가드 | 로딩·오류 |
|---|---|---|---|---|
| Overview | 작업별 큐/실패, 사건 상태별 수, 최근 검토 | 지표→목록 이동 | tenant/member | 집계 실패 알림, 0과 구분 |
| Import | 이벤트, 출처, schemaVersion, 파일 | 업로드/검증/분석 요청 | Analyst/Admin; batch ACCEPTED만 분석 | 행별 422 오류, 크기 오류 |
| Job detail | status, progress, attempt, failureCode | refresh/retry | 재시도 FAILED/RETRY_WAIT 및 횟수 | 장애 사유와 requestId 표시 |
| Finding list | event, score, members, signals | 필터/열기/Case 연결 | 분석 결과 COMPLETED 한정 | no matching findings 메시지 |
| Graph explorer | 선택 community + 근거 token 종류 + 예매 | 노드 선택/일부 확장/사건 생성 | tenant scope | 과도한 그래프 렌더 제한 |
| Fan report split | 원본 이미지 왼쪽, 오른쪽 extracted field, source | 비교/교정/추가자료요청/대조 | Reviewer만 교정, UNREADABLE 명시 | OCR/VLM 실패/제보 접수 성공 분리 |
| Case detail | timeline, findings/evidence, review notes, status | 배정/보류/해소/승인요청 | 버전 잠금, 허용된 전이만 | 409 발생 시 재조회 |
| PDF center | 스냅샷 요약, 승인자, hash, version | draft/approve/render/download | APPROVER 승인, GENERATED만 내려받기 | 생성실패 재시도 안내 |
| Members/settings | 사용자/역할/데이터 소스 | 초대/권한 변경 | Tenant Admin | 자기권한 초과 금지 |

## 버튼 활성 조건 소스
백엔드 `allowedActions: ["CASE_REVIEW","REQUEST_APPROVAL",...]`를 응답에 포함할 수 있으나, 이는 안내용. 서버에서 항상 상태+역할+tenant+version 재검증. 예: `approve`는 `status===READY_FOR_APPROVAL && hasRole(APPROVER) && evidenceValidated && !stale`.

## 프론트 기술 정책
- Vite + React + TypeScript strict; React Router, TanStack Query(server state), React Hook Form+Zod(forms); Zustand는 꼭 필요한 단기 UI 선택상태에 한정.
- OpenAPI generator(orval/openapi-typescript)로 typed API Client 생성; 손수 DTO 복제 방지.
- JSON error code와 requestId를 공통 오류 컴포넌트로 제공.
- 접근성: 키보드 포커스, 색만으로 위험도 표현 금지, 목록 대신 그래프만 제시하지 않음.
- 그래프는 Cytoscape.js 우선 평가(레이아웃/선택/expand), D3.js는 필요 시 세밀한 bespoke 시각화. 1회 최대 노드 수/edges 수 제한 및 서버 pagination.
- 파일 URL 클라이언트 로그/브라우저 storage에 영구 보관 금지; 비공개 접근 토큰 만료.

## 화면 계약 테스트
UI Mock Service Worker(MSW)로 Job RUNNING→FAILED/COMPLETED, Case version conflict, Report approval permissions, extraction UNREADABLE, cross tenant 404, network error를 재현. E2E는 Playwright.
