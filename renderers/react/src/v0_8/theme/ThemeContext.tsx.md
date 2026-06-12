# renderers/react/src/v0_8/theme/ThemeContext.tsx

## 개요

A2UI React 렌더러의 테마 시스템을 위한 React Context 기반 인프라를 제공한다. `ThemeProvider` 컴포넌트로 테마를 컴포넌트 트리에 전달하고, `useTheme` 및 `useThemeOptional` 두 훅으로 하위 컴포넌트에서 테마를 소비할 수 있게 한다. Provider 없이 `useTheme`를 사용할 경우 명시적인 에러를 던진다.

## 의존성

### 외부 패키지

- `react` — `createContext`, `useContext`, `ReactNode` 타입

### 저장소 내부 모듈

- `@a2ui/web_core/types/types` (타입 전용) — `Types.Theme` 타입 정의
- [`./litTheme`](./litTheme.ts.md) — `defaultTheme` 내보내기 (기본 테마 폴백으로 사용)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ThemeProviderProps` | 인터페이스 | `ThemeProvider`의 props 타입 |
| `ThemeProvider` | 함수 컴포넌트 | 테마를 하위 트리에 제공하는 Provider |
| `useTheme` | 훅 함수 | 현재 테마를 반환, Provider 외부 사용 시 에러 발생 |
| `useThemeOptional` | 훅 함수 | 현재 테마를 반환, Provider 외부이면 `undefined` 반환 |

## 상세 명세

### 모듈 내부 상수: `ThemeContext`

`createContext<Types.Theme | undefined>(undefined)`로 생성된 React Context 인스턴스. 초기값은 `undefined`이며, 외부로 내보내지 않는다.

### 인터페이스 `ThemeProviderProps`

| 필드 | 타입 | 필수 여부 | 설명 |
|------|------|-----------|------|
| `theme` | `Types.Theme` | 선택 | 제공할 테마. 미지정 시 `defaultTheme`이 사용됨 |
| `children` | `ReactNode` | 필수 | 테마에 접근할 하위 컴포넌트 트리 |

### `ThemeProvider({ theme, children }: ThemeProviderProps)`

**반환 타입:** JSX (`ThemeContext.Provider` 래핑된 `children`)

`theme` prop이 전달된 경우 해당 테마를, 전달되지 않은 경우 `defaultTheme`을 Context 값으로 제공한다(`theme ?? defaultTheme`). 조건 분기는 nullish coalescing 연산자 하나로 처리된다.

### `useTheme(): Types.Theme`

**매개변수:** 없음.  
**반환 타입:** `Types.Theme`  
**에러:** `ThemeProvider` 또는 `A2UIProvider` 외부에서 호출 시 `'useTheme must be used within a ThemeProvider or A2UIProvider'` 메시지로 `Error`를 던진다.

`useContext(ThemeContext)`를 호출하고 결과가 falsy이면 에러를 발생시킨다. 테마가 반드시 존재해야 하는 컴포넌트에서 사용한다.

### `useThemeOptional(): Types.Theme | undefined`

**매개변수:** 없음.  
**반환 타입:** `Types.Theme | undefined`

`useContext(ThemeContext)` 결과를 그대로 반환한다. Provider 없이도 안전하게 호출할 수 있으며, `undefined`를 반환 받아 호출자가 처리한다.

## 동작 흐름

앱 최상단에 `<ThemeProvider theme={myTheme}>` 또는 테마 없이 `<ThemeProvider>`를 배치하면 하위 모든 컴포넌트에서 `useTheme()`로 현재 테마를 가져올 수 있다. 테마 지정이 없으면 `litTheme.ts`에 정의된 `defaultTheme`이 자동으로 적용된다.
