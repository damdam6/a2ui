# renderers/react/src/v0_8/styles/index.ts

## 개요

A2UI React 렌더러의 전역 CSS 스타일을 정의하고 주입하는 모듈이다. Lit 렌더러가 Shadow DOM으로 격리하는 스타일을 Light DOM 환경인 React에서도 동작하도록 변환하여 제공한다. 세 종류의 스타일(reset, structural, component-specific)을 합산하여 `<style>` 태그를 `document.head`에 삽입하거나 제거하는 함수를 내보낸다.

## 의존성

### 외부 패키지

- `@a2ui/web_core/styles/index` — Lit 렌더러의 구조적 CSS 유틸리티 클래스 모음(`structuralStyles`). `:host` 선택자 기반으로 작성되어 있어 변환이 필요하다.

### 저장소 내부 모듈

- [`./reset`](./reset.ts.md) — `.a2ui-surface` 내부 요소의 브라우저 기본 스타일을 복원하는 CSS 문자열(`resetStyles`)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `structuralStyles` | 상수 (`string`) | `:host` → `.a2ui-surface`로 변환된 구조적 유틸리티 CSS |
| `componentSpecificStyles` | 상수 (`string`) | 각 A2UI 컴포넌트별 Light DOM 전용 CSS 규칙 문자열 |
| `injectStyles` | 함수 | 세 스타일을 합쳐 `document.head`에 `<style id="a2ui-structural-styles">` 삽입 |
| `removeStyles` | 함수 | 삽입된 `<style>` 태그를 `document.head`에서 제거 |

## 상세 명세

### `structuralStyles: string`

`@a2ui/web_core/styles/index`에서 가져온 `Styles.structuralStyles`에 대해 정규식 `/:host\s*\{/g`를 적용하여 `.a2ui-surface {`로 치환한 결과 문자열이다. 이로써 Lit Shadow DOM 전용 `:host` 선택자를 React Light DOM에서 사용 가능한 클래스 선택자로 변환한다.

### `componentSpecificStyles: string`

각 컴포넌트 클래스(`.a2ui-card`, `.a2ui-divider`, `.a2ui-text`, `.a2ui-textfield`, `.a2ui-checkbox`, `.a2ui-slider`, `.a2ui-button`, `.a2ui-icon`, `.a2ui-tabs`, `.a2ui-modal`, `.a2ui-image`, `.a2ui-video`, `.a2ui-audio`, `.a2ui-multiplechoice`, `.a2ui-column`, `.a2ui-row`, `.a2ui-list`, `.a2ui-datetime-input`)에 대한 글로벌 CSS를 담은 템플릿 리터럴 문자열이다.

변환 규칙:
- Lit의 `:host` → `.a2ui-surface .a2ui-{component}`
- Lit의 자식 요소 선택자(`section`, `input` 등) → `.a2ui-surface .a2ui-{component} > section` 형태로 `>` 자식 결합자 사용
- Lit의 `::slotted(*)` → `.a2ui-surface .a2ui-{component} > section > *`
- 테마 유틸리티 클래스에 의해 덮어쓰여야 하는 요소 선택자는 `:where()`로 감싸 명시도(specificity)를 0으로 유지

주요 값들:
- `.a2ui-divider hr`: `height: 1px`, `background: #ccc`, `border: none`
- `.a2ui-icon .g-icon`: `font-size: 24px`
- `.a2ui-modal dialog`: `padding: 0`, `border: none`, `background: none`
- `.a2ui-modal dialog section #controls`: `display: flex`, `justify-content: end`
- `.a2ui-modal dialog section #controls button`: `width: 20px`, `height: 20px`
- `.a2ui-datetime-input input`: `border-radius: 8px`, `padding: 8px`, `border: 1px solid #ccc`
- `.a2ui-image img`: `object-fit: var(--object-fit, fill)`
- `.a2ui-list[data-direction="horizontal"]` 자식: `flex: 1 0 fit-content`, `max-width: min(80%, 400px)`
- `.a2ui-surface *, *::before, *::after`: `box-sizing: border-box`
- Column/Row: `data-alignment`, `data-distribution` 속성으로 `align-items`/`justify-content` 변형

### `injectStyles(): void`

**매개변수:** 없음.  
**반환 타입:** `void`

1. `typeof document === 'undefined'` 확인 — SSR 환경이면 즉시 반환.
2. `document.getElementById('a2ui-structural-styles')` 확인 — 이미 삽입된 경우 중복 삽입을 방지하고 즉시 반환.
3. `document.createElement('style')`로 새 요소를 생성하고 `id`를 `'a2ui-structural-styles'`로 설정.
4. `styleElement.textContent`에 `resetStyles + '\n' + structuralStyles + '\n' + componentSpecificStyles`를 할당.
5. `document.head.appendChild(styleElement)`로 삽입.

CSS 변수(팔레트 `--n-*`, `--p-*` 등)는 호스트 앱이 `:root`에 정의해야 하며 이 함수는 주입하지 않는다.

### `removeStyles(): void`

**매개변수:** 없음.  
**반환 타입:** `void`

1. `typeof document === 'undefined'` 확인 — SSR이면 즉시 반환.
2. `document.getElementById('a2ui-structural-styles')`로 요소를 찾아 존재할 경우 `.remove()` 호출.

테스트나 컴포넌트 언마운트 시 정리용으로 설계되었다.

## 동작 흐름

모듈 로드 시 `structuralStyles`와 `componentSpecificStyles` 두 상수가 즉시 계산된다. 앱 시작 시 `injectStyles()`를 한 번 호출하면 reset → structural → component 순으로 결합된 CSS가 문서에 적용된다. 이후 A2UI 컴포넌트들은 별도의 Shadow DOM 없이 Light DOM에서 격리된 스타일을 받을 수 있다.
