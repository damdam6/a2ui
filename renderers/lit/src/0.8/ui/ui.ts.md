# renderers/lit/src/0.8/ui/ui.ts

## 개요

이 파일은 A2UI v0.8 렌더러의 공개 UI 컴포넌트 진입점(barrel)이다. 20개의 커스텀 엘리먼트 클래스(`Audio`, `Button`, `Card` 등)를 import하여 그대로 re-export하고, 전역 `HTMLElementTagNameMap`을 확장해 타입 안전한 태그명 조회를 지원한다. `Context`, `Utils`, `ComponentRegistry`, `registerCustomComponents`를 네임스페이스 또는 이름 있는 export로 제공하며, 태그명으로 커스텀 엘리먼트 인스턴스를 생성하는 `instanceOf` 헬퍼 함수를 내보낸다.

## 의존성

### 저장소 내부 모듈

- [`./audio.js`](./audio.ts.md)
- [`./button.js`](./button.ts.md)
- [`./card.js`](./card.ts.md)
- [`./checkbox.js`](./checkbox.ts.md)
- [`./column.js`](./column.ts.md)
- [`./datetime-input.js`](./datetime-input.ts.md)
- [`./divider.js`](./divider.ts.md)
- [`./icon.js`](./icon.ts.md)
- [`./image.js`](./image.ts.md)
- [`./list.js`](./list.ts.md)
- [`./multiple-choice.js`](./multiple-choice.ts.md)
- [`./modal.js`](./modal.ts.md)
- [`./root.js`](./root.ts.md)
- [`./row.js`](./row.ts.md)
- [`./slider.js`](./slider.ts.md)
- [`./surface.js`](./surface.ts.md)
- [`./tabs.js`](./tabs.ts.md)
- [`./text-field.js`](./text-field.ts.md)
- [`./text.js`](./text.ts.md)
- [`./video.js`](./video.ts.md)
- [`./context/context.js`](./context/context.ts.md) — `Context` 네임스페이스 re-export
- [`./utils/utils.js`](./utils/utils.ts.md) — `Utils` 네임스페이스 re-export
- [`./component-registry.js`](./component-registry.ts.md) — `ComponentRegistry`, `componentRegistry`
- [`./custom-components/index.js`](./custom-components/index.ts.md) — `registerCustomComponents`

### 외부 패키지

없음

## Exports

| 이름 | 종류 |
|---|---|
| `TagName` | 타입 (`keyof A2UITagNameMap`) |
| `CustomElementConstructorOf<T>` | 제네릭 타입 |
| `Context` | 네임스페이스 re-export |
| `Utils` | 네임스페이스 re-export |
| `ComponentRegistry` | 클래스 re-export |
| `componentRegistry` | 상수 re-export |
| `registerCustomComponents` | 함수 re-export |
| `Audio`, `Button`, `Card`, `Column`, `Checkbox`, `DateTimeInput`, `Divider`, `Icon`, `Image`, `List`, `MultipleChoice`, `Modal`, `Row`, `Slider`, `Root`, `Surface`, `Tabs`, `Text`, `TextField`, `Video` | 클래스 re-export |
| `instanceOf<T>` | 함수 |

## 상세 명세

### 타입: `TagName`

`type TagName = keyof A2UITagNameMap` — `A2UITagNameMap`의 모든 키를 유니온 타입으로 묶은 문자열 리터럴 타입이다. 유효한 A2UI 태그명만 허용한다.

### 타입: `CustomElementConstructorOf<T extends HTMLElement>`

`{ new(): T } & typeof HTMLElement` 형태의 교차 타입이다. `new()` 시그니처를 포함해 인스턴스화 가능성을 보장하면서, 동시에 `HTMLElement` 정적 멤버도 접근 가능하도록 한다.

### 인터페이스: `A2UITagNameMap` (파일 내부 비공개)

태그명 문자열 리터럴을 대응하는 커스텀 엘리먼트 클래스 타입에 매핑하는 인터페이스. 총 20개의 항목을 포함한다:

| 태그명 | 타입 |
|---|---|
| `'a2ui-audioplayer'` | `Audio` |
| `'a2ui-button'` | `Button` |
| `'a2ui-card'` | `Card` |
| `'a2ui-checkbox'` | `Checkbox` |
| `'a2ui-column'` | `Column` |
| `'a2ui-datetimeinput'` | `DateTimeInput` |
| `'a2ui-divider'` | `Divider` |
| `'a2ui-icon'` | `Icon` |
| `'a2ui-image'` | `Image` |
| `'a2ui-list'` | `List` |
| `'a2ui-modal'` | `Modal` |
| `'a2ui-multiplechoice'` | `MultipleChoice` |
| `'a2ui-root'` | `Root` |
| `'a2ui-row'` | `Row` |
| `'a2ui-slider'` | `Slider` |
| `'a2ui-surface'` | `Surface` |
| `'a2ui-tabs'` | `Tabs` |
| `'a2ui-text'` | `Text` |
| `'a2ui-textfield'` | `TextField` |
| `'a2ui-video'` | `Video` |

파일 끝 `declare global` 블록에서 `interface HTMLElementTagNameMap extends A2UITagNameMap {}` 선언으로 전역 DOM 타입을 확장한다. 이를 통해 `document.querySelector`, `document.createElement` 등에서도 A2UI 태그명이 올바른 타입으로 인식된다.

### 함수: `instanceOf<T extends keyof A2UITagNameMap>`

- **시그니처:** `instanceOf<T extends keyof A2UITagNameMap>(tagName: T): A2UITagNameMap[T] | undefined`
- **매개변수:** `tagName: T` — `A2UITagNameMap`에 정의된 태그명 문자열
- **반환 타입:** `A2UITagNameMap[T]` 인스턴스, 또는 등록이 없을 경우 `undefined`
- **동작:**
  1. `customElements.get(tagName)` 결과를 `CustomElementConstructorOf<A2UITagNameMap[T]> | undefined`로 타입 단언한다.
  2. 생성자가 `undefined`이면 `console.warn('No element definition for', tagName)` 경고를 출력하고 `undefined`를 반환한다.
  3. 생성자가 있으면 `new ctor()`으로 인스턴스를 생성하여 반환한다.

## 동작 흐름

파일 로드 시 20개 컴포넌트 모듈이 import되며, 각 모듈의 `@customElement` 데코레이터 실행으로 커스텀 엘리먼트가 브라우저 레지스트리에 등록된다. 소비자는 이 파일 하나만 import하면 모든 A2UI v0.8 컴포넌트와 유틸리티에 접근할 수 있다. `instanceOf`는 런타임 레지스트리 조회를 통해 타입 안전한 인스턴스 생성 팩토리 역할을 한다.
