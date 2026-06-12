# renderers/react/src/v0_8/lib/utils.ts

## 개요

React 컴포넌트 구현에서 공통으로 사용하는 CSS 클래스 관련 유틸리티 함수를 제공하는 파일이다. `clsx` 라이브러리 기반의 `cn`, 테마 유틸에서 re-export하는 `classMapToString`/`stylesToObject`, 그리고 Lit 스타일 시스템의 `merge` 함수를 래핑한 `mergeClassMaps`를 export한다.

## 의존성

### 외부 패키지
- `clsx` — `clsx`, `ClassValue` (클래스 이름 조합 라이브러리)
- `@a2ui/web_core/styles/index` — `Styles` 네임스페이스 (`merge` 함수 포함)

### 저장소 내부 모듈
- `../theme/utils` — `classMapToString`, `stylesToObject` (re-export)

## Exports

| 이름 | 종류 | 출처 |
|------|------|------|
| `cn` | 함수 | 이 파일 |
| `classMapToString` | 함수 | `../theme/utils` re-export |
| `stylesToObject` | 함수 | `../theme/utils` re-export |
| `mergeClassMaps` | 함수 | 이 파일 |

## 상세 명세

### `cn(...inputs: ClassValue[]): string`

`clsx(inputs)`를 호출하여 가변 인자로 받은 클래스 값들을 하나의 문자열로 병합한다. `ClassValue`는 `string | string[] | Record<string, boolean> | null | undefined` 등을 허용한다.

**예시:** `cn('base', condition && 'extra', {'active': isActive})` → 조건부 클래스가 적용된 문자열.

### `classMapToString` (re-export)

`../theme/utils`에서 re-export. `Record<string, boolean>` 형태의 클래스 맵을 받아 값이 `true`인 키들을 공백으로 이어붙인 문자열을 반환한다.

### `stylesToObject` (re-export)

`../theme/utils`에서 re-export. 테마 스타일 정의를 CSS 인라인 스타일 객체로 변환한다.

### `mergeClassMaps(...maps: (Record<string, boolean> | undefined)[]): Record<string, boolean>`

여러 클래스 맵을 하나로 합친다.

**동작 단계:**
1. `maps.filter((m): m is Record<string, boolean> => m !== undefined)`로 `undefined` 항목을 제거한다.
2. 유효한 맵이 0개이면 빈 객체 `{}`를 반환한다.
3. 유효한 맵이 1개 이상이면 `Styles.merge(...validMaps)`를 호출하여 반환한다.

**Lit `merge` 동작:** 같은 접두사를 공유하는 키들 간 충돌을 처리한다. 예를 들어 `'layout-p-2'`와 `'layout-p-4'`가 모두 있으면 접두사 `'layout-p-'`가 동일하므로 후자만 유지한다.

## 동작 흐름

각 함수는 독립적으로 호출 가능하다. `cn`은 조건부 className 생성, `classMapToString`은 테마 클래스 맵→문자열 변환, `mergeClassMaps`는 여러 테마 클래스 맵을 Lit 방식으로 병합하는 데 사용된다.
