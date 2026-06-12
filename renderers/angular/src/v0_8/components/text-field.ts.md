# renderers/angular/src/v0_8/components/text-field.ts

## 개요

텍스트 입력 필드를 렌더링하는 Angular standalone 컴포넌트다. 레이블과 `<input>` 요소를 한 쌍으로 구성하며, `textFieldType` 입력에 따라 HTML 입력 타입을 동적으로 결정한다. 사용자가 값을 변경할 때, 바인딩된 데이터 경로가 있으면 로컬 데이터 모델을 직접 업데이트하고, 없으면 서버로 액션 이벤트를 발송한다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — 기반 클래스 `DynamicComponent`
- [`../types`](../types.ts.md) — 타입 `ResolvedTextField`, `StringValue`, `TextFieldNode`

## Exports

| 이름 | 종류 |
|---|---|
| `TextField` | Angular 컴포넌트 클래스 |

## 상세 명세

### `TextField` 클래스

`DynamicComponent<TextFieldNode>`를 상속하는 Angular 컴포넌트.

**데코레이터 메타데이터**
- `selector`: `'a2ui-text-field'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `template`: `<div>`(컨테이너) 안에 `<label>`과 `<input>`을 나란히 배치. 컨테이너와 레이블, 입력 요소에 각각 테마 클래스와 추가 스타일을 바인딩.
- `styles`: `:host { display: block; }`

---

#### 입력 신호(Inputs)

| 이름 | 타입 | 기본값 | 필수 |
|---|---|---|---|
| `label` | `StringValue \| null` | — | 필수(`input.required`) |
| `text` | `StringValue \| null` | `null` | 선택 |
| `textFieldType` | `ResolvedTextField['textFieldType']` | `'shortText'` | 선택 |

---

#### 내부 필드 및 계산 신호

- `inputId`: `super.getUniqueId('a2ui-text-field')` 호출로 결정되는 고유 식별자. `<label for>`와 `<input id>`를 연결하는 데 사용.
- `resolvedLabel`: `computed(() => super.resolvePrimitive(this.label()))`. `label` 신호가 변경될 때마다 실제 문자열 값으로 변환.
- `resolvedText`: `computed(() => super.resolvePrimitive(this.text()))`. `text` 신호가 변경될 때마다 실제 문자열 값으로 변환.
- `htmlInputType`: `computed`로 `textFieldType()`을 switch-case로 분기.
  - `'number'` → `'number'`
  - `'date'` → `'date'`
  - 기타(기본) → `'text'`

---

#### `onInput(event: Event): void`

`<input>` 요소의 `(input)` 이벤트 핸들러.

1. `(event.target as HTMLInputElement).value`에서 현재 입력값을 추출.
2. `this.text()` 신호 값(`textNode`)을 조회.
3. `textNode`가 객체이고 `'path'` 속성이 존재하며 truthy한 경우 (데이터 경로 바인딩):
   - `this.processor.processMessages([{ dataModelUpdate: { surfaceId, path, contents } }])`를 호출하여 로컬 데이터 모델을 직접 수정. 경로는 `this.processor.resolvePath(textNode.path, this.component().dataContextPath)`로 해석. `contents`는 `[{key: '.', valueString: value}]`.
4. 데이터 경로가 없는 경우: `this.handleAction('input', {value})`를 호출.

---

#### `handleAction(name: string, context: Record<string, unknown>): void` (private)

`super.sendAction({ name, context: ... })`를 호출하여 사용자 액션을 상위로 위임한다. `context` 객체의 각 엔트리를 순회하여 값이 `number`이면 `{literalNumber: val}`, 아니면 `{literalString: String(val)}`으로 래핑한다.

## 동작 흐름

컴포넌트가 렌더링되면 `label`, `text`, `textFieldType` 입력 신호에 기반한 계산 신호들이 자동으로 HTML을 갱신한다. 사용자가 입력값을 변경하면 `onInput`이 실행되어, 데이터가 경로 참조인지 리터럴인지에 따라 로컬 모델 패치 또는 서버 액션 중 한 경로로 분기한다. 변경 감지는 OnPush 전략으로 최적화된다.
