# renderers/react/visual-parity/fixtures/types.ts

## 개요

시각적 동등성 테스트에서 사용하는 컴포넌트 픽스처(fixture)의 타입을 정의하는 파일이다. 단 하나의 인터페이스 `ComponentFixture`만 포함하며, 모든 픽스처 데이터 파일이 이 타입을 기반으로 작성된다.

## 의존성

### 외부 패키지

없음.

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `ComponentFixture` | 인터페이스 | 컴포넌트 픽스처의 데이터 구조 계약 |

## 상세 명세

### `ComponentFixture` (인터페이스)

컴포넌트 픽스처 하나의 완전한 구조를 기술한다.

#### 필드

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| `root` | `string` | 필수 | 렌더링 시작점이 되는 컴포넌트의 `id` |
| `components` | `Array<{id: string; component: Record<string, unknown>}>` | 필수 | 픽스처에 포함된 컴포넌트 목록 |
| `data` | `Record<string, unknown>` | 선택 | 렌더링 전 주입할 초기 데이터 모델 값 |

#### `components` 배열 원소

각 원소는 두 필드를 가지는 익명 객체이다.

- `id: string` — 컴포넌트를 식별하는 고유 문자열. 다른 컴포넌트가 자식을 참조할 때 이 값을 사용한다.
- `component: Record<string, unknown>` — A2UI 컴포넌트 명세를 담은 객체. 실제 구조는 A2UI 메시지 프로토콜에 따라 다양하므로 `unknown` 값 타입을 사용한다.

#### `data` 필드 (선택)

- 키는 JSON Pointer 경로 형식(예: `'/settings/checked'`)이다.
- 값은 임의 타입(`unknown`)이며, 렌더링 이전에 `updateDataModel` 메시지로 렌더러에 주입된다.

## 동작 흐름

이 파일은 순수한 타입 선언 모듈이다. 런타임 코드가 없으며, TypeScript 컴파일 타임에만 사용된다. 픽스처 데이터 파일들이 `import type {ComponentFixture} from '../types'`로 가져와 타입 안전성을 확보한다.
