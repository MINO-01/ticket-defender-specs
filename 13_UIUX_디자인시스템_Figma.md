# 13. UI/UX 디자인 시스템 및 Figma 실행 가이드

> 이것은 프론트엔드와 Figma 팀의 **공동 단일 디자인 계약**이다. 브랜드·접근성 검증 전의 초기 토큰이며 확정색이 아니다.

## 정보구조와 디자인 원칙
- 목표: 분석의 화려함보다 작업 상태/근거/다음 액션의 선명함.
- 같은 위험 점수라도 확정 사실처럼 빨간 '불법' 뱃지 사용 금지; 표시 '조사 우선순위'.
- Track1 graph는 요약→관계 확장, Track2는 이미지/관찰 필드/검토 패널 split screen.
- 관측 필드의 `VISIBLE, ABSENT, UNREADABLE, AMBIGUOUS`를 텍스트·아이콘·툴팁으로 구분.
- 심각한 분석 실패와 정상 결과 0개를 다른 empty/error components로 표시.

## 디자인 토큰 v0 (CSS 변수 예)
```css
:root {
  --color-bg:#f8fafc; --color-surface:#ffffff;
  --color-text:#0f172a; --color-text-muted:#475569;
  --color-primary:#2563eb; --color-border:#cbd5e1;
  --color-success:#15803d; --color-warning:#a16207; --color-danger:#b91c1c;
  --space-1:4px; --space-2:8px; --space-3:12px; --space-4:16px;
  --space-6:24px; --space-8:32px;
  --radius-sm:6px; --radius-md:10px; --radius-lg:14px;
  --font-sm:14px; --font-md:16px; --font-lg:20px;
  --shadow-card:0 1px 3px rgba(15,23,42,.08);
}
```
모든 주요 텍스트와 상태 조합은 WCAG 대비·키보드 포커스 확인 후 최종화; 색만으로 구분하지 않는다. 지원 화면 폭: desktop 1440 / laptop 1024 / tablet 768, 모바일은 read-only 제한 여부 별도 결정.

## 공통 컴포넌트와 속성
| Component | 주요 props | 상태 |
|---|---|---|
| AppShell | tenantName, breadcrumb, nav | compact, loading, forbidden |
| KPIStat | label, value, period, info | loading, unavailable |
| StatusBadge | domain(job/case/report), status | text/icon + tooltip |
| DataTable | columns, sort, page, actions | skeleton, empty, error |
| EvidenceViewer | assetUrl, imageMetadata, annotations | loading, redacted, denied |
| ExtractionField | label, value, observationState, revisedBy | visible/unreadable/absent |
| GraphCanvas | nodes, edges, selectedNode, maxNodes | loading/overflow/empty |
| DecisionTimeline | transitions, actors | empty/history |
| ConfirmDialog | action, impact, reasonRequired | idle/submitting/error |
| PdfPreview | version, status, pageCount | draft/rendering/generated/error |
| ErrorPanel | errorCode, requestId, retryable | network/contract/forbidden |

## Figma 팀 작업 절차
1. 페이지 `00 Foundations`에서 color/typography/spacing/radius/semantics 변수 생성, `light` mode 먼저 구성.
2. `01 Components`에서 버튼(primary/secondary/danger), input/textarea, dropdown, statusBadge, table, tab, modal, toast, split-pane를 **Variants + Auto layout**으로 만듦.
3. `02 IA and Flows`에서 메뉴 트리와 주요 user flow 선 연결: import→job→finding→case→report; fan-report→extraction→case→report.
4. `03 Wireframes`에 1440px frame 7개(Overview, Import, Finding, Graph, Report review, Case, PDF); 각 loading/empty/error/forbidden/running/resolved 상태도 생성.
5. `04 High Fidelity`에서 같은 component instance를 사용한 화면 디자인, 접근성 검토.
6. `05 Handoff`에 버튼 action + API endpoint + role + 상태 가드와 컴포넌트 속성 이름을 부착.
7. Dev Mode로 CSS 변수와 component 이름을 React 소스와 1:1 대응하고 화면별 인수 checklist 사인오프.

## 대표 와이어프레임 구조
```
┌ tenant/account header ─────────────────────────────────────┐
│ sidebar │ Title / Breadcrumb      [Filter]  [Primary Action] │
│         ├─────────────────────────────────────────────────┤
│         │ Job status + failure/retry / Analysis summary    │
│         ├────────────────────────┬────────────────────────┤
│         │ Graph or Image viewer  │ Evidence / Fields       │
│         │                        │ Source / Decision       │
│         ├────────────────────────┴────────────────────────┤
│         │ Audit timeline | related reservations | reports  │
└─────────┴──────────────────────────────────────────────────┘
```

## 개발 연결
Figma Button action spec은 `ui-actions.csv`(권한·상태)로 export하여 개발 이슈와 연결. 컴포넌트 구현은 React+TypeScript strict, server state는 TanStack Query; 컴포넌트 시나리오는 Storybook 또는 동등한 테스트 페이지에서 리뷰. 모든 화면 전용 오류/빈상태 텍스트 포함.
