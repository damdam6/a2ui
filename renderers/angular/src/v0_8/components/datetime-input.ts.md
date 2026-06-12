# renderers/angular/src/v0_8/components/datetime-input.ts

## 개요

`DateTimeInput`은 날짜·시간 입력 필드를 렌더링하는 Angular 컴포넌트다. `enableDate`/`enableTime` 플래그 조합에 따라 `<input type>` 을 `date`, `time`, `datetime-local` 중 하나로 동적 결정하며, 레이블과 입력 요소를 함께 렌더링한다. 값 변경 시 데이터 모델 경로가 있으면 직접 로컬 데이터 모델을 갱신하고, 없으면 액션 이벤트를 발송한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md): `DynamicComponent` 기반 클래스
- [`../types`](../types.ts.md): `DateTimeInputNode`, `StringValue` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `DateTimeInput` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### `DateTimeInput` 클래스

**선언:** `export class DateTimeInput extends DynamicComponent<DateTimeInputNode>`

**데코레이터 설정:**
- `selector`: `'a2ui-datetime-input'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**인라인 스타일:** `:host { display: block; }`

**템플릿 구조:**
- 최상위 `<div>` 에 `[class]="theme.components.DateTimeInput.container"`, `[style]="theme.additionalStyles?.DateTimeInput"` 바인딩
- 내부에 `<label [class]="theme.components.DateTimeInput.label" [for]="inputId">` 로 레이블 텍스트 출력
- `<input [type]="inputType()" [class]="theme.components.DateTimeInput.element" [id]="inputId" [value]="resolvedValue() ?? ''" (change)="onChange($event)">`

**입력 신호(Inputs):**

| 입력 이름 | 타입 | 기본값 |
|-----------|------|--------|
| `label` | `StringValue \| null` | `null` |
| `value` | `StringValue \| null` | (required) |
| `enableDate` | `boolean` | `true` |
| `enableTime` | `boolean` | `false` |

**필드:**

- `inputId` — `protected readonly string`. `super.getUniqueId('a2ui-datetime-input')` 로 초기화되는 고유 ID. `<label>`의 `for`와 `<input>`의 `id`를 연결하는 데 사용된다.

**computed 신호:**

- `inputType` — `protected`. `enableDate()`와 `enableTime()` 조합을 검사한다. 둘 다 true이면 `'datetime-local'`, `enableTime()`만 true이면 `'time'`, 그 외는 `'date'`를 반환한다.
- `resolvedLabel` — `protected readonly`. `super.resolvePrimitive(this.label())`를 통해 `StringValue`를 원시 문자열로 변환한다.
- `resolvedValue` — `protected`. `super.resolvePrimitive(this.value())`로 현재 값을 문자열로 변환한다.

**메서드:**

#### `onChange(event: Event): void`

HTML `change` 이벤트 핸들러. `event.target`을 `HTMLInputElement`로 캐스팅해 현재 `value` 문자열을 추출한다.

분기 처리:
1. `this.value()` 가 객체이고 `'path'` 키가 있으며 값이 truthy이면 — 로컬 데이터 모델을 직접 갱신한다. `this.processor.processMessages([{ dataModelUpdate: { surfaceId, path, contents: [{key: '.', valueString: value}] } }])` 형태로 호출하며, `path`는 `this.processor.resolvePath(valueNode.path, this.component().dataContextPath)`로 해석한다.
2. 그 외 — `this.handleAction('change', {value})`를 호출해 액션 이벤트를 발송한다.

#### `private handleAction(name: string, context: Record<string, unknown>): void`

`super.sendAction()`을 래핑하는 헬퍼. `context` 객체의 각 엔트리를 `{key, value}` 쌍으로 변환한다. `val`이 숫자이면 `{literalNumber: val}`, 아니면 `{literalString: String(val)}` 형태로 직렬화한다.

## 동작 흐름

1. 컴포넌트가 `value`, `enableDate`, `enableTime` 입력을 받는다.
2. `inputType()` computed 신호가 세 플래그 조합에 따라 `<input type>`을 결정한다.
3. 사용자가 값을 변경하면 `onChange`가 호출된다.
4. `value` 입력에 `path` 경로 참조가 있으면 데이터 모델 직접 갱신으로 즉각적인 UI 피드백을 제공하고, 없으면 액션으로 서버에 알린다.
