# renderers/react/src/v0_8/styles/reset.ts

## 개요

React Light DOM 환경에서 호스트 앱의 CSS 리셋(예: Tailwind preflight, normalize.css)이 A2UI 컴포넌트 내부 요소에 영향을 미치는 문제를 해결하는 CSS 문자열 상수를 내보내는 파일이다. `.a2ui-surface` 내부의 모든 비-SVG 요소에 `all: revert`를 적용하여 브라우저 기본 스타일을 복원한다. `@layer`를 사용하여 다른 모든 A2UI 스타일보다 우선순위가 낮게 설계되어 있다.

## 의존성

없음. 순수한 문자열 상수만 정의한다.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `resetStyles` | 상수 (`string`) | `@layer a2ui-reset` 규칙이 담긴 CSS 문자열 |

## 상세 명세

### `resetStyles: string`

아래 구조의 CSS를 담은 템플릿 리터럴 문자열이다:

```
@layer a2ui-reset {
  :where(.a2ui-surface) :where(*:not(svg, svg *:not(foreignObject *))) {
    all: revert;
  }
}
```

- `@layer a2ui-reset`: 레이어드 스타일은 author 우선순위 계층에서 가장 낮으므로 A2UI 유틸리티 클래스, 컴포넌트 스타일, 테마 클래스, 인라인 스타일이 자동으로 이 reset을 덮어쓴다.
- `:where(.a2ui-surface)`: 선택자 명시도(specificity)를 0으로 유지하기 위해 `:where()` 사용.
- `:where(*:not(svg, svg *:not(foreignObject *)))`: SVG 요소와 SVG 내부 요소(`foreignObject` 안 제외)에는 `all: revert`를 적용하지 않아 SVG 렌더링을 보호한다.
- `all: revert`: 해당 요소의 모든 CSS 속성을 브라우저(user-agent) 기본값으로 되돌린다.

## 동작 흐름

이 파일은 단순히 상수를 선언하고 내보낸다. 실제 동작은 [`../styles/index.ts`](./index.ts.md)의 `injectStyles()` 함수가 이 문자열을 `<style>` 태그의 맨 앞에 포함시킬 때 발생한다. 삽입 이후 `.a2ui-surface` 내 모든 요소는 호스트 CSS 리셋의 영향을 받지 않고 브라우저 기본 스타일 위에서 A2UI 전용 스타일만 적용된다.
