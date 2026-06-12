# renderers/angular/a2ui_explorer/src/app/tests/v0_8/07_task-card.spec.ts

## 개요

v0.8 버전의 "Task Card (basic)" 예제 컴포넌트가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 텍스트 콘텐츠만 검증하는 단일 케이스로 구성되어 있으며, 버튼 액션 테스트는 포함하지 않는다. `DemoComponent` 픽스처를 직접 사용하지 않고 `getCanvas()`로 텍스트만 추출한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 유틸리티

## Exports

이 파일은 `describe` 블록으로 테스트를 등록할 뿐 명시적으로 export하는 항목이 없다.

## 테스트 케이스 명세

### 스위트: `Example: Task Card (basic) (v0.8)`

**픽스처 및 공유 변수**
- `textContent: string` — 캔버스 요소의 텍스트 내용

**`beforeEach`**
`loadExample('Task Card (basic)', Version.V0_8)`를 비동기로 호출하여 예제를 로드하고, `getCanvas().textContent`로 렌더링된 텍스트를 추출해 `textContent`에 저장한다. `ComponentFixture`를 별도로 유지하지 않는다.

---

#### 테스트 1: `should render expected text content`
- **검증 동작**: 캔버스에 렌더링된 텍스트가 태스크 카드에 필요한 모든 정보를 포함하는지 확인한다.
- **검증 항목**: `'Review pull request'`(태스크 제목), `'Review and approve the authentication module changes.'`(태스크 설명), `'Today'`(마감일), `'Backend'`(태그/카테고리), `'priority_high'`(우선순위 아이콘/레이블) 포함 여부를 `toContain`으로 각각 단언한다.
- **픽스처/모킹**: `loadExample` 호출 결과 반환된 `textContent` 사용.

## 동작 흐름

`beforeEach`에서 v0.8 Task Card 예제를 로드하고 캔버스 텍스트를 가져온다. 단일 텍스트 렌더링 테스트에서 과제의 제목, 설명, 마감일, 카테고리, 우선순위 레이블 다섯 개 문자열이 모두 텍스트에 포함되어 있는지 순차적으로 검증한다.
