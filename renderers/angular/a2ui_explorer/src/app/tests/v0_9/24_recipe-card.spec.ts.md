# renderers/angular/a2ui_explorer/src/app/tests/v0_9/24_recipe-card.spec.ts

## 개요

`Recipe Card` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 탭 기반 UI의 초기 렌더링(탭 제목, 개요 내용, 아이콘)과 탭 전환 인터랙션(Ingredients, Instructions 탭 클릭 후 내용 변경)을 검증하는 5개의 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Recipe Card', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Recipe Card')`로 생성.
- `textContent`: `string` — `getCanvas().textContent` (초기 Overview 탭 상태).
- `beforeEach`는 `async`로 동작한다.

#### 테스트 케이스

1. **`should render tab titles`**
   - 검증 동작: 캔버스 텍스트에 `'Overview'`, `'Ingredients'`, `'Instructions'` 탭 제목이 모두 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render Overview content`**
   - 검증 동작: 캔버스 텍스트에 `'Mediterranean Quinoa Bowl'`, `'4.9'`, `'15 min prep'`, `'20 min cook'`, `'Serves 4'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

3. **`should render icon`**
   - 검증 동작: `.a2ui-icon` 클래스 요소가 DOM에 존재하는지 확인한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('.a2ui-icon')`.

4. **`should switch to Ingredients tab`**
   - 검증 동작: `.a2ui-tab-button` 클래스를 가진 요소 중 텍스트에 `'Ingredients'`가 포함된 탭을 찾아 클릭한다. `wait(50)` 후 `fixture.detectChanges()`를 다시 호출한다. 이후 `getCanvas().textContent`를 새로 읽어 `'1 cup quinoa'`와 `'1 cucumber, diced'`가 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-tab-button')`, `getCanvas()`.

5. **`should switch to Instructions tab`**
   - 검증 동작: `.a2ui-tab-button` 중 텍스트에 `'Instructions'`가 포함된 탭을 찾아 클릭한다. `wait(50)` 후 변경 감지를 수행한다. 이후 캔버스 텍스트에 `'Rinse quinoa and bring to a boil in water.'`와 `'Mix with diced vegetables.'`가 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-tab-button')`, `getCanvas()`.

## 동작 흐름

`beforeEach`에서 `'Recipe Card'` 예제를 로드하고 Overview 탭 상태의 텍스트 스냅샷을 설정한다. 탭 전환 테스트(케이스 4, 5)는 각각 탭 버튼 DOM 탐색 → 클릭 → `wait(50)` → 변경 감지 → 새 캔버스 텍스트 재취득 → 내용 검증의 흐름을 따른다.
