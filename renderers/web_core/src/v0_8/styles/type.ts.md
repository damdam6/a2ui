# renderers/web_core/src/v0_8/styles/type.ts

## 개요

타이포그래피에 관련된 CSS 규칙 전체를 하나의 문자열 상수로 패키징하는 파일이다. `:host` 범위 CSS 변수 선언, 폰트 패밀리 클래스, 텍스트 정렬·스타일·크기·줄 높이 클래스, 그리고 굵기(`font-weight`) 클래스들이 모두 이 단일 템플릿 리터럴에 포함된다. Web Component의 Shadow DOM `<style>` 태그에 직접 삽입하여 사용하도록 설계되어 있다.

## 의존성

- 외부 패키지: 없음
- 저장소 내부 모듈: 없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `type` | 상수 (`string`) | 타이포그래피 CSS 규칙 전체를 담은 문자열 |

## 상세 명세

### `type`

- 타입: `string` (템플릿 리터럴)
- 내용: 모듈 평가 시점에 문자열로 확정된다.

**`:host` 블록**

CSS 변수 두 가지를 선언한다.
- `--default-font-family`: `"Helvetica Neue", Helvetica, Arial, sans-serif`
- `--default-font-family-mono`: `"Courier New", Courier, monospace`

**폰트 패밀리 클래스**

| 클래스명 | 적용 속성 |
|---|---|
| `.typography-f-s` | `var(--font-family, var(--default-font-family))`, `font-optical-sizing: auto`, `font-variation-settings: "slnt" 0, "wdth" 100, "GRAD" 0` |
| `.typography-f-sf` | `var(--font-family-flex, var(--default-font-family))`, `font-optical-sizing: auto` |
| `.typography-f-c` | `var(--font-family-mono, var(--default-font-family))`, `font-optical-sizing: auto`, `font-variation-settings: "slnt" 0, "wdth" 100, "GRAD" 0` |
| `.typography-v-r` | `font-variation-settings: "slnt" 0, "wdth" 100, "GRAD" 0, "ROND" 100` |

**텍스트 정렬·스타일 클래스**

| 클래스명 | 속성 |
|---|---|
| `.typography-ta-s` | `text-align: start` |
| `.typography-ta-c` | `text-align: center` |
| `.typography-fs-n` | `font-style: normal` |
| `.typography-fs-i` | `font-style: italic` |
| `.typography-ws-p` | `white-space: pre-line` |
| `.typography-ws-nw` | `white-space: nowrap` |
| `.typography-td-none` | `text-decoration: none` |

**크기(size) 클래스** — 명칭 규칙: `typography-sz-{스케일코드}`, `font-size` / `line-height` 쌍

| 클래스명 | font-size | line-height |
|---|---|---|
| `.typography-sz-ls` | 11px | 16px |
| `.typography-sz-lm` | 12px | 16px |
| `.typography-sz-ll` | 14px | 20px |
| `.typography-sz-bs` | 12px | 16px |
| `.typography-sz-bm` | 14px | 20px |
| `.typography-sz-bl` | 16px | 24px |
| `.typography-sz-ts` | 14px | 20px |
| `.typography-sz-tm` | 16px | 24px |
| `.typography-sz-tl` | 22px | 28px |
| `.typography-sz-hs` | 24px | 32px |
| `.typography-sz-hm` | 28px | 36px |
| `.typography-sz-hl` | 32px | 40px |
| `.typography-sz-ds` | 36px | 44px |
| `.typography-sz-dm` | 45px | 52px |
| `.typography-sz-dl` | 57px | 64px |

**굵기(weight) 클래스** — 런타임 생성

길이 9인 배열을 순회하며 인덱스 `idx`(0~8)에 대해 `weight = (idx + 1) * 100` 을 계산한다. 결과로 `.typography-w-100` ~ `.typography-w-900` 클래스가 생성되며, 각 클래스는 `font-weight: <weight>` 속성 하나를 갖는다. 배열 요소들은 `\n` 으로 join되어 전체 문자열에 포함된다.

## 동작 흐름

모듈 임포트 시 템플릿 리터럴이 즉시 평가된다. weight 클래스 생성을 위해 `new Array(9).fill(0).map(...).join('\n')` 이 인라인으로 실행되고, 그 결과 문자열이 전체 CSS 텍스트 안에 보간된다. 이후 `type` 상수는 변경 불가능한 문자열 값이다.
