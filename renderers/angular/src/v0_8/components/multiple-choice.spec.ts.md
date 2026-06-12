# renderers/angular/src/v0_8/components/multiple-choice.spec.ts

## 개요

`MultipleChoice` 컴포넌트의 단위 테스트 파일이다. `MessageProcessor`를 모킹하여 독립적으로 컴포넌트의 렌더링 및 상호작용을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./multiple-choice`](./multiple-choice.ts.md) — 테스트 대상 `MultipleChoice` 컴포넌트
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (모킹 대상)
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`

## Exports

없음 (테스트 파일)

## 테스트 케이스

### 픽스처 및 모킹 설정 (`beforeEach`)

- `mockTheme`: `new Theme()` 인스턴스. `components.MultipleChoice`를 `{ container: 'container-class', label: 'label-class', element: 'select-class' }`로 설정.
- `mockProcessor`: `jasmine.createSpyObj`로 생성. `dispatch`, `resolvePath`, `getData` 메서드를 스파이로 설정. `dispatch`는 `Promise.resolve([])`를 반환.
- `mockOptions`: `[{label: {literalString: 'Option 1'}, value: 'opt1'}, {label: {literalString: 'Option 2'}, value: 'opt2'}]`
- `TestBed` 구성: `MultipleChoice` 임포트, `MessageProcessor`/`Theme`/`Catalog` 프로바이더 주입.
- 입력값 설정: `surfaceId='surface-1'`, `component={id:'mc-1', type:'MultipleChoice', weight:1}`, `weight=1`, `label={literalString:'Select an option'}`, `options=mockOptions`, `selections={literalArray:['opt1']}`.

---

### `'should create'`
- **검증**: 컴포넌트 인스턴스(`component`)가 truthy인지 확인.

---

### `'should render label and options'`
- **검증 동작**: 렌더링된 `<label>` 요소가 존재하고 텍스트가 `'Select an option'`인지 확인. `<option>` 요소가 2개이며 첫 번째는 텍스트 `'Option 1'`, 값 `'opt1'`, 두 번째는 텍스트 `'Option 2'`, 값 `'opt2'`인지 확인.
- **픽스처/모킹**: `By.css('label')`, `By.css('option')` 쿼리 사용.

---

### `'should trigger sendAction on change'`
- **검증 동작**: `<select>` 요소의 값을 `'opt2'`로 변경하고 `change` 이벤트를 디스패치했을 때, `mockProcessor.dispatch`가 호출되는지 확인. 가장 최근 호출의 첫 번째 인자 메시지가 `userAction.name === 'change'`이고 `userAction.context === {value: 'opt2'}`인지 검증.
- **픽스처/모킹**: `By.css('select')` 쿼리. `new Event('change')` 디스패치.
- **주의**: `selections`가 `literalArray` 형식(path 없음)이므로 `handleAction` → `sendAction` → `dispatch` 경로가 실행된다.

## 동작 흐름

`beforeEach`에서 모듈을 구성하고 컴포넌트를 생성한 뒤 `fixture.detectChanges()`로 초기 렌더링을 수행한다. 각 테스트는 독립적으로 DOM 상태 또는 스파이 호출을 검증한다.
