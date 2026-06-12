# renderers/angular/a2ui_explorer/src/app/tests/v0_9/23_step-counter.spec.ts

## 개요

`Step Counter` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 걸음 수 추적 UI의 텍스트 레이블과 아이콘 존재 여부를 검증하는 2개의 테스트 케이스로 구성된다. 인터랙션 테스트는 포함되지 않는다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Step Counter', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Step Counter')`로 생성.
- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로 동작한다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `"Today's Steps"`, `'Distance'`, `'Calories'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render icon`**
   - 검증 동작: `.a2ui-icon` 클래스 요소가 DOM에 존재하는지 확인하고, 캔버스 텍스트에 Material 아이콘 이름 `'person'`이 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('.a2ui-icon')`, `textContent`.

## 동작 흐름

`beforeEach`에서 `'Step Counter'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 두 테스트는 각각 렌더링된 UI 레이블과 아이콘을 독립적으로 검증한다.
