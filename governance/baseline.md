# 검증 기준선 기록

이 기록은 **아래 특정 commit에 대한 과거 검증**이며 현재 또는 미래 HEAD의 테스트 통과를 보증하지 않는다. 실제 구현 판단에는 대상 HEAD에서 다시 수행한 검증이 필요하다.

| 저장소 | 기준 commit | 당시 도구 및 검증 결과 |
|---|---|---|
| Backend | `8b4f756b78b911deb0462aa057971891e2b506a4` | Java 21.0.9, Gradle 9.7.1; 기존 Spring 테스트 35개 중 33개 성공, 2개 비활성 |
| Data | `cb388142491cd08cc77cfd3d8e113c808a980f28` | Python 3.13.3; Python 소스 12개 AST 문법 검사 성공; 자동 테스트 파일 및 pytest 없음 |
| Specs | `d697203b7c31d028ad3c7ea862f525cc5780ab6f` | 첫 원본 명세 Baseline commit; 제품 코드 테스트와 별개 |

Backend/Data 검증은 TD-00B-2의 2026-10-13 기준선 실행 기록이다. Spring의 Mock 단위 테스트와 H2 JPA 테스트 통과 범위만 확인되었고, Python AST는 문법 검사일 뿐 API·그래프 동작 검증이 아니다.

다음은 해당 기준선에서 검증되지 않았다: Spring 전체 컨텍스트, 실제 MySQL 호환성, Spring–FastAPI 통합, Neo4j/GDS/Louvain 실행, Circuit Breaker AOP, Gemini 외부 호출, 실제 운영 보안. 이 항목은 통과나 구현 완료로 표시하지 않는다.
