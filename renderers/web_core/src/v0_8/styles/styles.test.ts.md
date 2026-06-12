# renderers/web_core/src/v0_8/styles/styles.test.ts

## 개요

`utils.ts`에서 내보내는 네 가지 함수(`merge`, `appendToAll`, `toProp`, `createThemeStyles`)의 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너(`node:test`, `node:assert`)를 사용하며 외부 테스트 프레임워크에 의존하지 않는다.

## 의존성

- 외부 패키지: `node:test` (`describe`, `it`), `node:assert`
- 저장소 내부 모듈:
  - [`./utils.ts`](./utils.ts.md) — `merge`, `appendToAll`, `createThemeStyles`, `toProp` 임포트

## Exports

없음 (테스트 파일이므로 공개 API를 내보내지 않음)

## 테스트 케이스 명세

### `describe('Styles Utils')`

#### `describe('merge')`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `'merges multiple class objects'` | `{'text-red': true, 'bg-blue': true}`와 `{'text-green': true}`를 병합하면 같은 접두사 `text-`를 가진 `text-red`가 제거되고 `text-green`이 남으며, `bg-blue`는 그대로 유지됨 | 인라인 객체 리터럴 |
| `'handles complex prefixes'` | `{'border-t-2': true}`와 `{'border-t-4': true}`를 병합하면 `border-t-2`가 제거되고 `border-t-4`만 남음 | 인라인 객체 리터럴 |

**검증 방법:** `assert.strictEqual`로 결과 키의 존재 여부(`undefined` 여부) 및 값을 확인한다.

---

#### `describe('appendToAll')`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `'appends classes to target'` | `exclusions=[]` 일 때 `div`와 `span` 모두에 `text-red`가 추가되어 각각 `['base-class', 'text-red']`, `['other-class', 'text-red']`가 됨 | `target = {div: ['base-class'], span: ['other-class']}` |
| `'replaces classes with same prefix'` | `div`의 기존 `text-blue`가 `text-red`로 교체되어 `['text-red']`가 됨 | `target = {div: ['text-blue']}` |
| `'respects exclusions'` | `exclusions=['excluded']` 일 때 `div`에는 `new-class`가 추가되지만 `excluded` 태그는 변경되지 않음 | `target = {div: ['base'], excluded: ['base']}` |

**검증 방법:** `assert.deepStrictEqual`로 결과 배열을 깊은 비교한다.

---

#### `describe('toProp')`

| 테스트명 | 검증 동작 |
|---|---|
| `'converts nv- keys'` | `toProp('nv10')`이 `'--nv-10'`을 반환함 |
| `'converts other keys'` | `toProp('p20')`이 `'--p-20'`을 반환함 (주석: `p20 -> --p-20`) |

**검증 방법:** `assert.strictEqual`

---

#### `describe('createThemeStyles')`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `'creates css variables from palette'` | `{primary: {nv10: 'red', p20: 'blue'}}` 팔레트를 입력하면 `{'--nv-10': 'red', '--p-20': 'blue'}` 스타일 맵이 생성됨 | `as any` 캐스팅을 사용하여 `ColorPalettes` 타입 제약 우회 |

**검증 방법:** `assert.strictEqual`로 각 CSS 변수의 값을 확인한다.

## 동작 흐름

Node.js 테스트 러너가 파일을 실행하면 `describe` 블록이 등록되고 각 `it` 콜백이 순차적으로 실행된다. 모든 검증은 동기적으로 이루어지며 비동기 처리나 모킹 라이브러리는 사용하지 않는다.
