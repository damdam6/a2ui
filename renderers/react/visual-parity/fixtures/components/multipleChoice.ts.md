# renderers/react/visual-parity/fixtures/components/multipleChoice.ts

## 개요

MultipleChoice 컴포넌트의 비주얼 패리티 테스트용 단일 픽스처를 제공한다. Lit과 React 렌더러 모두 `<select>` 드롭다운과 설명 레이블을 렌더링하며, `maxAllowedSelections` 설정이 시각적 결과에 영향을 주지 않으므로 단일 픽스처로 충분하다는 점이 주석으로 명시되어 있다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `multipleChoice` | 상수 (`ComponentFixture`) | 3개 옵션을 가진 MultipleChoice 픽스처 |
| `multipleChoiceFixtures` | 상수 (객체) | `multipleChoice`를 감싸는 그룹 객체 |

## 상세 명세

### `multipleChoice: ComponentFixture`

- `root`: `'mc-1'`
- 단일 컴포넌트: `id: 'mc-1'`
- `MultipleChoice` 설정:
  - `selections`: `{path: '/mcSelections'}` — 런타임 데이터 모델의 `/mcSelections` 경로에 바인딩
  - `options`: 3개의 옵션 객체로 구성된 배열
    - `{value: 'option1', label: {literalString: 'Option 1'}}`
    - `{value: 'option2', label: {literalString: 'Option 2'}}`
    - `{value: 'option3', label: {literalString: 'Option 3'}}`
- `data` 필드 미포함 — `/mcSelections`의 초기값은 렌더러 기본값에 의존

### `multipleChoiceFixtures: object`

`multipleChoice` 픽스처를 단일 키-값으로 묶은 집계 객체.

## 동작 흐름

파일은 단일 픽스처와 그 집계 객체만 정의한다. `selections`가 path 바인딩을 사용하므로 런타임 데이터 모델의 `/mcSelections` 경로가 존재해야 선택 상태가 반영되지만, 시각적 렌더링(드롭다운 표시)은 데이터 모델 여부와 무관하게 동작한다.
