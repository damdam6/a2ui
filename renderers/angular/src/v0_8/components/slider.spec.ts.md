# renderers/angular/src/v0_8/components/slider.spec.ts

## 개요

`Slider` 컴포넌트의 단위 테스트 파일이다. `MessageProcessor`를 모킹하여 슬라이더의 렌더링, aria 접근성 속성, 입력 변경 시 액션 디스패치 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./slider`](./slider.ts.md) — 테스트 대상 `Slider` 컴포넌트
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (모킹 대상)
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`

## Exports

없음 (테스트 파일)

## 테스트 케이스

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockTheme`: `new Theme()`. `components.Slider`를 `{ container: 'slider-container', label: 'slider-label', element: 'slider-input' }`로 설정.
- `mockProcessor`: `jasmine.createSpyObj`로 `dispatch`, `resolvePath`, `getData` 메서드 스파이 생성. `dispatch`는 `Promise.resolve([])`를 반환.
- `TestBed` 구성: `Slider` 임포트, `MessageProcessor`/`Theme`/`Catalog` 프로바이더.
- 입력값: `surfaceId='surface-1'`, `component={id:'slider-1', type:'Slider', weight:1}`, `weight=1`, `label={literalString:'Volume'}`, `value={literalNumber:50}`, `minValue=0`, `maxValue=100`.

---

### `'should create'`
- **검증**: 컴포넌트 인스턴스가 truthy인지 확인.

---

### `'should render label and set input attributes'`
- **검증 동작**: `<label>` 요소가 존재하고 텍스트가 `'Volume'`인지 확인. `input[type="range"]` 요소의 `value`가 `'50'`, `min`이 `'0'`, `max`가 `'100'`인지 확인.
- **픽스처**: `By.css('label')`, `By.css('input[type="range"]')` 쿼리.

---

### `'should trigger sendAction with number context on input change'`
- **검증 동작**: `input[type="range"]`의 값을 `'75'`로 변경하고 `input` 이벤트를 디스패치했을 때 `mockProcessor.dispatch`가 호출되며, 가장 최근 호출의 메시지가 `userAction.name === 'change'`이고 `userAction.context === {value: 75}` (숫자 타입)인지 확인.
- **픽스처**: `new Event('input')` 디스패치. `value`는 `literalNumber` 형태로 직렬화됨.

---

### `'should use label as aria-label when label is set'`
- **검증 동작**: `label`이 설정된 경우 `input[type="range"]`의 `aria-label` 어트리뷰트가 `'Volume'`인지 확인.

---

### `'should use default aria-label when label is not set'`
- **검증 동작**: `label` 입력을 `null`로 변경하고 `detectChanges()` 후 `input[type="range"]`의 `aria-label` 어트리뷰트가 `'Slider'`인지 확인.

## 동작 흐름

모든 테스트는 `beforeEach`에서 공통 픽스처를 구성하고 `detectChanges()`로 초기 렌더링을 수행한 뒤 각각 독립적으로 DOM 또는 스파이 상태를 검증한다.
