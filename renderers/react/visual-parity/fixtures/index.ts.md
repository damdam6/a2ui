# renderers/react/visual-parity/fixtures/index.ts

## 개요

비주얼 패리티 테스트 전체 픽스처 시스템의 최상위 진입점이다. `components/` 및 `nested/` 디렉토리의 모든 픽스처를 집계하여 `allFixtures` 단일 객체로 제공하며, 타입 유틸리티(`FixtureName`, `fixtureNames`, `fixtureCount`)도 함께 노출한다. React와 Lit 두 렌더러의 최소 페이지가 공통으로 사용하는 공유 모듈이다.

## 의존성

### 저장소 내부 모듈

| 경로 | 설명 |
|---|---|
| `./types` | `ComponentFixture` 타입 정의 |
| [`./components`](./components/index.ts.md) | 모든 컴포넌트 픽스처 그룹 (`textFixtures`, `buttonFixtures` 등 18개) |
| `./nested` | 중첩 레이아웃 픽스처 그룹 (`nestedFixtures`) |

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `ComponentFixture` | 타입 (re-export) | `./types`의 인터페이스 |
| `*` from `./components` | 여러 named export | 컴포넌트 픽스처 전체 |
| `*` from `./nested` | 여러 named export | 중첩 레이아웃 픽스처 전체 |
| `allFixtures` | 상수 (`as const` 객체) | 전체 픽스처를 스프레드로 병합한 단일 맵 |
| `FixtureName` | 타입 | `keyof typeof allFixtures` — 유효한 픽스처 키 유니온 타입 |
| `fixtureNames` | 상수 (`FixtureName[]`) | `Object.keys(allFixtures)`로 생성된 픽스처 이름 배열 |
| `fixtureCount` | 상수 (`number`) | `fixtureNames.length` — 전체 픽스처 수 |

## 상세 명세

### `allFixtures` (상수, `as const` 객체)

컴포넌트별 `*Fixtures` 그룹 객체 18개와 `nestedFixtures`를 스프레드(`...`) 연산자로 병합한 단일 플랫 맵이다. 각 픽스처는 `allFixtures.textBasic`, `allFixtures.imageAvatar` 등 직접 키로 접근 가능하다.

포함된 그룹과 각 픽스처 수 (주석 기준):
- `textFixtures` — 9개
- `buttonFixtures` — 4개
- `iconFixtures` — 2개
- `imageFixtures` — 7개
- `dividerFixtures` — 2개
- `cardFixtures` — 3개
- `rowFixtures` — 6개
- `columnFixtures` — 5개
- `listFixtures` — 4개
- `tabsFixtures` — 3개
- `checkboxFixtures` — 3개
- `textFieldFixtures` — 4개 (주석 수치; 실제 파일은 2개)
- `sliderFixtures` — 3개 (주석 수치; 실제 파일은 1개)
- `dateTimeInputFixtures` — 3개
- `multipleChoiceFixtures` — 1개
- `videoFixtures` — 2개
- `audioPlayerFixtures` — 3개
- `modalFixtures` — 2개
- `nestedFixtures` — 7개

`as const` 어서션이 적용되어 있어 키와 값의 타입이 리터럴로 좁혀진다.

### `FixtureName` (타입)

`keyof typeof allFixtures`로 정의된 유니온 타입. `allFixtures`의 모든 키를 컴파일 타임에 타입으로 표현한다.

### `fixtureNames` (상수, `FixtureName[]`)

`Object.keys(allFixtures) as FixtureName[]` 캐스트로 생성된 배열. 전체 픽스처 이름을 런타임에 순회할 때 사용된다.

### `fixtureCount` (상수, `number`)

`fixtureNames.length`로 계산된 전체 픽스처 수.

## 동작 흐름

파일은 먼저 `./types`에서 `ComponentFixture` 타입을 re-export하고, `./components`와 `./nested`에서 픽스처 그룹 객체를 named import한다. 그 뒤 `allFixtures`를 스프레드로 조합하고, 파생 타입(`FixtureName`)과 파생 상수(`fixtureNames`, `fixtureCount`)를 정의한다. 마지막에 `export * from './components'`와 `export * from './nested'`로 하위 모듈의 모든 export를 재-export한다. 소비자 코드는 이 파일 하나에서 개별 픽스처 접근, 전체 순회, 타입 검사 모두를 수행할 수 있다.
