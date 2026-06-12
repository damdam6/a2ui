# renderers/react/visual-parity/fixtures/components/dateTimeInput.ts

## 개요

`DateTimeInput` 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. 날짜만 활성화된 케이스, 시간만 활성화된 케이스, 날짜와 시간 모두 활성화된 케이스 세 가지를 제공한다. `value`는 모두 `literalString`으로 지정하며 `data` 필드는 사용하지 않는다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `dateTimeInputDate`: `ComponentFixture` — `enableDate: true`, `enableTime: false` 픽스처
- `dateTimeInputTime`: `ComponentFixture` — `enableDate: false`, `enableTime: true` 픽스처
- `dateTimeInputBoth`: `ComponentFixture` — `enableDate: true`, `enableTime: true` 픽스처
- `dateTimeInputFixtures`: `{ dateTimeInputDate, dateTimeInputTime, dateTimeInputBoth }` — 컬렉션 객체

## 상세 명세

### `dateTimeInputDate: ComponentFixture`

- `root`: `'dt-date'`
- `data`: 없음
- `components`:
  - `id: 'dt-date'`, `DateTimeInput`
  - `value: {literalString: '2025-02-15'}` — ISO 날짜 형식
  - `enableDate: true`, `enableTime: false`

### `dateTimeInputTime: ComponentFixture`

- `root`: `'dt-time'`
- `data`: 없음
- `components`:
  - `id: 'dt-time'`, `DateTimeInput`
  - `value: {literalString: '14:30'}` — `HH:MM` 시간 형식
  - `enableDate: false`, `enableTime: true`

### `dateTimeInputBoth: ComponentFixture`

- `root`: `'dt-both'`
- `data`: 없음
- `components`:
  - `id: 'dt-both'`, `DateTimeInput`
  - `value: {literalString: '2025-02-15T14:30'}` — ISO 날짜+시간 형식
  - `enableDate: true`, `enableTime: true`

### `dateTimeInputFixtures`

세 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

`enableDate`와 `enableTime` 조합에 따라 DateTimeInput이 날짜 선택기, 시간 선택기, 또는 날짜+시간 선택기로 렌더링되는 세 가지 모드를 각각 검증한다. 각 value 문자열 형식은 해당 모드에 맞는 ISO 표현을 사용한다: 날짜만 `YYYY-MM-DD`, 시간만 `HH:MM`, 둘 다 `YYYY-MM-DDTHH:MM`.
