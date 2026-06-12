# renderers/angular/a2ui_explorer/src/app/tests/v0_9/34_child-list-template.spec.ts

## 개요

`ChildList Template Expansion` 예제의 브라우저 통합 테스트 파일이다. 자식 목록 템플릿 확장 기능이 올바르게 렌더링되는지 검증하며, 제목·아이템 이름·수량 데이터 표시 여부를 확인한다. 다른 테스트와 달리 `ComponentFixture`를 직접 import하지 않고 `getCanvas()`에만 의존하는 경량 구조이다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 테스트 스위트 정의만 포함하며, 외부로 내보내는 항목이 없다.

## 테스트 케이스 명세

### 픽스처 및 셋업

`describe('Example: ChildList Template Expansion')` 블록에서 `textContent: string`만 공유 변수로 선언한다 (`fixture` 없음).

`beforeEach` — 각 테스트 전에 비동기로 실행된다.
1. `loadExample('ChildList Template Expansion')`를 호출하여 예제를 로드한다 (반환값 무시).
2. `getCanvas().textContent`를 `textContent`에 저장한다.

### 테스트 케이스 1: `should render title`

- **검증 대상**: 캔버스에 `'Dynamic Item List'`라는 제목이 존재해야 한다.

### 테스트 케이스 2: `should render list items`

- **검증 대상**: 다음 아이템 이름 및 수량이 모두 캔버스에 존재해야 한다.
  - `'Apple'`, `'10'`
  - `'Banana'`, `'5'`
  - `'Cherry'`, `'20'`

### 테스트 케이스 3: `should render label`

- **검증 대상**: 캔버스에 `'Qty'` 레이블이 존재해야 한다 (수량 컬럼 헤더 확인).

## 동작 흐름

`loadExample` → `getCanvas().textContent` 셋업 후, 목록 제목·각 아이템 이름과 수량·레이블을 순차적으로 검증한다. DOM fixture 없이 텍스트 콘텐츠만으로 검증하는 경량 테스트 구조이므로 컴포넌트 인스턴스에 직접 접근할 수 없다.
