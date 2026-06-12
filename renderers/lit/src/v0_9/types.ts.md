# renderers/lit/src/v0_9/types.ts

## 개요

Lit 렌더러 전용 컴포넌트 API 타입을 정의하는 모듈이다. 프레임워크 중립적인 `ComponentApi`를 확장하여 Lit 커스텀 엘리먼트에서 필수적으로 요구되는 `tagName` 속성을 추가한다. 이 인터페이스는 `A2uiNode`가 동적으로 적절한 커스텀 엘리먼트를 렌더링할 때 사용되며, 사용자 정의 `Catalog`의 타입 파라미터로도 사용된다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9` — `ComponentApi` 인터페이스

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|------|------|
| `LitComponentApi` | 인터페이스 |

## 상세 명세

### `LitComponentApi` (인터페이스)

- 상속: `ComponentApi` (`@a2ui/web_core/v0_9`)
- 추가 필드:
  - `tagName: string` — Lit 커스텀 엘리먼트의 HTML 태그 이름 (예: `'a2ui-basic-text'`). `customElements.define()`에 등록된 이름과 일치해야 한다.
- 용도: `Catalog<LitComponentApi>` 형태로 사용하여 카탈로그 내의 모든 컴포넌트 항목이 반드시 `tagName`을 갖도록 강제한다.

## 동작 흐름

이 파일은 타입 정의만 포함하며 런타임 코드는 없다. `ComponentApi`의 계약을 그대로 유지하면서 Lit 렌더러가 필요로 하는 `tagName` 속성 하나만 추가한다.
