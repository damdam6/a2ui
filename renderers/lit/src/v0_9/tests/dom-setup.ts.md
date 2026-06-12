# renderers/lit/src/v0_9/tests/dom-setup.ts

## 개요

Lit 기반 컴포넌트를 Node.js 테스트 환경에서 실행할 수 있도록 JSDOM 인스턴스를 생성하고 브라우저 전역(global) 객체를 주입하는 테스트 헬퍼 모듈이다. LitElement는 클래스 정의 시점에 `window`, `document`, `HTMLElement`, `customElements` 등의 DOM API가 전역 스코프에 존재해야 하므로, 이 모듈이 테스트 파일의 `before` 훅에서 반드시 먼저 호출되어야 한다. 또한 Lit 업데이트 사이클을 기다리는 편의 함수 `asyncUpdate`를 제공한다.

## 의존성

### 외부 패키지
- `jsdom` — `JSDOM` 클래스 (브라우저 DOM 환경 시뮬레이션)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|------|------|
| `setupTestDom` | 함수 |
| `teardownTestDom` | 함수 |
| `asyncUpdate` | 함수 (제네릭) |

## 상세 명세

### 모듈 수준 변수

- `dom: JSDOM | null` — 싱글턴 JSDOM 인스턴스. 모듈 내부에서만 관리된다. 초기값은 `null`.
- `originalGlobals: Record<string, any>` — 전역 오염을 되돌리기 위해 원래 Node.js 전역 값들을 보존하는 딕셔너리. 초기값은 빈 객체 `{}`.

### `applyGlobals(obj: Record<string, any>): void` (비공개 헬퍼)

- 매개변수: `obj` — 전역에 적용할 키-값 쌍의 객체
- 반환 타입: `void`
- 동작: `obj`의 각 항목을 순회하면서 값이 `undefined`이면 해당 키를 `global`에서 삭제하고, 그렇지 않으면 `(global as any)[key] = value`로 설정한다. 이를 통해 전역 적용과 복원 모두 동일한 함수로 처리한다.

### `setupTestDom(): void`

- 매개변수: 없음
- 반환 타입: `void`
- 동작 로직:
  1. `dom`이 `null`이면(최초 호출 시) `new JSDOM('<!DOCTYPE html><body></body>')`로 인스턴스를 생성한다.
  2. 관리할 전역 이름 목록(`window`, `document`, `HTMLElement`, `customElements`, `Element`, `Node`, `Event`, `MutationObserver`, `requestAnimationFrame`, `cancelAnimationFrame`, `CSSStyleSheet`)에 대해 현재 `global` 값을 `originalGlobals`에 한 번만 저장한다.
  3. `dom`이 이미 존재하면(재사용 시) `dom.window.document.body.innerHTML = ''`으로 body를 초기화하여 테스트 간 DOM 누수를 방지한다.
  4. `dom.window.document.adoptedStyleSheets`가 없으면 빈 배열 `[]`로 초기화한다(JSDOM v29 미만 호환성 패치).
  5. `dom.window.CSSStyleSheet`가 존재하고 `prototype.replaceSync`가 없으면 폴리필을 주입한다. 이 폴리필은 `_styleEl` 속성에 `<style>` 요소를 캐싱하고, 호출 시 `textContent`를 갱신하는 방식으로 동작한다.
  6. 최종적으로 `applyGlobals`를 호출하여 JSDOM의 `window`, `document`, `HTMLElement`, `customElements`, `Element`, `Node`, `Event`, `MutationObserver`, `CSSStyleSheet`와, `requestAnimationFrame`은 `setTimeout(cb, 16)`으로, `cancelAnimationFrame`은 `clearTimeout`으로 전역에 설정한다.

### `teardownTestDom(): void`

- 매개변수: 없음
- 반환 타입: `void`
- 동작 로직:
  1. `dom`이 존재하면 `dom.window.document.body.innerHTML = ''`으로 정리한 뒤 `dom = null`로 해제한다.
  2. `applyGlobals(originalGlobals)`를 호출하여 원래 Node.js 전역 상태를 복원한다.

### `asyncUpdate<T = any>(target: T, updateFn: (el: T) => void | Promise<void>): Promise<void>`

- 제네릭: `T` — 기본값 `any`
- 매개변수:
  - `target: T` — 업데이트 대상 요소(LitElement 또는 임의 객체)
  - `updateFn: (el: T) => void | Promise<void>` — 상태 변경을 수행하는 콜백
- 반환 타입: `Promise<void>`
- 동작 로직:
  1. `await updateFn(target)`을 호출하여 상태 변경을 적용한다.
  2. `(target as any).updateComplete`가 존재하면 이를 `await`하여 LitElement의 업데이트 사이클 완료를 기다린다.
  3. 그렇지 않으면 `await new Promise(r => setTimeout(r, 0))`으로 매크로태스크 큐를 양보하여, `updateComplete`가 없는 모의 객체에서도 비동기 처리가 완료될 수 있도록 한다.

## 동작 흐름

모듈 수준에서 `dom`과 `originalGlobals`를 유지하는 싱글턴 패턴을 사용한다. 테스트 스위트의 `before` 훅에서 `setupTestDom()`을 호출하면 JSDOM이 생성되고 전역이 오염된다. 각 테스트 파일의 `after` 훅에서 `teardownTestDom()`을 호출하면 전역이 복원된다. `asyncUpdate`는 LitElement 또는 일반 객체에 변경을 가하고 그 결과 렌더링이 완료될 때까지 기다리는 단일 진입점이다.
