# renderers/angular/src/v0_8/components/slider.ts

## 개요

`Slider` 컴포넌트는 레이블과 `<input type="range">` 요소를 렌더링하는 Angular 독립형 컴포넌트다. `DynamicComponent<SliderNode>`를 상속하며, 슬라이더 값 변경 시 데이터 모델 직접 업데이트 또는 `userAction` 디스패치를 수행한다. 접근성을 위해 `aria-label`을 레이블 텍스트 또는 기본값 `'Slider'`로 설정한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `NumberValue`, `SliderNode`, `StringValue` 타입
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스

## Exports

| 이름 | 종류 |
|------|------|
| `Slider` | 클래스 (Angular Component) |

## 상세 명세

### `Slider` 클래스

`@Component` 데코레이터 설정:
- `selector`: `'a2ui-slider'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `styles`: `:host { display: block; }`

`DynamicComponent<SliderNode>`를 상속한다.

#### 필드

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `label` | `InputSignal<StringValue \| null>` | `null` | 슬라이더 위에 표시할 레이블 |
| `value` | `InputSignal<NumberValue \| null>` | required | 현재 슬라이더 값 (`literalNumber` 또는 `path` 포함 객체) |
| `minValue` | `InputSignal<number>` | `0` | 슬라이더 최솟값 |
| `maxValue` | `InputSignal<number>` | `100` | 슬라이더 최댓값 |
| `inputId` | `string` (protected) | `super.getUniqueId('a2ui-slider-input')` | `<input>` 요소의 고유 ID |
| `labelId` | `string` (protected) | `super.getUniqueId('a2ui-slider-label')` | `<label>` 요소의 고유 ID |

#### computed 시그널

- **`resolvedLabel`** (protected): `super.resolvePrimitive(this.label())`로 `StringValue`를 문자열로 변환한다.
- **`resolvedValue`** (protected): `super.resolvePrimitive(this.value())`로 `NumberValue`를 숫자로 변환한다.

#### `onInput(event: Event): void`
1. `Number((event.target as HTMLInputElement).value)`로 새 숫자 값을 추출한다.
2. `this.value()`를 가져와 `'path'` 키가 존재하고 값이 있는지 확인한다.
3. **path가 있는 경우**: `this.processor.processMessages([{ dataModelUpdate: { surfaceId, path, contents: [{key: '.', valueNumber: value}] } }])`를 호출한다. `path`는 `this.processor.resolvePath(valueNode.path, this.component().dataContextPath)`로 해석된다.
4. **path가 없는 경우**: `this.handleAction('change', {value})`를 호출한다.

#### `handleAction(name: string, context: Record<string, unknown>): void` (private)
`super.sendAction()`을 호출한다. `context`의 각 항목을 `{key, value}` 형태로 변환하며, 값이 `number`이면 `{literalNumber: val}`, 그 외에는 `{literalString: String(val)}`을 사용한다.

#### 템플릿 구조
`theme.components.Slider.container` 클래스와 `theme.additionalStyles?.Slider` 스타일이 적용된 div 내부에:
1. `@if (resolvedLabel())` 블록: 레이블이 있을 때만 `<label>` 렌더링. `theme.components.Slider.label` 클래스, `[id]="labelId"`.
2. `<input type="range">`: `theme.components.Slider.element` 클래스, `[id]="inputId"`, `[attr.aria-label]="resolvedLabel() ? resolvedLabel() : 'Slider'"`, `[min]="minValue()"`, `[max]="maxValue()"`, `[value]="resolvedValue() ?? 0"`, `(input)="onInput($event)"`.

## 동작 흐름

1. 컴포넌트 렌더링 시 `resolvedLabel`과 `resolvedValue`를 computed로 계산하여 템플릿에 반영.
2. 레이블이 있으면 `<label>` 요소 표시, 없으면 숨김. 접근성을 위한 `aria-label`은 레이블 텍스트 또는 기본 `'Slider'`.
3. 사용자가 슬라이더를 조작하면 `onInput` 실행.
4. `value`에 `path`가 있으면 즉시 로컬 데이터 모델 업데이트 (UI 즉각 반응 + 불필요한 네트워크 요청 방지).
5. `path`가 없으면 `change` 액션을 숫자 값과 함께 상위 처리기로 디스패치.
