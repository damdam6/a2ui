# renderers/angular/src/v0_8/components/checkbox.ts

## 개요

a2ui v0.8 명세의 `CheckBox` UI 컴포넌트를 구현하는 Angular 스탠드얼론 컴포넌트 파일이다. `DynamicComponent`를 상속하며, 레이블과 체크 상태를 가진 HTML 체크박스를 렌더링한다. 체크 상태가 데이터 경로(`path`)를 참조할 경우 로컬 데이터 모델을 직접 업데이트하고, 그렇지 않으면 `'toggle'` 액션을 서버로 전송한다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- `../types` (타입 전용) — `BooleanValue`, `CheckboxNode`, `StringValue`

## Exports

| 이름 | 종류 |
|------|------|
| `Checkbox` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### 클래스: `Checkbox`
- 데코레이터: `@Component`
- 셀렉터: `a2ui-checkbox`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- 상속: `DynamicComponent<CheckboxNode>`
- 스탠드얼론 컴포넌트 (별도 `imports` 없음)

#### 템플릿
`<label>` 요소 내부에 `<input type="checkbox">`와 텍스트를 함께 렌더링한다. `input`에는 `[id]="inputId"`, `[checked]="inputChecked()"`, `(change)="onToggle($event)"`가 바인딩된다. `<label>` 내부 텍스트는 `{{ resolvedLabel() }}`로 보간된다.

#### 스타일
- `:host` — `display: flex; align-items: center; gap: 0.5rem`

#### 입력 (Signal input)

| 이름 | 타입 | 필수 여부 |
|------|------|-----------|
| `label` | `StringValue \| null` | 필수 (`input.required`) |
| `value` | `BooleanValue \| null` | 필수 (`input.required`) |

#### 계산 신호

| 이름 | 접근 제어 | 설명 |
|------|-----------|------|
| `inputChecked` | `protected` | `super.resolvePrimitive(this.value())`를 호출하여 `BooleanValue`를 `boolean`으로 해석한다. 해석 결과가 `null`이면 `false`를 기본값으로 반환한다 (`?? false`). |
| `resolvedLabel` | `protected` | `super.resolvePrimitive(this.label())`를 호출하여 `StringValue`를 문자열로 해석한다. |

#### 필드

| 이름 | 접근 제어 | 설명 |
|------|-----------|------|
| `inputId` | `protected readonly` | `super.getUniqueId('a2ui-checkbox')` 호출로 생성된 전역 고유 ID. `<input id>` 속성에 사용된다. |

#### 메서드: `onToggle(event: Event): void`
- 접근 제어: public
- 매개변수: `event: Event` — 체크박스 `change` 이벤트
- 반환 타입: `void`
- 동작:
  1. `(event.target as HTMLInputElement).checked`로 새로운 체크 상태(`checked: boolean`)를 읽는다.
  2. `this.value()` 신호로 현재 `BooleanValue` 노드(`checkedNode`)를 가져온다.
  3. `checkedNode`가 객체이고 `'path'` 키를 가지며 `path` 값이 truthy이면 — 데이터 경로를 참조하는 경우 — 직접 데이터 모델 업데이트 경로로 진입한다. `this.processor.processMessages`를 호출하여 `dataModelUpdate` 메시지를 전송하며, `path`는 `this.processor.resolvePath(checkedNode.path, this.component().dataContextPath)`로 해석하고, `contents`는 `[{key: '.', valueBoolean: checked}]`로 설정한다.
  4. 경로가 없는 경우(리터럴 값이거나 `checkedNode`가 null인 경우) `sendAction`을 호출한다. 액션명은 `'toggle'`이고, context는 `[{key: 'checked', value: {literalBoolean: checked}}]`다.

## 동작 흐름

컴포넌트 초기화 시 `label`과 `value` 신호에서 `resolvedLabel`과 `inputChecked` 계산 신호가 파생된다. `inputId`는 컴포넌트 생성 시 한 번만 생성된다. 사용자가 체크박스를 클릭하면 `change` 이벤트가 발생하고 `onToggle`이 호출된다. 값이 데이터 경로를 참조하면 즉각적인 UI 피드백을 위해 로컬 데이터 모델을 직접 수정하고, 리터럴 값이면 서버로 `toggle` 액션을 전송한다.
