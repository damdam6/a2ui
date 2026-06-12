# renderers/react/visual-parity/fixtures/themes/index.ts

## 개요

시각적 동등성 테스트에서 사용 가능한 테마들의 레지스트리를 제공하는 모듈이다. URL 쿼리 파라미터로 전달된 테마 이름을 실제 `Types.Theme` 객체로 변환하는 `getTheme` 함수와, 테마 이름 목록·매핑 객체를 내보낸다. `default` 항목에 `undefined`를 할당하여 테마 미지정(폴백 스타일링) 케이스도 명시적으로 처리한다.

## 의존성

### 외부 패키지

- `@a2ui/react` — `litTheme` (A2UI 기본 Lit 스타일 테마)
- `@a2ui/lit/0.8` — `Types.Theme` 타입

### 저장소 내부 모듈

- [`./visualParityTheme`](./visualParityTheme.ts.md) — `visualParityTheme` 객체
- [`./minimalTheme`](./minimalTheme.ts.md) — `minimalTheme` 객체

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `testThemes` | 상수 (`Record<string, Types.Theme \| undefined>`) | 테마 이름 → 테마 객체 레지스트리 |
| `ThemeName` | 타입 | `testThemes`의 키 유니온 타입 |
| `themeNames` | 상수 (`ThemeName[]`) | `testThemes`의 모든 키 배열 |
| `getTheme` | 함수 | 이름으로 테마 객체를 조회해 반환 |
| `litTheme` | 재내보내기 | `@a2ui/react`의 `litTheme` |
| `visualParityTheme` | 재내보내기 | `./visualParityTheme`의 `visualParityTheme` |
| `minimalTheme` | 재내보내기 | `./minimalTheme`의 `minimalTheme` |

## 상세 명세

### `testThemes: Record<string, Types.Theme | undefined>`

4개의 항목을 가지는 객체 상수다.

- `'default'`: `undefined` — 테마 없음, 렌더러 폴백 스타일 검증용
- `'lit'`: `litTheme` — `@a2ui/react`에서 가져온 A2UI 기본 테마
- `'visualParity'`: `visualParityTheme` — 테마 전환 검증용 대체 테마
- `'minimal'`: `minimalTheme` — 구조적 정확성 검증용 최소 테마

### `ThemeName`

`type ThemeName = keyof typeof testThemes`로 선언된 타입. 가능한 값: `'default' | 'lit' | 'visualParity' | 'minimal'`.

### `themeNames: ThemeName[]`

`Object.keys(testThemes) as ThemeName[]`로 생성. `testThemes`의 키를 런타임 배열로 제공하여, 테스트 하네스가 전체 테마를 순회하거나 UI 목록을 생성할 때 사용한다.

### `getTheme(name: string | null): Types.Theme | undefined`

- 매개변수: `name` — URL 쿼리 파라미터 등에서 넘어온 테마 이름 문자열 또는 `null`
- 반환 타입: `Types.Theme | undefined`
- 동작:
  1. `name`이 falsy이거나 `testThemes`의 키에 존재하지 않으면 `testThemes.default`(`undefined`)를 반환한다.
  2. 조건을 통과하면 `testThemes[name]`을 반환한다(유효한 테마 객체 또는 `'default'` 키에 해당하는 `undefined`).

## 동작 흐름

모듈이 로드될 때 세 개의 테마 객체(`litTheme`, `visualParityTheme`, `minimalTheme`)가 임포트되어 `testThemes` 레코드에 등록된다. `themeNames` 배열이 즉시 파생 계산된다. 이후 소비자(테스트 파일, 렌더러 진입점 등)가 `getTheme(name)` 또는 `themeNames`를 사용하여 원하는 테마를 선택한다.
