# Ticket Defender — Repo-Based Spec Pack v2.0

2026-10-11 / 2 GitHub public repositories / **15개 개발 기준 문서(00 감사 + 01~14 요청 명세)** + 실행백로그 + 2 OpenAPI 제안 계약 + 2 Mermaid 상태도.

## 중요한 상태 표시
- **실제 확인**: 저장소에서 원문 파일 열람에 성공한 범위. 실제 빌드/데이터베이스 접속을 수행한 것은 아님.
- **제안**: 새 DB 테이블, API, 사용자 역할, UI, worker 등은 실제 구현 여부 확인 필요하며 자동 적용 금지.
- GitHub 폴더 목록 API와 전체 tree, repo clone은 이 환경에서 제한되었으므로, 모든 기존 Java 클래스/SQL migrations/test 경로 전수 확인이 완료된 것이 아님.
- GitHub `main`은 이후 변경될 수 있다. Codex 적용 시작 시 **정확한 HEAD commit SHA 기록** 필수.

## 문서 목차
| 순서 | 파일 | 담당 |
|---|---|---|
| 00 | 00_시작하기_현행코드감사와_결정사항.md | Tech Lead |
| 01 | 01_PRD_제품요구사항.md | PM |
| 02 | 02_기능명세서.md | PM/Engineering |
| 03 | 03_역할권한정의.md | Security/Backend |
| 04 | 04_사용자시나리오_업무흐름_상태전이.md | PM/Frontend/Backend |
| 05 | 05_시스템아키텍처_워크비동기.md | Architect/Backend/Data |
| 06 | 06_ERD_데이터사전_MySQL_Neo4j.md | Backend/Data |
| 07 | 07_API명세_외부내부_계약.md | Backend/Data/Frontend |
| 08 | 08_데이터수집_토큰화_중복.md | Security/Data |
| 09 | 09_탐지알고리즘_평가.md | Data/Research |
| 10 | 10_IA_화면상세_React.md | Frontend/PM |
| 11 | 11_보안_개인정보_위협모델.md | Security/Compliance |
| 12 | 12_증거관리_PDF.md | Backend/PM |
| 13 | 13_UIUX_디자인시스템_Figma.md | Designer/Frontend |
| 14 | 14_테스트_운영_릴리즈.md | QA/Infra |
| 실행 | 15_개발순서_Codex_이슈백로그.md | Tech Lead |

## 인터페이스 계약
- `contracts/track1-internal-v1-proposal.yaml`: Spring 기존 MacroAnalysisRequest/Response를 기준으로 Python에 구현할 목표 **내부 API**. 현재 Python은 다른 라우트/DTO/WCC → 아직 일치하지 않음.
- `contracts/admin-api-v1-proposal.yaml`: Spring 신규 공용 관리 API 일부; 실제 구현 아님. DTO/권한/상태 전이 검토 및 코드 생성 기준.
- `contracts/analysis-job.mmd`: 기술적 작업 상태도.
- `contracts/case-workflow.mmd`: 담당자 사건 상태도.

## 검토한 소스 파일
- https://raw.githubusercontent.com/MINO-01/ticket-defender-backend/main/build.gradle
- https://raw.githubusercontent.com/MINO-01/ticket-defender-backend/main/src/main/resources/application.yml
- https://raw.githubusercontent.com/MINO-01/ticket-defender-backend/main/src/main/java/com/ticket/defender_core/adapter/out/api/FastApiAdapter.java
- https://raw.githubusercontent.com/MINO-01/ticket-defender-backend/main/src/main/java/com/ticket/defender_core/adapter/out/api/dto/MacroAnalysisRequest.java
- https://raw.githubusercontent.com/MINO-01/ticket-defender-backend/main/src/main/java/com/ticket/defender_core/adapter/out/api/dto/MacroAnalysisResponse.java
- https://raw.githubusercontent.com/MINO-01/ticket-defender-data/main/api/routes/graph_routes.py
- https://raw.githubusercontent.com/MINO-01/ticket-defender-data/main/api/routes/vlm_routes.py
- https://raw.githubusercontent.com/MINO-01/ticket-defender-data/main/services/graph_service.py
- https://raw.githubusercontent.com/MINO-01/ticket-defender-data/main/services/vlm_service.py
- https://raw.githubusercontent.com/MINO-01/ticket-defender-data/main/schemas/dto.py
- https://raw.githubusercontent.com/MINO-01/ticket-defender-data/main/docker-compose.yml

## 추천 승인 흐름
PM이 01-04를 확인하고 범위 승인 → Tech Lead가 05-09 계약 및 11 보안 승인 → Frontend Lead가 10·13, Backend Lead가 12 확인 → QA가 14 인수 기준 고정. 구현 시작은 **00 감사와 TD-00 기준 테스트 완료 후**. 문서 승인만으로 실서비스 법적·보안 검증이 완료되었다고 보지 않는다.
