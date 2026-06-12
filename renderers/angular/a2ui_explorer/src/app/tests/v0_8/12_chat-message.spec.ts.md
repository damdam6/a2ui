# renderers/angular/a2ui_explorer/src/app/tests/v0_8/12_chat-message.spec.ts

## 개요

v0.8 버전의 "Chat Message (basic)" 예제 컴포넌트가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 채널명, 메시지 발신자, 타임스탬프, 메시지 본문을 포함하는 텍스트 렌더링만 검증하는 단일 케이스로 구성되어 있으며, 버튼 액션 테스트는 포함하지 않는다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 유틸리티

## Exports

이 파일은 `describe` 블록으로 테스트를 등록할 뿐 명시적으로 export하는 항목이 없다.

## 테스트 케이스 명세

### 스위트: `Example: Chat Message (basic) (v0.8)`

**픽스처 및 공유 변수**
- `textContent: string` — 캔버스 요소의 텍스트 내용

**`beforeEach`**
`loadExample('Chat Message (basic)', Version.V0_8)`를 비동기로 호출하여 예제를 로드하고, `getCanvas().textContent`로 렌더링된 텍스트를 추출해 `textContent`에 저장한다. `ComponentFixture`를 별도로 유지하지 않는다.

---

#### 테스트 1: `should render expected text content`
- **검증 동작**: 캔버스에 렌더링된 텍스트가 채팅 메시지 UI에 필요한 모든 정보를 포함하는지 확인한다.
- **검증 항목**: `'project-updates'`(채널명), `'Mike Chen'`(첫 번째 메시지 발신자), `'10:32 AM'`(첫 번째 타임스탬프), `'Just pushed the new API changes. Ready for review.'`(첫 번째 메시지 본문), `'Sarah Kim'`(두 번째 메시지 발신자), `'10:45 AM'`(두 번째 타임스탬프), `"Great! I'll take a look after standup."`(두 번째 메시지 본문) 포함 여부를 `toContain`으로 각각 단언한다.
- **픽스처/모킹**: `loadExample` 호출 결과 반환된 `textContent` 사용.

## 동작 흐름

`beforeEach`에서 v0.8 Chat Message 예제를 로드하고 캔버스 텍스트를 가져온다. 단일 텍스트 렌더링 테스트에서 채널명, 두 발신자(Mike Chen, Sarah Kim)의 이름, 각 타임스탬프, 각 메시지 본문 일곱 개 항목이 모두 텍스트에 포함되어 있는지 순차적으로 검증한다.
