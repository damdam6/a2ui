# renderers/web_core/src/v0_8/styles/colors.ts

## 개요

팔레트 기반의 색상 CSS 유틸리티 클래스들을 생성하여 `colors` 배열로 export한다. 경계(border), 배경(background), 텍스트(color) 각각에 대해 `light-dark()` CSS 함수를 활용한 다크모드 지원 클래스를 생성한다. 배경 클래스는 `::backdrop` 의사 요소에 대한 투명도 변형도 함께 생성한다. 색상 값은 CSS 커스텀 프로퍼티(변수)로 참조되며, 라이트/다크 모드의 대칭 색상은 셰이드 반전 로직으로 자동 계산된다.

## 의존성

### 저장소 내부 모듈
- [`../types/colors.ts`](../types/colors.ts.md) — `PaletteKey`, `PaletteKeyVals`, `shades`
- [`./utils.ts`](./utils.ts.md) — `toProp`

## Exports

| 이름 | 종류 |
|------|------|
| `colors` | 상수 (`string[]`) |

## 상세 명세

### 비공개 함수: `color`

시그니처: `<C extends PaletteKeyVals>(src: PaletteKey<C>) => string`

색상 키 배열(`src`)을 받아 세 종류의 CSS 클래스 블록을 포함하는 문자열을 반환한다:

**1. border-color 클래스**: `src`의 각 `key`에 대해 `.color-bc-${key}`를 생성. `light-dark(var(--<key>), var(--<inverseKey>))`를 `border-color`로 적용한다.

**2. background-color 클래스**: 각 `key`에 대해 다음을 생성한다:
- `.color-bgc-${key}` — `background-color: light-dark(var(--<key>), var(--<inverseKey>))`
- `.color-bbgc-${key}::backdrop` — 동일한 배경색을 `::backdrop`에 적용
- `.color-bbgc-${key}_10` ~ `.color-bbgc-${key}_90` (0.1 단위, 총 9개) — `::backdrop`에 `oklch(from var(...) l c h / calc(alpha * ${o}))` 형식으로 알파값을 `alpha`의 비율로 조절하는 변형 클래스. `o`는 0.1~0.9 사이를 0.1 단위로 증가하며, 클래스명 숫자는 `(o * 100).toFixed(0)`.

**3. text-color 클래스**: `.color-c-${key}` — `color: light-dark(var(--<key>), var(--<inverseKey>))`

각 케이스에서 `toProp(key)`로 CSS 변수 이름을 생성하고, `getInverseKey(key)`로 다크모드용 반전 색상 키를 구한다.

---

### 비공개 함수: `getInverseKey`

시그니처: `(key: string) => string`

색상 키 문자열(예: `'p50'`)을 받아 대칭 반전 키를 반환한다. 정규식 `/^([a-z]+)(\d+)$/`로 접두어(prefix)와 셰이드 숫자를 분리한다. 패턴에 맞지 않으면 원래 키를 그대로 반환한다.

반전 로직:
1. `shade = parseInt(shadeStr, 10)`
2. `target = 100 - shade` (예: 셰이드 30 → target 70)
3. `shades` 배열에서 `target`과의 절댓값 차이가 가장 작은 값을 `reduce`로 찾아 `inverseShade`로 설정
4. 결과: `${prefix}${inverseShade}` (예: `'p50'` → `'p50'`, `'p30'` → 가장 가까운 70 계열 셰이드)

---

### 비공개 함수: `keyFactory`

시그니처: `<K extends PaletteKeyVals>(prefix: K) => PaletteKey<K>`

`shades` 배열의 각 값에 접두어를 붙여 색상 키 배열을 생성한다. 예: `keyFactory('p')`는 `shades`의 각 요소 `v`에 대해 `'p' + v` 형태의 문자열 배열을 반환한다.

---

### `colors`

export되는 `string[]` 상수. `color(keyFactory(...))` 호출 결과(6개)와 추가 문자열(1개)로 구성된다:

- `color(keyFactory('p'))` — `p` 접두어(primary 팔레트) 클래스
- `color(keyFactory('s'))` — `s` 접두어(secondary 팔레트) 클래스
- `color(keyFactory('t'))` — `t` 접두어(tertiary 팔레트) 클래스
- `color(keyFactory('n'))` — `n` 접두어(neutral 팔레트) 클래스
- `color(keyFactory('nv'))` — `nv` 접두어(neutral-variant 팔레트) 클래스
- `color(keyFactory('e'))` — `e` 접두어(error 팔레트) 클래스
- 추가 리터럴 문자열:
  - `.color-bgc-transparent { background-color: transparent; }` — 투명 배경 유틸리티
  - `:host { color-scheme: var(--color-scheme); }` — 호스트 요소에 color-scheme 적용

## 동작 흐름

모듈 로드 시 `getInverseKey`, `color`, `keyFactory`가 정의되고, `colors` 배열이 즉시 평가된다. 각 팔레트 접두어에 대해 `keyFactory`로 키 배열이 생성되고 `color` 함수가 CSS 클래스 문자열을 반환하여 배열에 포함된다. 이 배열은 `index.ts`에서 `structuralStyles`를 구성할 때 `.flat(Infinity)`로 펼쳐진다.
