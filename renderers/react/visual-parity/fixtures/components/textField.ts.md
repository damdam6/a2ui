# renderers/react/visual-parity/fixtures/components/textField.ts

## 개요

TextField 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. 레이블만 있는 빈 입력 필드와 레이블 및 초기 텍스트 값이 모두 설정된 입력 필드 두 가지 시나리오를 제공한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `textField` | 상수 (`ComponentFixture`) | 레이블만 있는 기본 TextField 픽스처 |
| `textFieldWithValue` | 상수 (`ComponentFixture`) | 레이블과 초기 텍스트 값이 있는 TextField 픽스처 |
| `textFieldFixtures` | 상수 (객체) | 위 2개 픽스처를 묶은 그룹 객체 |

## 상세 명세

### `textField: ComponentFixture`

- `root`: `'tf-1'`
- 단일 컴포넌트: `id: 'tf-1'`
- `TextField.label`: `{literalString: 'Your Name'}`
- `text` 속성 없음 — 빈 입력 필드 상태

### `textFieldWithValue: ComponentFixture`

- `root`: `'tf-value'`
- 단일 컴포넌트: `id: 'tf-value'`
- `TextField.label`: `{literalString: 'Email'}`
- `TextField.text`: `{literalString: 'user@example.com'}` — 미리 채워진 텍스트 값

### `textFieldFixtures: object`

`textField`와 `textFieldWithValue`를 키-값으로 묶은 집계 객체.

## 동작 흐름

두 픽스처 모두 `data` 필드 없이 `literalString`으로 값을 지정하므로 데이터 모델 주입 없이 정적으로 렌더링된다. `textField`는 빈 필드 표시를, `textFieldWithValue`는 사전 입력된 필드 표시를 검증한다.
