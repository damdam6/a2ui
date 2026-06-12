# renderers/angular/a2ui_explorer/src/app/tests/v0_9/16_workout-summary.spec.ts

## 개요

`Workout Summary` 예제 컴포넌트의 렌더링을 검증하는 Angular 통합 테스트 파일이다. 운동 완료 화면의 제목·통계 항목 텍스트(Duration, Calories, Distance)와 아이콘(`directions_run`)이 올바르게 렌더링되는지 확인한다. 버튼 인터랙션 테스트는 포함하지 않는다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Workout Summary')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach`**
- `loadExample('Workout Summary')`를 비동기로 호출하여 `fixture`를 얻는다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 텍스트에 `'Workout Complete'`, `'Duration'`, `'32:15'`, `'Calories'`, `'385'`, `'Distance'`, `'5.2 km'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render icon`**
- 검증 동작: `.a2ui-icon` 요소가 DOM에 존재하는지, 텍스트에 `'directions_run'` 아이콘 리터럴이 포함되어 있는지 확인한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리 및 `textContent`.

## 동작 흐름

`beforeEach`에서 `'Workout Summary'` 예제를 로드하고 텍스트를 추출한다. 두 개의 `it` 블록이 각각 운동 통계 텍스트와 아이콘 존재를 독립적으로 검증한다.
