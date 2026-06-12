# renderers/react/src/v0_9/catalog/basic/context/MarkdownContext.tsx

## 개요

Markdown 렌더러 함수를 React Context로 공급하기 위한 파일이다. `MarkdownRenderer` 타입의 Context 객체와 이를 소비하는 커스텀 hook을 정의하며, `useMarkdown` hook이 실제 렌더러 인스턴스에 접근하는 경로를 제공한다. 기본값은 `undefined`이며, 렌더러가 주입되지 않으면 Markdown 기능이 비활성화된다.

## 의존성

### 외부 패키지
- `react` — `createContext`, `useContext`
- `@a2ui/web_core/types/types` — `MarkdownRenderer` (타입 import)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|------|------|
| `MarkdownContext` | 상수 (`React.Context<MarkdownRenderer \| undefined>`) |
| `useMarkdownRenderer` | 함수 (React hook) |

## 상세 명세

### `MarkdownContext`

`createContext<MarkdownRenderer | undefined>(undefined)`

`MarkdownRenderer | undefined` 타입을 갖는 React Context. 초기 기본값은 `undefined`. `MarkdownRenderer`는 `@a2ui/web_core/types/types`에서 import한 타입으로, 마크다운 문자열을 받아 HTML 문자열 Promise를 반환하는 비동기 함수 형태다.

### `useMarkdownRenderer`

**시그니처:** `() => MarkdownRenderer | undefined`

`useContext(MarkdownContext)`를 호출하여 현재 Context 트리에서 가장 가까운 `MarkdownContext.Provider`의 값을 반환하는 hook이다. Provider가 없으면 기본값 `undefined`를 반환한다.

## 동작 흐름

외부에서 `<MarkdownContext.Provider value={someRenderer}>` 형태로 렌더러를 공급하면, 트리 하위의 컴포넌트가 `useMarkdownRenderer()`를 통해 해당 렌더러에 접근할 수 있다. `useMarkdown` hook이 이 hook을 호출하여 렌더러의 유무를 판단하고 동작을 분기한다.
