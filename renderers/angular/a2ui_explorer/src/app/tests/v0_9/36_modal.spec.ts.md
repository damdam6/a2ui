# renderers/angular/a2ui_explorer/src/app/tests/v0_9/36_modal.spec.ts

## 개요

`Modal Sample` 예제의 브라우저 통합 테스트 파일이다. 단일 테스트 케이스로 모달 컴포넌트의 전체 생명주기(닫힘 초기 상태 → 트리거 클릭으로 열림 → 내용 검증 → 닫기 버튼으로 닫힘)를 검증한다. `beforeEach` 없이 `it` 블록 내에서 직접 `loadExample`을 호출하는 구조이다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `loadExample`, `wait`

## Exports

이 파일은 테스트 스위트 정의만 포함하며, 외부로 내보내는 항목이 없다.

## 테스트 케이스 명세

### 테스트 케이스 1: `should open and close modal`

이 단일 테스트가 모달의 전체 인터랙션 흐름을 순서대로 검증한다. `async` 함수 본문 안에서 `loadExample('Modal Sample')`을 직접 호출하여 `fixture`를 얻는다.

**단계별 검증 흐름:**

1. **트리거 렌더링 확인**: `fixture.nativeElement.querySelector('.a2ui-modal-trigger')`로 트리거 요소를 찾고 truthy인지 확인한다.

2. **초기 닫힘 상태 확인**: `fixture.nativeElement.querySelector('.a2ui-modal-overlay')`가 falsy(null)인지 확인하여 모달이 처음에는 닫혀 있음을 검증한다.

3. **모달 열기**: `trigger.click()`을 호출한 후 `fixture.detectChanges()` → `wait(100)` → `fixture.detectChanges()` 순으로 변경을 반영한다.

4. **열린 상태 확인**: `.a2ui-modal-overlay`가 truthy인지 확인하여 모달이 열렸음을 검증한다.

5. **내용 확인**: `.a2ui-modal-overlay`의 `textContent`에 `'This is the content inside the modal.'`이 포함되어야 한다.

6. **닫기 버튼 확인**: `fixture.nativeElement.querySelector('.a2ui-modal-close')`가 truthy인지 확인한다.

7. **모달 닫기**: `closeBtn.click()`을 호출한 후 `fixture.detectChanges()`로 변경을 반영한다 (`wait()` 미사용).

8. **닫힘 상태 확인**: `.a2ui-modal-overlay`가 다시 falsy인지 확인하여 모달이 완전히 닫혔음을 검증한다.

**사용하는 CSS 클래스**: `.a2ui-modal-trigger`, `.a2ui-modal-overlay`, `.a2ui-modal-close`

## 동작 흐름

`loadExample` → 트리거 DOM 쿼리 → 초기 닫힘 상태 검증 → `click()` + `detectChanges()` + `wait(100)` → 열린 상태 검증 → 내용 검증 → 닫기 버튼 클릭 + `detectChanges()` → 최종 닫힘 상태 검증의 선형 흐름으로 구성된다. 모달의 표시/숨김은 `.a2ui-modal-overlay` 요소의 DOM 존재 여부로 판단한다.
