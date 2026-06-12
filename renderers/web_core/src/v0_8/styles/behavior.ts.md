# renderers/web_core/src/v0_8/styles/behavior.ts

## 개요

인터랙티브 요소에 호버/포커스 시 불투명도 전환(opacity transition) 효과를 부여하는 CSS 유틸리티 클래스들과 overflow 관련 클래스들을 CSS 문자열로 생성하여 export한다. 외부 의존성 없이 순수 문자열 조작으로 CSS를 생성하는 간단한 파일이다.

## 의존성

없음 (외부 패키지, 저장소 내부 모듈 모두 없음)

## Exports

| 이름 | 종류 |
|------|------|
| `behavior` | 상수 (`string`) |

## 상세 명세

### 비공개 상수: `opacityBehavior`

CSS 규칙 블록을 담은 문자열. `:not([disabled])` 선택자 범위 내에서 다음을 정의한다:
- `cursor: pointer`
- `opacity: var(--opacity, 0)` — CSS 변수 `--opacity`가 없으면 기본값 0
- `transition: opacity var(--speed, 0.2s) cubic-bezier(0, 0, 0.3, 1)` — CSS 변수 `--speed`가 없으면 기본값 0.2s
- `:hover` 및 `:focus` 의사 클래스에서 `opacity: 1`로 완전히 보이도록 전환

---

### `behavior`

export되는 CSS 문자열 상수. 두 부분으로 구성된다:

**1. 호버 불투명도 클래스 (21개)**

`new Array(21).fill(0).map((_, idx) => ...)` 패턴으로 `idx` = 0~20에 대해 클래스를 생성한다:
- 클래스명: `.behavior-ho-${idx * 5}` → `.behavior-ho-0`, `.behavior-ho-5`, ..., `.behavior-ho-100`
- `--opacity` 변수를 `idx / 20`으로 설정 (0.0 ~ 1.0, 0.05 단위)
- `opacityBehavior` 블록을 포함하여 비활성화되지 않은 상태에서의 호버/포커스 전환 동작을 부여

예: `.behavior-ho-50`은 `--opacity: 0.5`를 설정하여 평소 50% 투명도로, 호버/포커스 시 100% 불투명도로 전환된다.

**2. Overflow 제어 클래스 (4개)**
- `.behavior-o-s` — `overflow: scroll`
- `.behavior-o-a` — `overflow: auto`
- `.behavior-o-h` — `overflow: hidden`
- `.behavior-sw-n` — `scrollbar-width: none`

## 동작 흐름

모듈 로드 시 `opacityBehavior` 문자열이 정의되고, `behavior` 상수가 템플릿 리터럴 내에서 배열 생성 및 `map`/`join` 연산으로 즉시 평가되어 완성된 CSS 문자열이 된다. 이 상수는 `index.ts`에서 import되어 `structuralStyles` 합산에 포함된다.
