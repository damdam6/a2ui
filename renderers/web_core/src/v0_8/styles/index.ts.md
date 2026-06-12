# renderers/web_core/src/v0_8/styles/index.ts

## 개요

`v0_8/styles` 디렉토리의 진입점(entry point) 파일이다. 각 스타일 모듈(`behavior`, `border`, `colors`, `icons`, `layout`, `opacity`, `type`)에서 CSS 문자열을 가져와 단일 문자열 상수 `structuralStyles`로 합산하고 export한다. `utils.ts`의 모든 export도 그대로 재-export한다.

## 의존성

### 저장소 내부 모듈
- [`./behavior.ts`](./behavior.ts.md) — `behavior`
- [`./border.ts`](./border.ts.md) — `border`
- [`./colors.ts`](./colors.ts.md) — `colors`
- [`./icons.ts`](./icons.ts.md) — `icons`
- [`./layout.ts`](./layout.ts.md) — `layout`
- [`./opacity.ts`](./opacity.ts.md) — `opacity`
- `./type.ts` — `type`
- [`./utils.ts`](./utils.ts.md) — (모든 export를 재-export)

## Exports

| 이름 | 종류 |
|------|------|
| `structuralStyles` | 상수 (`string`) |
| `utils.ts`의 모든 export | 재-export (`export * from './utils.js'`) |

## 상세 명세

### `structuralStyles`

시그니처: `const structuralStyles: string`

`[behavior, border, colors, icons, layout, opacity, type]` 배열을 `.flat(Infinity)`로 완전히 평탄화한 뒤 `.join('\n')`으로 합산한 CSS 문자열. `colors`가 `string[]` 타입이므로 `.flat(Infinity)` 호출이 필수적이다. 나머지 항목들은 이미 `string` 타입이지만 동일한 처리를 거친다. 결과는 모든 구조적 스타일을 하나의 문자열로 연결한 완성된 CSS이다.

## 동작 흐름

모듈 로드 시 7개 스타일 모듈이 import되고, `structuralStyles`가 즉시 평가되어 단일 CSS 문자열이 생성된다. 소비자는 이 문자열을 `<style>` 태그에 주입하거나 Shadow DOM의 `adoptedStyleSheets`에 적용하여 구조적 스타일을 일괄 활성화할 수 있다.
