# renderers/angular/a2ui_explorer/src/app/tests/v0_9/06_music-player.spec.ts

## 개요

v0.9 버전의 "Music Player" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 곡 정보 텍스트(곡명, 아티스트, 재생 시간), 앨범 아트 이미지 URL, 그리고 플레이어 컨트롤 Material 아이콘 텍스트를 총 3개의 테스트로 검증한다. 인터랙션 이벤트 테스트는 포함하지 않으며 순수 정적 렌더링에 집중한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Music Player'`

**픽스처 / 셋업**
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값.
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Music Player')`를 `await`로 호출해 `fixture`를 얻고, `getCanvas().textContent`를 저장한다.

---

#### 테스트 1: `'should render text content'`

캔버스 텍스트에 다음이 포함되는지 확인한다.
- 곡명: `'Blinding Lights'`
- 아티스트: `'The Weeknd'`
- 현재 재생 위치: `'1:48'`
- 전체 재생 시간: `'4:22'`

---

#### 테스트 2: `'should render image'`

1. `fixture.nativeElement.querySelector('img')`로 이미지 요소를 조회하고 `truthy`인지 확인한다.
2. `img.getAttribute('src')`가 `'https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?w=300&h=300&fit=crop'`과 정확히 일치하는지 검증한다.

---

#### 테스트 3: `'should render icons'`

캔버스 텍스트에 Material 아이콘 이름이 포함되는지 확인한다.
- 이전 곡: `'skip_previous'`
- 다음 곡: `'skip_next'`
- 일시 정지: `'pause'`

**참고**: Material Icons는 폰트 리가처(ligature) 방식으로 렌더링되므로 아이콘 이름이 캔버스 `textContent`에 그대로 노출된다.

## 동작 흐름

`beforeEach` → `loadExample`로 예제 초기화 → 텍스트 스냅샷 → 테스트 1(곡 정보 텍스트) → 테스트 2(`fixture.nativeElement`에서 `img` DOM 조회 및 src 검증) → 테스트 3(플레이어 컨트롤 아이콘 이름 검증).
