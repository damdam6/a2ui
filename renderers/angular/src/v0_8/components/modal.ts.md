# renderers/angular/src/v0_8/components/modal.ts

## 개요

`Modal` 컴포넌트는 두 개의 자식 노드(진입점 자식과 컨텐츠 자식)를 받아, 진입점을 클릭하면 오버레이 모달을 열고 닫는 UI를 제공하는 Angular 독립형 컴포넌트다. `DynamicComponent<ModalNode>`를 상속하여 a2ui 렌더링 파이프라인에 통합된다. 모달의 열림/닫힘 상태는 내부 `signal`로 관리되며, 외부 상태 관리 없이 자체적으로 동작한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `input`, `signal`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브 (자식 컴포넌트 동적 렌더링)
- [`../types`](../types.ts.md) — `AnyComponentNode`, `ModalNode` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `Modal` | 클래스 (Angular Component) |

## 상세 명세

### `Modal` 클래스

`@Component` 데코레이터 설정:
- `selector`: `'a2ui-modal'`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

`DynamicComponent<ModalNode>`를 상속한다. `theme` 객체는 부모 클래스에서 주입되며 `theme.components.Modal.backdrop`과 `theme.components.Modal.element` 클래스를 오버레이/컨텐츠 div에 바인딩한다.

#### 필드

| 이름 | 타입 | 설명 |
|------|------|------|
| `entryPointChild` | `InputSignal<AnyComponentNode>` (required) | 클릭 가능한 진입점으로 렌더링할 자식 컴포넌트 노드 |
| `contentChild` | `InputSignal<AnyComponentNode>` (required) | 모달 내부에 렌더링할 자식 컴포넌트 노드 |
| `isOpen` | `WritableSignal<boolean>` (protected) | 모달 열림 상태, 초기값 `false` |

#### `openModal(): void`
`isOpen` 시그널을 `true`로 설정한다. 진입점 div의 `(click)` 이벤트 핸들러로 바인딩된다.

#### `closeModal(): void`
`isOpen` 시그널을 `false`로 설정한다. 오버레이 배경 클릭 및 닫기 버튼(`&times;`)의 `(click)` 이벤트 핸들러로 바인딩된다.

#### 템플릿 구조
1. `.a2ui-modal-entry-point` div: `(click)="openModal()"` 바인딩. `entryPointChild()`가 존재하면 `a2ui-renderer`를 통해 동적으로 렌더링한다.
2. `@if (isOpen())` 블록: 모달이 열렸을 때만 렌더링.
   - `.a2ui-modal-overlay` div: 전체 화면 고정 오버레이. `(click)="closeModal()"`. `theme.components.Modal.backdrop` 클래스 적용.
   - `.a2ui-modal-content` div: 실제 모달 컨텐츠 영역. `(click)="$event.stopPropagation()"` 으로 이벤트 버블링 차단. `theme.components.Modal.element` 클래스 적용. `contentChild()`를 `a2ui-renderer`로 동적 렌더링.
   - `.a2ui-modal-close` 버튼: `&times;` 텍스트, `(click)="closeModal()"`.

#### 스타일 (컴포넌트 스코프)
- `:host`: `display: inline-block`
- `.a2ui-modal-overlay`: `position: fixed`, 전체 뷰포트 크기, `background: var(--a2ui-modal-backdrop-bg, rgba(0,0,0,0.5))`, flexbox 중앙 정렬, `z-index: 1000`
- `.a2ui-modal-content`: `background: var(--a2ui-modal-background, var(--a2ui-color-surface, white))`, `padding: var(--a2ui-modal-padding, var(--a2ui-spacing-xl, 32px))`, `border-radius: var(--a2ui-modal-border-radius, var(--a2ui-border-radius, 8px))`, `min-width: 300px`, `max-width: 80%`, `max-height: 80%`, `overflow-y: auto`, `box-shadow: var(--a2ui-modal-box-shadow, 0 10px 25px rgba(0,0,0,0.2))`
- `.a2ui-modal-close`: absolute 우상단 배치(`top:10px`, `right:15px`), 배경/테두리 없음, `font-size: 24px`, 기본색 `var(--a2ui-text-caption-color, #999)`, hover 시 `var(--a2ui-text-color, #333)`

## 동작 흐름

1. 컴포넌트 초기화 시 `isOpen`은 `false`. 화면에는 진입점만 표시.
2. 사용자가 진입점 영역을 클릭 → `openModal()` 호출 → `isOpen = true` → 오버레이와 컨텐츠가 렌더링됨.
3. 사용자가 오버레이 배경 또는 닫기 버튼 클릭 → `closeModal()` 호출 → `isOpen = false` → 오버레이 제거.
4. 컨텐츠 영역 클릭 시 이벤트 전파를 중단하여 배경 클릭에 의한 닫힘 방지.
5. `entryPointChild`와 `contentChild` 모두 `Renderer` 디렉티브를 통해 동적으로 렌더링되므로 임의의 a2ui 컴포넌트 노드를 수용한다.
