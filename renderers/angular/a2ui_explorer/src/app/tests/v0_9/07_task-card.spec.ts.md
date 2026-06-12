# renderers/angular/a2ui_explorer/src/app/tests/v0_9/07_task-card.spec.ts

## 개요

`Task Card` 예제 컴포넌트의 렌더링 동작을 검증하는 Angular 통합 테스트 파일이다. `loadExample`을 통해 `'Task Card'` 예제를 로드한 뒤 텍스트 콘텐츠, 날짜/시간 입력 필드, 체크박스, 아이콘이 올바르게 렌더링되는지 확인한다. 동적 컴포넌트 초기화를 위해 `wait(100)` 딜레이를 사용하는 것이 특징이다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Task Card')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach`**
- `loadExample('Task Card')`를 비동기로 호출하여 `fixture`를 얻는다.
- `wait(100)`으로 동적 컴포넌트가 초기화될 시간을 확보한다.
- `fixture.detectChanges()`로 Angular 변경 감지를 수동으로 트리거한다.
- `getCanvas().textContent`로 캔버스 DOM의 텍스트를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 캔버스 텍스트에 `'Review pull request'`, `'Review and approve the authentication module changes.'`, `'Due'`, `'Backend'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `beforeEach`에서 로드된 `fixture`와 `textContent` 사용.

---

**`should render date time input`**
- 검증 동작: `fixture.nativeElement.querySelectorAll('input')`로 모든 입력 요소를 수집한 뒤, `type !== 'checkbox'`인 첫 번째 입력 요소가 존재하는지 확인하고, 해당 입력 필드의 `value`가 `'2025-12-15'`를 포함하는지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 직접 쿼리.

---

**`should render checkbox`**
- 검증 동작: `input[type="checkbox"]` 셀렉터로 체크박스 요소가 DOM에 존재하는지 확인한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 직접 쿼리.

---

**`should render icon`**
- 검증 동작: `.a2ui-icon` 클래스를 가진 요소가 DOM에 존재하는지 확인한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 직접 쿼리.

## 동작 흐름

각 테스트는 `beforeEach`에서 `'Task Card'` 예제를 로드하고 100ms 대기 후 변경 감지를 실행하여 공통 상태를 준비한다. 이후 각 `it` 블록이 텍스트 포함 여부, 특정 input 타입 존재 및 값, 아이콘 요소 존재를 순서 독립적으로 검증한다.
