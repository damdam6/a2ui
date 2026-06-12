# renderers/angular/a2ui_explorer/src/app/tests/v0_9/14_sports-player.spec.ts

## 개요

`Sports Player` 예제 컴포넌트의 렌더링을 검증하는 Angular 통합 테스트 파일이다. 선수 이름·등번호·소속팀·통계 수치(PPG, RPG, APG) 텍스트가 올바르게 표시되는지, 그리고 선수 이미지의 `src` URL이 올바른지 확인한다. 인터랙션 테스트는 포함하지 않는다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Sports Player')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach`**
- `loadExample('Sports Player')`를 비동기로 호출하여 `fixture`를 얻는다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 텍스트에 `'Marcus Johnson'`, `'#23'`, `'LA Lakers'`, `'28.4'`, `'PPG'`, `'7.2'`, `'RPG'`, `'6.8'`, `'APG'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render image`**
- 검증 동작: `querySelector('img')`로 이미지 요소가 존재하는지 확인하고, `getAttribute('src')`가 정확히 `'https://images.unsplash.com/photo-1546519638-68e109498ffc?w=200&h=200&fit=crop'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리.

## 동작 흐름

`beforeEach`에서 `'Sports Player'` 예제를 로드하고 텍스트를 추출한다. 두 개의 `it` 블록이 각각 선수 정보 텍스트와 이미지 URL을 독립적으로 검증한다.
