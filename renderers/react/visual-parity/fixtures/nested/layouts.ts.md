# renderers/react/visual-parity/fixtures/nested/layouts.ts

## 개요

시각적 동등성(visual parity) 테스트를 위한 복합 레이아웃 픽스처(fixture)들을 정의하는 파일이다. 각 픽스처는 `ComponentFixture` 타입의 상수이며, 여러 컴포넌트가 중첩된 트리 구조를 평탄화된 배열로 표현한다. 총 7개의 픽스처가 정의되며, 이를 단일 객체 `nestedFixtures`로 묶어 내보낸다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입 정의

### 외부 패키지

없음.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `nestedCardInList` | 상수 (`ComponentFixture`) | 두 개의 카드를 List로 감싼 픽스처 |
| `nestedForm` | 상수 (`ComponentFixture`) | 연락처 폼을 Card로 감싼 픽스처 |
| `nestedRowInColumn` | 상수 (`ComponentFixture`) | Column 안에 세 개의 Row가 중첩된 픽스처 |
| `nestedColumnInRow` | 상수 (`ComponentFixture`) | Row 안에 세 개의 Column이 중첩된 픽스처 |
| `nestedDashboard` | 상수 (`ComponentFixture`) | 헤더 + 통계 카드 3개로 구성된 대시보드 픽스처 |
| `nestedProfile` | 상수 (`ComponentFixture`) | 아바타·이름·약력·연락처가 포함된 프로필 카드 픽스처 |
| `nestedSettings` | 상수 (`ComponentFixture`) | 체크박스·슬라이더·버튼이 포함된 설정 패널 픽스처 |
| `nestedFixtures` | 상수 (객체) | 위 7개 픽스처를 이름을 키로 묶은 레지스트리 객체 |

## 상세 명세

모든 픽스처는 `ComponentFixture` 구조를 따른다. `root` 필드는 렌더링 시작점이 되는 컴포넌트 `id`를 지정하며, `components` 배열은 컴포넌트 트리를 평탄화하여 leaf → parent 순서로 나열한다(부모가 자식의 `id`를 참조하므로 순서가 중요하다).

### `nestedCardInList`

- `root`: `'nested-card-list'`
- Card 2개를 `List`로 감싼 구조. 각 Card는 Icon + Text(h3 타이틀) 를 Row로 묶고, 그 Row와 Text(caption) 를 Column으로 묶은 뒤 Card에 넣는다.
  - Card 1: icon `'folder'`, 타이틀 `'Documents'`, 카운트 `'24 files'`
  - Card 2: icon `'image'`, 타이틀 `'Photos'`, 카운트 `'156 images'`
- 최상위 컴포넌트 `nested-card-list`는 `List`이며 두 카드를 `explicitList`로 나열한다.

### `nestedForm`

- `root`: `'nested-form'`
- `data` 없음.
- h2 타이틀(`'Contact Form'`) + caption(`'Fill out the form below'`) + TextField 3개(`'Name'`, `'Email'`, `'Message'`) + 기본 버튼(`'Submit'`, `action.name: 'submit'`, `primary: true`)를 Column으로 묶고, 최상위를 Card로 감싼다.

### `nestedRowInColumn`

- `root`: `'nested-row-col'`
- 3×3 텍스트 그리드: Row 3개(각각 `'A1'`·`'A2'`·`'A3'`, `'B1'`~`'B3'`, `'C1'`~`'C3'`)를 하나의 Column에 수직으로 쌓는다.

### `nestedColumnInRow`

- `root`: `'nested-col-row'`
- Column 3개(각각 텍스트 2개: `'Col 1 - A'`/`'Col 1 - B'` 등)를 Row 하나에 수평으로 나란히 배치한다.

### `nestedDashboard`

- `root`: `'nested-dashboard'`
- 헤더 Row(icon `'dashboard'` + h1 `'Dashboard'`)와 통계 Row로 구성된 Column.
- 통계 Row에는 Card 3개: 값(h2) + 레이블(caption) 쌍으로 구성되며, 값/레이블은 각각 `'1,234'`/`'Users'`, `'567'`/`'Orders'`, `'$12,345'`/`'Revenue'`.

### `nestedProfile`

- `root`: `'nested-profile'`
- 아바타 `Image`(`url: 'https://picsum.photos/seed/profile/80/80'`, `usageHint: 'avatar'`) + Column(이름 h2 `'Jane Doe'`, 역할 caption `'Product Designer'`)을 Row로 묶어 프로필 헤더를 구성한다.
- 이후 `Divider`, 약력 제목 h3(`'About'`), 약력 본문 Text, 연락처 제목 h3(`'Contact'`), icon(`'mail'`) + text(`'jane@example.com'`)을 Row로 묶은 Row를 순서대로 Column에 쌓는다.
- 최상위는 Card(`nested-profile`).

### `nestedSettings`

- `root`: `'nested-settings'`
- `data`: JSON Pointer 경로로 초기값 설정
  - `'/settings/notify'`: `true`
  - `'/settings/dark'`: `false`
  - `'/settings/auto'`: `true`
  - `'/settings/volume'`: `60`
- h2 제목(`'Settings'`) + 환경설정 섹션(h3 + CheckBox 3개, 각 `value`가 경로 바인딩) + Slider(`value: {path: '/settings/volume'}`, `minValue: 0`, `maxValue: 100`) + Divider + 버튼 Row(Save primary, Cancel non-primary) 를 Column으로 묶고 Card로 감싼다.

### `nestedFixtures`

키: 픽스처 이름 문자열, 값: 해당 `ComponentFixture` 상수인 일반 객체이다. 키 목록: `nestedCardInList`, `nestedForm`, `nestedRowInColumn`, `nestedColumnInRow`, `nestedDashboard`, `nestedProfile`, `nestedSettings`.

## 동작 흐름

이 파일은 순수한 데이터 선언 파일이다. 모듈 로드 시 각 상수가 메모리에 할당되며, 소비자는 `nestedFixtures` 또는 개별 named export를 통해 픽스처를 가져다 테스트 하네스(harness)에 주입한다. 런타임 로직은 없다.
