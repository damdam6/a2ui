# renderers/web_core/src/v0_8/types/colors.ts

## 개요

Material Design 색상 시스템(팔레트)을 표현하기 위한 TypeScript 타입 정의와 상수를 제공하는 파일이다. 6가지 팔레트 카테고리(neutral, neutralVariant, primary, secondary, tertiary, error)를 제네릭 타입으로 모델링하고, 각 팔레트가 가져야 하는 정해진 색조(shade) 값들을 리터럴 유니언 타입으로 제약한다.

## 의존성

- 외부 패키지: 없음
- 저장소 내부 모듈: 없음

## Exports

| 이름 | 종류 |
|------|------|
| `PaletteKeyVals` | 타입 (리터럴 유니언) |
| `shades` | 상수 (배열) |
| `PaletteKey<Prefix>` | 타입 (제네릭) |
| `PaletteKeys` | 타입 (객체) |
| `ColorPalettes` | 타입 (객체) |

## 상세 명세

### `ColorShade` (비공개 타입)

- 타입: 숫자 리터럴 유니언
- 허용 값: `0 | 5 | 10 | 15 | 20 | 25 | 30 | 35 | 40 | 50 | 60 | 70 | 80 | 90 | 95 | 98 | 99 | 100`
- 역할: 팔레트 내 색조를 나타내는 18가지 정해진 숫자 값만 허용하는 내부 타입. 비공개이므로 파일 외부에서 직접 참조할 수 없다.

---

### `PaletteKeyVals`

- 타입: `'n' | 'nv' | 'p' | 's' | 't' | 'e'`
- 역할: 6가지 팔레트 카테고리에 대응하는 단문자/이중문자 접두사 리터럴 유니언. `n`=neutral, `nv`=neutralVariant, `p`=primary, `s`=secondary, `t`=tertiary, `e`=error.

---

### `shades`

- 타입: `ColorShade[]`
- 값: `[0, 5, 10, 15, 20, 25, 30, 35, 40, 50, 60, 70, 80, 90, 95, 98, 99, 100]`
- 역할: `ColorShade` 타입에 포함된 모든 색조 숫자를 배열로 나열한 런타임 상수. 팔레트 키 목록을 동적으로 생성하거나 순회할 때 사용한다.

---

### `CreatePalette<Prefix extends PaletteKeyVals>` (비공개 유틸리티 타입)

- 제네릭: `Prefix extends PaletteKeyVals`
- 구조: `{ [Key in \`${Prefix}${ColorShade}\`]: string }`
- 역할: 접두사와 모든 `ColorShade` 값의 조합으로 키를 생성하는 매핑 타입. 예를 들어 `CreatePalette<'p'>` 는 `p0`, `p5`, `p10`, ..., `p100` 이라는 18개의 키를 string 타입 값으로 갖는 객체 타입이 된다. 비공개이므로 `PaletteKey`와 `ColorPalettes` 정의에서만 내부적으로 사용된다.

---

### `PaletteKey<Prefix extends PaletteKeyVals>`

- 제네릭: `Prefix extends PaletteKeyVals`
- 타입: `Array<keyof CreatePalette<Prefix>>`
- 역할: 특정 접두사 팔레트의 키 이름 문자열 배열 타입. 예: `PaletteKey<'p'>` = `Array<'p0' | 'p5' | ... | 'p100'>`.

---

### `PaletteKeys`

- 타입: 객체 타입
- 구조: 6개의 팔레트 이름을 키로, 대응하는 `PaletteKey<접두사>` 를 값으로 갖는다.

| 키 | 타입 |
|---|---|
| `neutral` | `PaletteKey<'n'>` |
| `neutralVariant` | `PaletteKey<'nv'>` |
| `primary` | `PaletteKey<'p'>` |
| `secondary` | `PaletteKey<'s'>` |
| `tertiary` | `PaletteKey<'t'>` |
| `error` | `PaletteKey<'e'>` |

---

### `ColorPalettes`

- 타입: 객체 타입
- 역할: 실제 색상값을 담는 팔레트 컬렉션의 완전한 구조를 정의한다. `utils.ts`의 `createThemeStyles`가 이 타입을 파라미터로 받는다.

| 키 | 타입 |
|---|---|
| `neutral` | `CreatePalette<'n'>` |
| `neutralVariant` | `CreatePalette<'nv'>` |
| `primary` | `CreatePalette<'p'>` |
| `secondary` | `CreatePalette<'s'>` |
| `tertiary` | `CreatePalette<'t'>` |
| `error` | `CreatePalette<'e'>` |

각 팔레트는 접두사+색조 이름(예: `nv10`, `p50`)을 키로, 색상 문자열을 값으로 갖는 완전한 객체여야 한다.

## 동작 흐름

이 파일은 타입과 한 개의 런타임 상수(`shades`)만 포함한다. `shades`는 모듈 임포트 시 즉시 배열로 평가된다. 타입들은 컴파일 타임에만 존재하며 런타임에는 영향을 주지 않는다.
