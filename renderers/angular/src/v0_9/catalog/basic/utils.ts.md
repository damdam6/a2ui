# renderers/angular/src/v0_9/catalog/basic/utils.ts

## 개요

A2UI 레이아웃 속성 값을 CSS 속성 값으로 변환하기 위한 매핑 상수를 제공하는 유틸리티 모듈이다. `justify-content`와 `align-items` CSS 속성에 대응하는 두 개의 룩업 테이블을 export한다. 외부 의존성이 없는 순수 데이터 파일이다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `JUSTIFY_MAP` | 상수 (`Record<string, string>`) | A2UI justify 값 → CSS `justify-content` 값 매핑 |
| `ALIGN_MAP` | 상수 (`Record<string, string>`) | A2UI align 값 → CSS `align-items` 값 매핑 |

## 상세 명세

### `JUSTIFY_MAP`

타입: `Record<string, string>`

A2UI 프로토콜의 justification 키를 CSS flexbox `justify-content` 값으로 변환하는 객체. 키-값 쌍은 다음과 같다:

- `'start'` → `'flex-start'`
- `'center'` → `'center'`
- `'end'` → `'flex-end'`
- `'spaceBetween'` → `'space-between'`
- `'spaceAround'` → `'space-around'`
- `'spaceEvenly'` → `'space-evenly'`
- `'stretch'` → `'stretch'`

### `ALIGN_MAP`

타입: `Record<string, string>`

A2UI 프로토콜의 alignment 키를 CSS flexbox `align-items` 값으로 변환하는 객체. 키-값 쌍은 다음과 같다:

- `'start'` → `'flex-start'`
- `'center'` → `'center'`
- `'end'` → `'flex-end'`
- `'stretch'` → `'stretch'`
- `'baseline'` → `'baseline'`

## 동작 흐름

이 파일은 런타임 로직 없이 두 개의 상수 객체만을 선언한다. 컴포넌트에서 A2UI 프로토콜 속성값을 CSS 스타일 바인딩에 적용할 때 이 맵을 룩업 테이블로 사용한다 (예: `JUSTIFY_MAP[props.justify]`).
