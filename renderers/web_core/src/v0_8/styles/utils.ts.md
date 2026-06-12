# renderers/web_core/src/v0_8/styles/utils.ts

## 개요

Tailwind 스타일의 클래스 객체를 병합하고, 여러 HTML 태그에 걸쳐 클래스를 일괄 적용하며, 색상 팔레트 키를 CSS 변수로 변환하는 유틸리티 함수들을 제공한다. "같은 접두사를 가진 클래스는 하나만 유효"라는 일관성 원칙을 구현하여 스타일 충돌을 방지한다. 테마 생성 시 `ColorPalettes` 타입으로부터 CSS 커스텀 프로퍼티 맵을 만드는 `createThemeStyles`도 포함한다.

## 의존성

- 외부 패키지: 없음
- 저장소 내부 모듈:
  - [`../types/colors.ts`](../types/colors.ts.md) — `ColorPalettes` 타입 임포트

## Exports

| 이름 | 종류 |
|------|------|
| `merge` | 함수 |
| `appendToAll` | 함수 |
| `createThemeStyles` | 함수 |
| `toProp` | 함수 |

## 상세 명세

### `merge(...classes: Array<Record<string, boolean>>): Record<string, boolean>`

여러 클래스 객체(Tailwind-style `{클래스명: boolean}` 맵)를 하나로 합친다.

**동작 로직:**

1. 빈 결과 객체 `styles`를 생성한다.
2. 인자로 받은 각 `clazz`를 순서대로 처리한다.
3. 각 `clazz` 안의 `[key, val]` 쌍에 대해 접두사(`prefix`)를 계산한다. 접두사는 `key`를 `-`로 분리한 뒤 마지막 세그먼트를 빈 문자열로 교체하고 다시 `-`로 join한 값이다. 예: `"text-red"` → prefix `"text-"`.
4. 현재 `styles` 안에서 해당 접두사로 시작하는 기존 키를 모두 찾아 삭제한다.
5. 새 `key: val`를 `styles`에 추가한다.
6. 모든 처리가 끝난 `styles`를 반환한다.

**경계 케이스:** 나중에 전달된 클래스 객체의 키가 이전 객체와 같은 접두사를 가지면 이전 항목이 제거되고 새 항목만 남는다.

---

### `appendToAll(target: Record<string, string[]>, exclusions: string[], ...classes: Array<Record<string, boolean>>): Record<string, string[]>`

`target`(태그명 → 클래스 배열 맵)의 각 항목에 새 클래스들을 추가한다. 같은 접두사를 가진 기존 클래스는 새 클래스로 교체되며, `exclusions` 에 포함된 태그는 수정에서 제외된다.

**동작 로직:**

1. `structuredClone(target)`으로 깊은 복사본 `updatedTarget`을 만든다.
2. 각 `clazz` 객체의 모든 `key`에 대해 접두사(`prefix`)를 계산한다 (`merge`와 동일한 방식).
3. `updatedTarget`의 각 `[tagName, classesToAdd]` 쌍에 대해:
   - `tagName`이 `exclusions` 배열에 포함되어 있으면 건너뛴다.
   - `classesToAdd` 배열을 순회하며 `prefix`로 시작하는 항목을 찾아 `key`로 직접 교체한다. `found` 플래그로 기존 항목 존재 여부를 추적한다.
   - 배열 전체를 끝까지 순회하여 같은 접두사를 가진 항목이 여러 개 있어도 모두 교체한다(안전 처리).
   - 기존 항목이 없었다면(`found === false`) `key`를 배열 끝에 추가(`push`)한다.
4. 수정된 `updatedTarget`을 반환한다.

---

### `createThemeStyles(palettes: ColorPalettes): Record<string, string>`

`ColorPalettes` 객체의 모든 팔레트를 순회하여 CSS 커스텀 프로퍼티 맵을 생성한다.

**동작 로직:**

1. 빈 결과 객체 `styles`를 생성한다.
2. `Object.values(palettes)` 로 각 팔레트(예: `neutral`, `primary` 등)를 순회한다.
3. 각 팔레트의 `[key, val]` 쌍에 대해 `toProp(key)`로 CSS 변수명을 생성하고 `styles[prop] = val`로 저장한다.
4. `styles`를 반환한다.

---

### `toProp(key: string): string`

팔레트 키 문자열을 CSS 커스텀 프로퍼티 이름으로 변환한다.

**동작 로직:**

- `key`가 `"nv"`로 시작하면: `--nv-` + `key.slice(2)` 를 반환한다. 예: `"nv10"` → `"--nv-10"`.
- 그 외의 경우: `--` + `key[0]` + `-` + `key.slice(1)` 을 반환한다. 예: `"p20"` → `"--p-20"`.

## 동작 흐름

`merge`와 `appendToAll`은 접두사 기반 충돌 해소 원칙을 공유한다. `toProp`은 팔레트 키 명명 규칙(`nv` 접두사 vs. 단일 문자)에 따라 두 가지 변환 경로를 가진다. `createThemeStyles`는 `toProp`을 내부적으로 호출하여 팔레트 전체를 CSS 변수 레코드로 변환하는 최종 조합 함수 역할을 한다.
