# renderers/angular/src/v0_9/catalog/basic/modal.component.ts

## 개요

A2UI v0.9 Modal 컴포넌트의 Angular 구현체다. 클릭 시 오버레이를 열어 콘텐츠를 표시하는 트리거 영역과, 전체 화면 백드롭 위에 렌더링되는 모달 콘텐츠 영역으로 구성된다. 열림/닫힘 상태를 내부 signal로 관리하며, 트리거와 콘텐츠 모두 `ComponentHostComponent`를 통해 동적으로 렌더링한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`, `signal`
- `@a2ui/web_core/v0_9/basic_catalog`: `ModalApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md): 동적 컴포넌트 렌더링에 사용
- [`./basic-catalog-component`](./basic-catalog-component.ts.md): 기반 추상 클래스

## Exports

- `ModalComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `ModalComponent`

**데코레이터**: `@Component`
- `selector`: `'a2ui-v09-modal'`
- `standalone: true`
- `imports`: `[ComponentHostComponent]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**상속**: `BasicCatalogComponent<typeof ModalApi>`

**필드**

| 이름 | 타입 | 설명 |
|------|------|------|
| `isOpen` | `WritableSignal<boolean>` | 모달의 열림/닫힘 상태. 초기값 `false` |
| `trigger` | `Signal<string \| undefined>` | `props()['trigger']?.value()`를 읽는 computed signal. 트리거 컴포넌트 키를 반환 |
| `content` | `Signal<string \| undefined>` | `props()['content']?.value()`를 읽는 computed signal. 콘텐츠 컴포넌트 키를 반환 |

**메서드**

#### `openModal(): void`
`isOpen` signal을 `true`로 설정한다.

#### `closeModal(): void`
`isOpen` signal을 `false`로 설정한다.

**템플릿 구조**

1. 최상위 `div.a2ui-modal-wrapper`(인라인 블록) 안에 `div.a2ui-modal-trigger`를 배치한다.
2. `trigger()` 값이 존재하면 `<a2ui-v09-component-host>`를 렌더링한다. 트리거 영역 클릭 시 `openModal()`이 호출된다.
3. `isOpen()`이 `true`이면 `div.a2ui-modal-overlay`(fixed 전체 화면 백드롭)를 렌더링한다. 오버레이 클릭 시 `closeModal()`이 호출된다.
4. 오버레이 내부 `div.a2ui-modal-content` 클릭 이벤트는 `$event.stopPropagation()`으로 버블링을 막는다.
5. 모달 콘텐츠 영역 우상단에 `button.a2ui-modal-close`(`&times;` 문자)가 있으며 클릭 시 `closeModal()`을 호출한다.
6. `content()` 값이 존재하면 내부에 `<a2ui-v09-component-host>`를 렌더링한다.

**스타일 (CSS 변수 지원)**

| CSS 변수 | 기본값 | 역할 |
|----------|--------|------|
| `--a2ui-modal-backdrop-bg` | `rgba(0,0,0,0.5)` | 백드롭 배경색 |
| `--a2ui-modal-background` | `var(--a2ui-color-surface, white)` | 모달 콘텐츠 배경색 |
| `--a2ui-modal-padding` | `var(--a2ui-spacing-xl, 32px)` | 모달 콘텐츠 패딩 |
| `--a2ui-modal-border-radius` | `var(--a2ui-border-radius, 8px)` | 모달 콘텐츠 테두리 반지름 |
| `--a2ui-modal-box-shadow` | `0 10px 25px rgba(0,0,0,0.2)` | 모달 박스 그림자 |

모달 콘텐츠는 `min-width: 300px`, `max-width: 80%`, `max-height: 80%`, `overflow-y: auto`이며 `z-index: 1000`의 fixed 오버레이 위에 중앙 정렬된다.

## 동작 흐름

초기 상태에서 `isOpen`은 `false`이므로 트리거 영역만 표시된다. 사용자가 트리거 영역을 클릭하면 `openModal()`이 호출되어 `isOpen`이 `true`가 되고 백드롭과 모달 콘텐츠가 DOM에 삽입된다. 닫기 버튼 클릭 또는 백드롭 클릭 시 `closeModal()`이 `isOpen`을 `false`로 되돌려 모달이 DOM에서 제거된다. `ChangeDetectionStrategy.OnPush`와 Angular signal을 사용하므로 상태 변경 시 변경 감지가 최소한으로 이루어진다.
