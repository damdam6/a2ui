# renderers/react/src/v0_9/catalog/basic/hooks/useMarkdown.ts

## 개요

마크다운 텍스트를 HTML 문자열로 비동기 변환하는 React hook이다. `MarkdownContext`에서 렌더러를 가져와 텍스트를 변환하고, 결과를 로컬 state에 저장한다. 렌더러가 없으면 경고 메시지를 한 번만 출력하고 `null`을 반환한다. 컴포넌트 언마운트나 의존성 변경 시 경쟁 조건(race condition)을 방지하는 `active` 플래그를 사용한다.

## 의존성

### 외부 패키지
- `react` — `useState`, `useEffect`
- `@a2ui/web_core/types/types` — `MarkdownRendererOptions` (타입 import)

### 저장소 내부 모듈
- [`../context/MarkdownContext`](../context/MarkdownContext.tsx.md) — `useMarkdownRenderer`

## Exports

| 이름 | 종류 |
|------|------|
| `useMarkdown` | 함수 (React hook) |

## 상세 명세

### `warningLogged` (모듈 레벨 변수)

`let warningLogged = false`

렌더러 미설정 경고 메시지가 이미 출력되었는지를 추적하는 모듈 스코프 플래그다. 경고가 여러 번 중복 출력되지 않도록 한다. 모듈 생애 주기 동안 영구적으로 유지된다.

### `useMarkdown`

**시그니처:** `function useMarkdown(text: string, options?: MarkdownRendererOptions): string | null`

**매개변수:**
- `text: string` — 변환할 마크다운 원본 텍스트
- `options?: MarkdownRendererOptions` — 렌더러에 전달할 추가 옵션 (옵셔널)

**반환값:** 변환된 HTML 문자열 또는 `null` (렌더러 없음 또는 대기 중)

**내부 상태:**
- `html: string | null` — 초기값 `null`. 변환 결과 HTML이 여기 저장된다.

**동작 로직:**

1. `useMarkdownRenderer()`로 Context에서 렌더러 함수를 가져온다.
2. `optionsKey = JSON.stringify(options)` — `options` 객체를 직렬화하여 `useEffect` 의존성 배열에 안전하게 포함시킨다.
3. `useEffect`는 `[text, renderer, optionsKey]`를 의존성으로 가진다.

**useEffect 내부:**
- `renderer`가 없으면:
  - `warningLogged`가 `false`일 경우, `console.warn('[useMarkdown]', ...)` 으로 렌더러 미설정 안내 메시지(`"can't render markdown because no markdown renderer is configured.\nUse \`@a2ui/markdown-it\`, or your own markdown renderer."`)를 출력하고 `warningLogged = true`로 설정한다.
  - `setHtml(null)` 후 `return`한다.
- `renderer`가 있으면:
  - `active = true` 플래그를 설정한다.
  - `parsedOptions = optionsKey ? JSON.parse(optionsKey) : undefined`로 options를 복원한다.
  - `renderer(text, parsedOptions)`를 호출하여 Promise를 얻는다.
  - `.then(result => { if (active) setHtml(result); })` — 컴포넌트가 여전히 활성 상태일 때만 state를 업데이트한다.
  - `.catch(err => { console.error('[useMarkdown] Render failed:', err); })` — 렌더링 실패 시 콘솔 오류 출력.
  - cleanup 함수: `() => { active = false; }` — effect 재실행 또는 언마운트 시 이전 Promise 결과를 무시한다.

4. 최종적으로 `html` state를 반환한다.

## 동작 흐름

hook 호출 → `useMarkdownRenderer()`로 렌더러 확인 → 렌더러 있으면 `renderer(text, options)` 비동기 호출 → 완료 후 `active` 확인 → `setHtml(result)` → 반환값이 `null`에서 HTML 문자열로 변경 → 이를 구독하는 컴포넌트 재렌더링. `text` 또는 `options`가 변경되면 effect가 재실행되고 이전 실행의 `active`가 `false`로 설정되어 오래된 결과가 적용되지 않는다.
