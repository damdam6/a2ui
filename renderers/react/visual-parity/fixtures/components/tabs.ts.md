# renderers/react/visual-parity/fixtures/components/tabs.ts

## 개요

Tabs 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. 2개 탭의 단순 구조, 4개 탭의 다중 구조, 그리고 중첩 컴포넌트(Image, CheckBox, Column 등)와 데이터 바인딩이 포함된 복잡한 3탭 구조까지 세 가지 시나리오를 제공한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `tabsBasic` | 상수 (`ComponentFixture`) | 2개 탭, 텍스트 콘텐츠 |
| `tabsMultiple` | 상수 (`ComponentFixture`) | 4개 탭, 텍스트 콘텐츠 |
| `tabsComplex` | 상수 (`ComponentFixture`) | 3개 탭, 복합 컴포넌트 + 데이터 바인딩 |
| `tabsFixtures` | 상수 (객체) | 위 3개 픽스처를 묶은 그룹 객체 |

## 상세 명세

### `tabsBasic: ComponentFixture`

- `root`: `'tabs-basic'`
- 컴포넌트:
  - `id: 'tab1-content'` — Text(`'Content for Tab 1'`)
  - `id: 'tab2-content'` — Text(`'Content for Tab 2'`)
  - `id: 'tabs-basic'` — Tabs, `tabItems` 2개: `{title: {literalString: 'Tab 1'}, child: 'tab1-content'}`, `{title: {literalString: 'Tab 2'}, child: 'tab2-content'}`

### `tabsMultiple: ComponentFixture`

- `root`: `'tabs-multi'`
- 컴포넌트: 4개의 Text 콘텐츠 컴포넌트와 루트 Tabs
  - `id: 'tabm1-content'` — Text(`'Overview content goes here.'`)
  - `id: 'tabm2-content'` — Text(`'Details and specifications.'`)
  - `id: 'tabm3-content'` — Text(`'Reviews from users.'`)
  - `id: 'tabm4-content'` — Text(`'Related items.'`)
  - `id: 'tabs-multi'` — Tabs, `tabItems` 4개: 각 title은 `'Overview'`, `'Details'`, `'Reviews'`, `'Related'`

### `tabsComplex: ComponentFixture`

- `root`: `'tabs-complex'`
- `data`: `{'/tabs/notify': true, '/tabs/dark': false}` — CheckBox 바인딩용 초기값
- 총 14개 컴포넌트로 구성된 3개 탭:

  **Profile 탭** (`child: 'profile-col'`):
  - `id: 'profile-avatar'` — Image(`url: 'https://picsum.photos/seed/user/64/64'`, `usageHint: 'avatar'`)
  - `id: 'profile-name'` — Text(`'John Doe'`, `usageHint: 'h3'`)
  - `id: 'profile-bio'` — Text(`'Software developer and UI enthusiast.'`)
  - `id: 'profile-col'` — Column(`['profile-avatar', 'profile-name', 'profile-bio']`)

  **Settings 탭** (`child: 'settings-col'`):
  - `id: 'settings-title'` — Text(`'Preferences'`, `usageHint: 'h3'`)
  - `id: 'settings-cb1'` — CheckBox(`label: 'Enable notifications'`, `value: {path: '/tabs/notify'}`)
  - `id: 'settings-cb2'` — CheckBox(`label: 'Dark mode'`, `value: {path: '/tabs/dark'}`)
  - `id: 'settings-col'` — Column(`['settings-title', 'settings-cb1', 'settings-cb2']`)

  **Activity 탭** (`child: 'activity-col'`):
  - `id: 'activity-title'` — Text(`'Recent Activity'`, `usageHint: 'h3'`)
  - `id: 'activity-1'` — Text(`'Completed task A'`)
  - `id: 'activity-2'` — Text(`'Updated profile'`)
  - `id: 'activity-3'` — Text(`'Joined project B'`)
  - `id: 'activity-col'` — Column(`['activity-title', 'activity-1', 'activity-2', 'activity-3']`)

  - `id: 'tabs-complex'` — Tabs, `tabItems`: `{title: 'Profile', child: 'profile-col'}`, `{title: 'Settings', child: 'settings-col'}`, `{title: 'Activity', child: 'activity-col'}`

### `tabsFixtures: object`

`tabsBasic`, `tabsMultiple`, `tabsComplex` 3개를 키-값으로 묶은 집계 객체.

## 동작 흐름

각 픽스처는 탭 콘텐츠 컴포넌트를 먼저 선언하고, 마지막 엔트리의 Tabs 컴포넌트가 `tabItems` 배열을 통해 각 탭의 title과 child ID를 연결한다. `tabsComplex`는 `data` 필드를 통해 CheckBox 초기 상태를 주입하며, picsum.photos의 seed `user`를 사용한 아바타 이미지도 포함한다.
