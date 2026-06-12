# renderers/lit/src/v0_9/tests/custom-catalogs.test.ts

## 개요

사용자 정의 `Catalog`를 생성하고 `MessageProcessor`와 통합하는 시나리오를 검증하는 통합 테스트 파일이다. 단일 커스텀 카탈로그만으로 서피스를 구성하는 경우와, 기본 카탈로그와 커스텀 카탈로그를 동시에 여러 서피스에서 구동하는 경우를 모두 테스트한다. 두 번째 테스트는 `a2ui-surface` 엘리먼트를 실제 DOM에 마운트하여 각 카탈로그가 올바른 커스텀 엘리먼트 태그를 렌더링하는지까지 검증한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `before`, `after`
- `@a2ui/web_core/v0_9` — `MessageProcessor`, `Catalog`
- `@a2ui/lit/v0_9` — `LitComponentApi` (타입 import)
- `zod` — `z` (스키마 정의)

### 저장소 내부 모듈
- [`./dom-setup.js`](./dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- `../surface/a2ui-surface.js` — `A2uiSurface` (타입 import, 동적 import로도 로드)
- `../catalogs/basic/index.js` — `basicCatalog` (동적 import)

## Exports

없음 (테스트 파일)

## 모듈 수준 상수

- `CustomWidgetApi`: 이름 `'CustomWidget'`, 태그명 `'a2ui-customwidget'`, 스키마 `z.object({ label: z.string().optional() })`를 가진 커스텀 컴포넌트 API 객체. `LitComponentApi`를 만족하도록 설계되었다.
- `customCatalog`: `new Catalog<LitComponentApi>('custom-catalog-v1', [CustomWidgetApi])`로 생성된 카탈로그 인스턴스. 카탈로그 ID는 `'custom-catalog-v1'`이다.

## 테스트 케이스

### describe: `Custom Catalogs Integration`

**픽스처 / 모킹**
- `before` 훅:
  1. `setupTestDom()`으로 JSDOM 전역을 주입한다.
  2. `../surface/a2ui-surface.js`를 동적으로 import하여 `a2ui-surface` 커스텀 엘리먼트를 등록한다.
  3. `../catalogs/basic/index.js`에서 `basicCatalog`를 동적으로 획득한다.
- `after` 훅: `teardownTestDom()` 호출.

---

#### `should allow users to define and use their own custom catalogs`
- 검증 동작:
  1. `new MessageProcessor<LitComponentApi>([customCatalog])`로 커스텀 카탈로그만을 사용하는 프로세서를 생성한다.
  2. `createSurface`(`surfaceId: 'custom-surface'`, `catalogId: customCatalog.id`)와 `updateComponents`(`id: 'root'`, `component: 'CustomWidget'`, `label: 'Hello Custom'`) 메시지를 처리한다.
  3. `processor.model.getSurface('custom-surface')`가 truthy임을 검증한다.
  4. `surface.componentsModel.get('root')`의 `type`이 `'CustomWidget'`임을 검증한다.

#### `should handle multiple catalogs concurrently across different surfaces`
- 검증 동작:
  1. `new MessageProcessor<LitComponentApi>([basicCatalog, customCatalog])`로 두 카탈로그를 동시에 사용하는 프로세서를 생성한다.
  2. 두 개의 `createSurface` 메시지를 처리한다: `'minimal-surface'`(`catalogId: basicCatalog.id`), `'custom-surface'`(`catalogId: customCatalog.id`).
  3. `'minimal-surface'`에 `component: 'Text'`, `text: 'From Minimal'`의 루트 컴포넌트를 추가한다.
  4. `'custom-surface'`에 `component: 'CustomWidget'`, `label: 'From Custom'`의 루트 컴포넌트를 추가한다.
  5. 두 서피스 모두 truthy임을 검증한다.
  6. `a2ui-surface` 엘리먼트 `el1`과 `el2`를 생성하여 각각 `minimalSurf`, `customSurf`를 설정하고 `asyncUpdate`로 DOM에 마운트한다.
  7. `el1.shadowRoot?.querySelector('a2ui-basic-text')`가 존재하는지 검증하여 기본 카탈로그 서피스가 `Text` 컴포넌트를 올바른 태그로 렌더링함을 확인한다.
  8. `el2.shadowRoot?.querySelector('a2ui-customwidget')`가 존재하는지 검증하여 커스텀 카탈로그 서피스가 동적으로 `<a2ui-customwidget>` 태그를 렌더링함을 확인한다.

## 동작 흐름

모듈 로드 시점에 `CustomWidgetApi`와 `customCatalog`가 정적으로 생성된다. 테스트 스위트 시작 시 JSDOM 전역 설정과 `a2ui-surface` 등록이 이루어진다. 첫 번째 테스트는 모델 레이어(데이터 모델만)를 검증하고, 두 번째 테스트는 실제 DOM 렌더링까지 포함한 end-to-end 검증이다.
