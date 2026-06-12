# renderers/react/src/v0_8/theme/utils.ts

## 개요

A2UI 테마 시스템에서 사용하는 두 가지 데이터 변환 유틸리티 함수를 제공하는 파일이다. Lit 스타일의 클래스맵(`Record<string, boolean>`)을 HTML `className` 문자열로 변환하고, CSS 속성 객체(`Record<string, string>`)를 React의 `style` prop 형태로 변환한다.

## 의존성

### 외부 패키지

- `react` (타입 전용) — `React.CSSProperties` 타입 참조

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `classMapToString` | 함수 | 클래스맵 객체를 공백 구분 클래스 문자열로 변환 |
| `stylesToObject` | 함수 | CSS 속성 문자열 객체를 React `CSSProperties` 객체로 변환 |

## 상세 명세

### `classMapToString(classMap: Record<string, boolean> | undefined): string`

**매개변수:** `classMap` — 키가 CSS 클래스명, 값이 활성화 여부를 나타내는 불리언인 객체. `undefined`도 허용.  
**반환 타입:** `string`

1. `classMap`이 falsy이면 빈 문자열 `''`을 즉시 반환한다.
2. `Object.entries(classMap)`으로 `[클래스명, 불리언]` 쌍의 배열을 생성한다.
3. `.filter(([, enabled]) => enabled)`로 값이 `true`인 항목만 남긴다.
4. `.map(([className]) => className)`으로 클래스명만 추출한다.
5. `.join(' ')`으로 공백 구분 문자열을 반환한다.

예시: `{ 'a2ui-button': true, 'a2ui-button--primary': true, 'disabled': false }` → `'a2ui-button a2ui-button--primary'`

### `stylesToObject(styles: Record<string, string> | undefined): React.CSSProperties | undefined`

**매개변수:** `styles` — CSS 속성명(kebab-case 또는 CSS 커스텀 프로퍼티)을 키로 하고 문자열 값을 가지는 객체. `undefined`도 허용.  
**반환 타입:** `React.CSSProperties | undefined`

1. `styles`가 falsy이거나 `Object.keys(styles).length === 0`이면 `undefined`를 즉시 반환한다.
2. 빈 `result` 객체(`Record<string, string>`)를 생성한다.
3. `Object.entries(styles)`를 순회하며 각 키-값 쌍을 처리:
   - 키가 `'--'`로 시작하는 CSS 커스텀 프로퍼티는 변환 없이 그대로 `result[key] = value`로 설정한다.
   - 그 외 kebab-case 키는 `key.replace(/-([a-z])/g, (_, letter) => letter.toUpperCase())`로 camelCase로 변환한 뒤 `result[camelKey] = value`로 설정한다.
4. 결과를 `React.CSSProperties`로 타입 단언하여 반환한다.

예시: `{ 'background-color': 'red', 'font-size': '16px', '--custom-var': 'blue' }` → `{ backgroundColor: 'red', fontSize: '16px', '--custom-var': 'blue' }`

## 동작 흐름

두 함수는 독립적이며 상태를 갖지 않는다. 컴포넌트 렌더 시점에 테마 클래스맵이나 additionalStyles를 React가 이해하는 형태로 변환할 때 호출된다. `classMapToString`은 `className` prop 구성에, `stylesToObject`는 `style` prop 구성에 사용된다.
