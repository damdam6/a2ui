# renderers/lit/src/0.8/ui/context/theme.ts

## 개요

`theme.ts`는 A2UI의 테마 객체를 Lit 컨텍스트로 제공하기 위한 컨텍스트 심볼 정의 파일이다. `@lit/context`의 `createContext`를 사용해 `Types.Theme | undefined` 타입의 컨텍스트를 생성한다. 하위 호환성을 위해 `theme`의 별칭인 `themeContext`도 함께 내보낸다.

## 의존성

### 외부 패키지
- `@lit/context` — `createContext`
- `@a2ui/web_core/types/types` — `Types` (네임스페이스)

### 저장소 내부 모듈
없음.

## Exports

- `theme` (상수, `Context<Types.Theme | undefined>`)
- `themeContext` (상수, `theme`의 별칭, `@deprecated`)

## 상세 명세

### 상수 `theme`

- 타입: `Context<Types.Theme | undefined>`
- 값: `createContext<Types.Theme | undefined>(Symbol('A2UITheme'))`
- 고유 식별자로 `Symbol('A2UITheme')`을 사용한다.
- `Root` 클래스에서 `@consume({context: themeContext})`를 통해 소비된다.
- 제공자(예: `Surface` 컴포넌트)가 `Theme` 객체를 컨텍스트로 제공하면, 하위 컴포넌트 트리 전체가 동일한 테마를 공유한다.

### 상수 `themeContext`

- 타입: `Context<Types.Theme | undefined>` (`theme`과 동일)
- 값: `theme` (단순 참조 별칭, `export const themeContext = theme`)
- `@deprecated` — 신규 코드에서는 `theme`을 사용해야 한다.
- 이전 버전 코드와의 호환성 유지 목적으로만 존재한다.

## 동작 흐름

모듈 로드 시 `Symbol('A2UITheme')`을 키로 하는 컨텍스트가 한 번 생성된다. `Surface` 등 최상위 컴포넌트가 `@provide({context: theme})`으로 `Theme` 객체를 주입하면, 자식 컴포넌트들(`Root` 및 모든 파생 클래스)은 `@consume`을 통해 `this.theme`으로 테마 데이터에 접근해 CSS 클래스 매핑과 추가 스타일을 적용한다.
