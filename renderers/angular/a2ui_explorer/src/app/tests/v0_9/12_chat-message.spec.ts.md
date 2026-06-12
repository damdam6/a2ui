# renderers/angular/a2ui_explorer/src/app/tests/v0_9/12_chat-message.spec.ts

## 개요

`Chat Message` 예제 컴포넌트의 렌더링을 검증하는 Angular 통합 테스트 파일이다. 채널 이름, 두 사용자의 메시지 내용, 그리고 두 아바타 이미지의 URL이 올바르게 렌더링되는지 확인한다. 버튼 클릭 등 인터랙션 테스트는 포함하지 않는다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Chat Message')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach`**
- `loadExample('Chat Message')`를 비동기로 호출하여 `fixture`를 얻는다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render channel name`**
- 검증 동작: 텍스트에 `'project-updates'` 채널 이름이 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render messages content`**
- 검증 동작: 텍스트에 첫 번째 발신자 이름 `'Mike Chen'`과 메시지 `'Just pushed the new API changes. Ready for review.'`, 두 번째 발신자 `'Sarah Kim'`과 메시지 `"Great! I'll take a look after standup."` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render images`**
- 검증 동작: `querySelectorAll('img')`로 이미지 요소 배열을 수집하고 길이가 2 이상인지 확인한다. 각 `img` 요소의 `src` 배열에 `'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=40&h=40&fit=crop'`과 `'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=40&h=40&fit=crop'` 두 URL이 모두 포함되어 있는지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리.

## 동작 흐름

`beforeEach`에서 `'Chat Message'` 예제를 로드하고 텍스트를 추출한다. 세 개의 `it` 블록이 각각 채널 이름, 두 사용자 메시지 본문, 아바타 이미지 URL을 독립적으로 검증한다.
