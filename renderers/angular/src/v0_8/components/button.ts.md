# renderers/angular/src/v0_8/components/button.ts

## 개요

a2ui v0.8 명세의 `Button` UI 컴포넌트를 구현하는 Angular 스탠드얼론 컴포넌트 파일이다. `DynamicComponent`를 상속하며, 자식 컴포넌트를 동적으로 렌더링하는 `<button>` 요소를 제공한다. 클릭 시 연결된 `Action`을 부모의 `sendAction`으로 전달하고, `action`이 `null`이면 아무 동작도 하지 않는다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `input`

### 저장소 내부 모듈
- `../types` (타입 전용) — `ButtonNode`, `Action`, `AnyComponentNode`
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브

## Exports

| 이름 | 종류 |
|------|------|
| `Button` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### 클래스: `Button`
- 데코레이터: `@Component`
- 셀렉터: `a2ui-button`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- 상속: `DynamicComponent<ButtonNode>`

#### 템플릿
`<button>` 요소에 `[class]="theme.components.Button"`과 `[style]="theme.additionalStyles?.Button"`을 바인딩하고, `(click)="handleClick()"` 이벤트를 연결한다. `child()` 신호가 truthy일 때만 `@if` 블록 내부에서 `a2ui-renderer` 디렉티브가 적용된 `<ng-container>`를 렌더링한다. `[component]` 바인딩은 `child() ?? component().properties.child`를 사용하여 입력 `child`가 없을 경우 컴포넌트 노드의 `properties.child`로 폴백한다.

#### 스타일
- `:host` — `display: block; flex: var(--weight); min-height: 0`

#### 입력 (Signal input)

| 이름 | 타입 | 기본값 |
|------|------|--------|
| `action` | `Action \| null` | `null` |
| `child` | `AnyComponentNode \| null` | `null` |
| `primary` | `boolean \| null` | `false` |

`primary`는 현재 템플릿에서 사용되지 않는다.

#### 메서드: `handleClick(): void`
- 접근 제어: `protected`
- 동작: `this.action()`으로 현재 액션 값을 조회한다. 값이 truthy이면 `super.sendAction(action)`을 호출하여 `A2UIClientEventMessage`를 프로세서로 디스패치한다. `action`이 `null`이면 아무 동작도 하지 않는다.

## 동작 흐름

컴포넌트가 초기화되면 `child`와 `action` 신호가 설정된다. `child()`가 truthy이면 템플릿이 `Renderer` 디렉티브를 통해 자식 컴포넌트를 동적으로 렌더링한다. 사용자가 버튼을 클릭하면 `handleClick`이 호출되고, `action`이 정의된 경우 `sendAction`을 통해 서버로 사용자 액션 이벤트가 전송된다.
