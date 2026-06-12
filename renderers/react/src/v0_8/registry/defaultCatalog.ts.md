# renderers/react/src/v0_8/registry/defaultCatalog.ts

## 개요

A2UI React 렌더러의 기본 컴포넌트 카탈로그를 등록하는 모듈이다. Content, Layout, Interactive 세 범주로 나뉜 18개의 표준 컴포넌트를 `ComponentRegistry`에 등록하는 역할을 담당한다. 앱 초기화 시점에 한 번 호출하여 싱글톤 레지스트리를 준비하는 진입점 함수도 제공한다.

## 의존성

### 저장소 내부 모듈

- [`./ComponentRegistry`](./ComponentRegistry.ts.md) — 컴포넌트를 이름-구현체 쌍으로 저장하는 싱글톤 레지스트리 클래스
- `../components/content/Text`, `../components/content/Image`, `../components/content/Icon`, `../components/content/Divider`, `../components/content/Video`, `../components/content/AudioPlayer` — Content 컴포넌트 구현체들
- `../components/layout/Row`, `../components/layout/Column`, `../components/layout/List`, `../components/layout/Card`, `../components/layout/Tabs`, `../components/layout/Modal` — Layout 컴포넌트 구현체들
- `../components/interactive/Button`, `../components/interactive/TextField`, `../components/interactive/CheckBox`, `../components/interactive/Slider`, `../components/interactive/DateTimeInput`, `../components/interactive/MultipleChoice` — Interactive 컴포넌트 구현체들

### 외부 패키지

없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `registerDefaultCatalog` | 함수 | 주어진 `ComponentRegistry` 인스턴스에 모든 기본 컴포넌트를 등록한다 |
| `initializeDefaultCatalog` | 함수 | 싱글톤 레지스트리에 기본 카탈로그를 등록한다 (앱 시작 시 1회 호출) |

## 상세 명세

### `registerDefaultCatalog(registry: ComponentRegistry): void`

**매개변수:** `registry` — 컴포넌트를 등록할 `ComponentRegistry` 인스턴스.  
**반환 타입:** `void`

`registry.register(name, { component })` 형태로 총 18개의 컴포넌트를 순서대로 등록한다.

- **Content 그룹 (6개):** `'Text'`, `'Image'`, `'Icon'`, `'Divider'`, `'Video'`, `'AudioPlayer'` — 크기가 작고 즉시 로드된다는 주석이 있음.
- **Layout 그룹 (4개):** `'Row'`, `'Column'`, `'List'`, `'Card'`
- **추가 Layout 그룹 (2개):** `'Tabs'`, `'Modal'`
- **Interactive 그룹 (6개):** `'Button'`, `'TextField'`, `'CheckBox'`, `'Slider'`, `'DateTimeInput'`, `'MultipleChoice'`

각 등록 호출은 문자열 키와 `{ component: <컴포넌트 구현체> }` 형태의 객체를 전달한다. 조건 분기나 에러 처리는 없으며 순수하게 순차 등록만 수행한다.

### `initializeDefaultCatalog(): void`

**매개변수:** 없음.  
**반환 타입:** `void`

`ComponentRegistry.getInstance()`로 싱글톤 인스턴스를 가져온 뒤 `registerDefaultCatalog`를 호출한다. 앱 시작 시 단 한 번 호출하도록 설계되어 있으며, 중복 호출 방지 로직은 `ComponentRegistry` 내부에 위임된다.

## 동작 흐름

1. 모듈 로드 시 모든 컴포넌트 구현체가 정적으로 import된다.
2. `initializeDefaultCatalog()`가 호출되면 싱글톤 레지스트리를 획득한다.
3. `registerDefaultCatalog`가 18개 컴포넌트를 순차적으로 레지스트리에 등록한다.
4. 이후 A2UI 렌더러는 레지스트리에서 컴포넌트 이름으로 구현체를 조회하여 렌더링한다.
