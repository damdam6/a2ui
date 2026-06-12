# renderers/angular/src/v0_8/components/multiple-choice.ts

## 개요

`MultipleChoice` 컴포넌트는 레이블과 드롭다운 `<select>` 요소를 렌더링하는 Angular 독립형 컴포넌트다. `DynamicComponent<MultipleChoiceNode>`를 상속하며, 옵션 목록과 현재 선택값을 입력으로 받아 사용자의 선택 변경 시 데이터 모델 업데이트 또는 `userAction` 디스패치를 수행한다. 선택값 노드에 `path`가 있으면 로컬 데이터 모델을 직접 업데이트하고, 없으면 `change` 액션을 전송한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- [`../types`](../types.ts.md) — `AnyComponentNode`, `MultipleChoiceNode`, `StringValue` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `MultipleChoice` | 클래스 (Angular Component) |

## 상세 명세

### `MultipleChoice` 클래스

`@Component` 데코레이터 설정:
- `selector`: `'a2ui-multiple-choice'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `styles`: `:host { display: block; }`

`DynamicComponent<MultipleChoiceNode>`를 상속한다.

#### 필드

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `label` | `InputSignal<StringValue \| null>` | `null` | 드롭다운 위에 표시할 레이블 |
| `options` | `InputSignal<{label: StringValue; value: string}[]>` | required | 선택 가능한 옵션 목록 |
| `selections` | `InputSignal<AnyComponentNode \| null>` | required | 현재 선택값 노드 (`literalArray` 또는 `path` 포함 객체) |
| `selectId` | `string` (protected) | `super.getUniqueId('a2ui-multiple-choice')` | `<label>`과 `<select>`를 연결하는 고유 ID |

#### computed 시그널

- **`resolvedLabel`** (protected): `this.resolvePrimitive(this.label())`를 호출하여 `StringValue`를 실제 문자열로 변환한다.
- **`resolvedOptions`** (protected): `options()` 배열을 순회하며 각 옵션의 `label`을 `resolvePrimitive`로 변환한 `{label: string, value: string}[]`를 반환한다.
- **`resolvedSelections`** (protected): `selections()` 값을 검사하여 `'literalArray'` 키가 존재하면 `literalArray` 속성을 `string[]`로 캐스팅해 반환한다. 그렇지 않으면 빈 배열 `[]`를 반환한다.

#### `onChange(event: Event): void`
1. `(event.target as HTMLSelectElement).value`로 새로 선택된 문자열 값을 추출한다.
2. `this.selections()`를 가져와 `'path'` 키가 존재하고 값이 있는지 확인한다.
3. **path가 있는 경우**: `this.processor.processMessages([{ dataModelUpdate: { surfaceId, path, contents: [{key: '.', valueString: JSON.stringify({literalArray: [value]})}] } }])`를 호출하여 로컬 데이터 모델을 직접 업데이트한다. `path`는 `this.processor.resolvePath(selectionsNode.path, this.component().dataContextPath)`로 해석된다.
4. **path가 없는 경우**: `this.handleAction('change', {value})`를 호출한다.

#### `handleAction(name: string, context: Record<string, unknown>): void` (private)
`super.sendAction()`을 호출한다. `context` 객체를 `{key, value}` 배열로 변환하며, 값이 `number`이면 `{literalNumber: val}`, 그 외에는 `{literalString: String(val)}`을 사용한다.

#### 템플릿 구조
`theme.components.MultipleChoice.container` 클래스와 `theme.additionalStyles?.MultipleChoice` 스타일이 적용된 div 내부에:
1. `<label>`: `theme.components.MultipleChoice.label` 클래스, `[for]="selectId"`, 텍스트는 `resolvedLabel()`.
2. `<select>`: `theme.components.MultipleChoice.element` 클래스, `[id]="selectId"`, `[value]="resolvedSelections()[0] || ''"`, `(change)="onChange($event)"`. `@for (option of resolvedOptions(); track option.value)` 블록으로 `<option>` 렌더링.

## 동작 흐름

1. 컴포넌트 렌더링 시 `resolvedLabel`, `resolvedOptions`, `resolvedSelections`를 computed로 계산하여 템플릿에 반영.
2. 현재 선택값은 `resolvedSelections()[0]`으로 `<select>`의 value를 설정 (단일 선택 기준).
3. 사용자가 드롭다운에서 옵션 선택 시 `onChange` 실행.
4. `selections`에 `path`가 있으면 즉시 로컬 데이터 모델 업데이트 (UI 즉각 반응 + 불필요한 네트워크 요청 방지).
5. `path`가 없으면 `change` 액션을 상위 처리기로 디스패치.
